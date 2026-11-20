# param2-17b
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Param-2 17B-A2.4B Thinking (Q8_0 self-quant, CUDA, F16 KV, 32k)

## Model card summary (from the deployed config)

- Type: MoE (param2moe via glm4_moe graph, 64 experts / 6 active + 2 shared, sigmoid) | thinking always-on | text-only
- Parameter: 17B total / 2.4B active
- Source: https://huggingface.co/bharatgenai/Param2-17B-A2.4B-Thinking (self-quant)
- Engine: CUDA (sm_120) — side-build llama.cpp-param2
- Inference: anantshri/llama.cpp fleet-stable-20261021 @ 458f4d672 + 0001-param2moe-20261103.patch
- Quant: self-quant Q8_0 (17.2 GiB)
- KV Cache: F16 @ 32768 (native; ~1.4 GiB)
- Context window: 32768 (native)
- Chat template: embedded (--jinja; reasoning parsed, deepseek format)
- Sampling: temp 0.7 / top_p 0.9 / top_k 50 (card) — min_p pinned 0.0

## llama-swap entry

- **ttl:** `600`

## Config block

```yaml
param2-17b:
  name: Param-2 17B-A2.4B Thinking (Q8_0 self-quant, CUDA, F16 KV, 32k)
  description: 'Type: MoE (param2moe via glm4_moe graph, 64 experts / 6 active + 2 shared, sigmoid) |
    thinking always-on | text-only

    Parameter: 17B total / 2.4B active

    Source: https://huggingface.co/bharatgenai/Param2-17B-A2.4B-Thinking (self-quant)

    Engine: CUDA (sm_120) — side-build llama.cpp-param2

    Inference: anantshri/llama.cpp fleet-stable-20261021 @ 458f4d672 + 0001-param2moe-20261103.patch

    Quant: self-quant Q8_0 (17.2 GiB)

    KV Cache: F16 @ 32768 (native; ~1.4 GiB)

    Context window: 32768 (native)

    Chat template: embedded (--jinja; reasoning parsed, deepseek format)

    Sampling: temp 0.7 / top_p 0.9 / top_k 50 (card) — min_p pinned 0.0

    '
  cmd: '${param2-llama-server-bin}

    -m ${gguf-models}/Param2-17B-A2.4B-Thinking/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 32768

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --reasoning-format deepseek

    --temp 0.7 --top-p 0.9 --top-k 50 --min-p 0.0

    '
  ttl: 600
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Param2-17B-A2.4B-Thinking/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `32768` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--temp` | `0.7` | Default sampling temperature (client per-request values override). |
| `--top-p` | `0.9` | Default top-p sampling. |
| `--top-k` | `50` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
