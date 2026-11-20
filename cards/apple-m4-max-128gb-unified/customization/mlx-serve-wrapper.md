# MLX serving wrapper (deployable)

Wraps `mlx_lm.server` so MLX models appear as regular gateway entries with the
served model id == the model dir name (mlx_lm uses the literal `--model` string
as the served id and tries to FETCH from the HF Hub on mismatch - the wrapper
`cd`s into the models dir and passes the bare dir name). Adds reasoning-off and
a self-healing wedge watchdog: kill only when /health=200 AND a streaming probe
gets 0 chunks AND the log has been stale > 60 s. The log-freshness check is
load-bearing: a long prefill also yields 0 chunks but keeps the log fresh; the
first version of this script killed healthy workers mid-prefill and looked
exactly like the wedge it was fixing (2026-07-19).

```sh
#!/bin/sh
# mlx-serve.sh — wrapper so llama-swap can serve an MLX model under its bare dir name,
# with reasoning DISABLED and a self-healing watchdog.
#
# Why a wrapper at all: mlx_lm.server uses the literal `--model` string as the served
# model id, and if a request's `model` field doesn't match, it tries to FETCH it from
# the HF Hub (401/404). llama-swap forwards the client's model field unchanged and
# execs backends directly (no shell), so we can't `cd ... && ...` inline to pass a
# relative name. This wrapper does the `cd` into the MLX models dir and runs
# mlx_lm.server with the BARE model dir name, making the served id == the dir name
# (which matches the llama-swap entry key clients request).
#
# Reasoning OFF (2026-07-19): `--chat-template-args '{"enable_thinking":false}'`.
# The Qwen3.6 template gates <think> on `enable_thinking` (chat_template.jinja:149);
# default ON burns hundreds-to-thousands of tokens before any visible content. a coding agent
# uses the model as a fast chat/tool model, so reasoning is unwanted. Re-enable per-request
# via chat_template_kwargs.enable_thinking=true.
#
# 2026-10-16: MLX_PYTHON env-overridable (see assignment below) for models needing a
# newer mlx-lm than venv-mlx's pinned 0.31.3; default behavior unchanged.
#
# Self-healing watchdog (2026-07-19): the mlx MODEL WORKER can wedge — a stuck GPU eval
# (thread parks in mlx::core::eval -> condition_variable::wait, ~0% CPU) — while the HTTP
# layer stays up, so /health returns 200 and llama-swap keeps routing into a dead worker.
# The watchdog probes the bound port every 30s and restarts the worker when wedged.
#
#   CRITICAL — must distinguish WEDGE from LONG PREFILL (learned the hard way):
#   A first version killed on "0 stream chunks in 15s" alone. That FALSE-POSITIVED on
#   large-context requests: a long prefill monopolizes the worker, so the probe gets 0
#   chunks too, and the watchdog nuked a healthy worker mid-prefill -> a restart loop
#   that looked exactly like the "stuck" we were trying to fix. The worker's CPU time is
#   NOT a discriminator (it's GPU-bound + low-CPU during BOTH healthy prefill and a wedge).
#   The reliable signal is LOG FRESHNESS: mlx emits "Prompt processing progress" lines
#   while prefilling (and per-request access-log lines on completion), so the log mtime
#   stays fresh during any legitimate work; a true wedge goes silent. So we kill ONLY when
#   the probe gets 0 chunks AND the log has been stale > 60s. That accommodates any
#   prefill size (progress lines stay <60s apart at mlx's prefill throughput) while still
#   clearing a real wedge in ~60-75s.
#
# Usage: mlx-serve.sh <model-dir-name> [mlx_lm.server args...]
#   e.g. mlx-serve.sh my-mlx-model --host 127.0.0.1 --port 5800
set -u

MODEL_DIR_NAME="$1"; shift

MLX_MODELS_DIR="~/inference/models/mlx"
# Default stays venv-mlx (mlx-lm 0.31.3, backs the existing entries). A model needing a
# newer mlx-lm sets MLX_PYTHON via its llama-swap `env:` (e.g. spark2_5 native support,
# venv-mlx-spark). Set before this line, unbound-safe via ${MLX_PYTHON:-...} + set -u.
MLX_PYTHON="${MLX_PYTHON:-~/inference/venv-mlx/bin/python}"
export PYTHONUNBUFFERED=1   # ensure progress lines flush to the log promptly

# Pull --host/--port out of the forwarded args.
HOST="127.0.0.1"; PORT=""
prev=""
for a in "$@"; do
  case "$prev" in
    --host) HOST="$a" ;;
    --port) PORT="$a" ;;
  esac
  prev="$a"
done
: "${PORT:?mlx-serve.sh: --port is required (the watchdog needs it)}"

# mlx stdout+stderr -> a per-instance log the watchdog reads for freshness, and that we
# can tail for crash tracebacks. Truncated on every (re)start, so it stays bounded.
LOGFILE="~/inference/logs/mlx-${MODEL_DIR_NAME}-${PORT}.log"
: > "$LOGFILE"

cd "${MLX_MODELS_DIR}" || { echo "mlx-serve.sh: cannot cd to ${MLX_MODELS_DIR}" >&2; exit 1; }

MLX_PID=""
WD_PID=""

teardown() {
  [ -n "${MLX_PID:-}" ] && kill -TERM "$MLX_PID" 2>/dev/null || true
  [ -n "${WD_PID:-}" ]  && kill -TERM "$WD_PID"  2>/dev/null || true
  sleep 2
  [ -n "${MLX_PID:-}" ] && kill -KILL "$MLX_PID" 2>/dev/null || true
}
trap 'exit 0' INT TERM HUP
trap 'teardown' EXIT

# watchdog(host, port, mlx_pid, model, logfile): restart the worker if it WEDGES.
# Wedge = /health=200 AND probe returns 0 stream chunks AND log stale > 60s.
# A long prefill also yields 0 chunks, but keeps the log fresh -> not killed.
watchdog() {  # $1=host $2=port $3=mlx_pid $4=model_dir_name $5=logfile
  _host="$1"; _port="$2"; _pid="$3"; _model="$4"; _log="$5"
  while kill -0 "$_pid" 2>/dev/null; do
    sleep 30
    kill -0 "$_pid" 2>/dev/null || break
    _hc=$(curl -s -o /dev/null -w '%{http_code}' --max-time 5 \
            "http://${_host}:${_port}/health" 2>/dev/null || true)
    [ "${_hc:-000}" = "200" ] || continue
    # tiny streaming probe — a healthy, idle worker streams the first chunk in ~1-2s.
    # during a long prefill this may time out with 0 chunks; that alone is NOT a wedge.
    _body=$(printf '{"model":"%s","stream":true,"max_tokens":4,"messages":[{"role":"user","content":"hi"}]}' "$_model")
    _chunks=$(printf '%s' "$_body" \
        | curl -s --max-time 15 "http://${_host}:${_port}/v1/chat/completions" \
            -H 'Content-Type: application/json' --data @- 2>/dev/null \
        | grep -c '^data:' || true)
    [ "${_chunks:-0}" -gt 0 ] && continue
    # 0 chunks: discriminate WEDGE from busy prefill via log freshness.
    _age=999
    if [ -f "$_log" ]; then
      _mt=$(stat -f %m "$_log" 2>/dev/null || echo 0)
      _now=$(date +%s)
      _age=$((_now - _mt))
    fi
    if [ "$_age" -lt 60 ]; then
      continue   # log fresh -> worker is making progress (prefill/generation) -> leave it
    fi
    echo "[mlx-watchdog] ${_model}: wedged (/health=200, 0 stream chunks in 15s, log stale ${_age}s); restarting PID ${_pid}" >&2
    kill -TERM "$_pid" 2>/dev/null || true
    sleep 4
    kill -KILL "$_pid" 2>/dev/null || true
    return
  done
}

# --max-tokens 16384 is only the FALLBACK for requests that omit max_tokens (a request
# carrying max_tokens / max_completion_tokens still overrides it). mlx_lm.server's own
# default is 512.
"${MLX_PYTHON}" -m mlx_lm.server --model "${MODEL_DIR_NAME}" \
    --max-tokens 16384 \
    --chat-template-args '{"enable_thinking":false}' \
    "$@" > "$LOGFILE" 2>&1 &
MLX_PID=$!

watchdog "$HOST" "$PORT" "$MLX_PID" "${MODEL_DIR_NAME}" "$LOGFILE" &
WD_PID=$!

# Block until the mlx worker exits (clean stop, crash, or watchdog kill).
wait "$MLX_PID" 2>/dev/null || true
exit 0

```
