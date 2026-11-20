# gemma4-31b-mtp
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Gemma 4 31B-it — UD-Q8_K_XL, Q8_0 KV, 163840, MTP+ngram draft

## Model card summary (from the deployed config)

- Type: MoE (gemma-4) | MTP speculative (~1.4x decode) | reasoning tiers
- Parameter: 31B total
- Source: https://huggingface.co/unsloth/gemma-4-31B-it-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: UD-Q8_K_XL + mtp-drafter.gguf
- KV Cache: Q8_0 @ 163840
- Context window: 163840 (VRAM-capped; native 262144)
- VRAM: 45.7 GiB of 47.8 (2.1 GiB free) — measured 2026-09-08
- Chat template: Gemma (--jinja; reasoning_effort top-level only)

## llama-swap entry

- **ttl:** `600`
- **filters:** `{'setParamsByID': {'${MODEL_ID}:low': {'reasoning_effort': 'low'}, '${MODEL_ID}:medium': {'reasoning_effort': 'medium'}, '${MODEL_ID}:high': {'reasoning_effort': 'high'}}}`
- **pricing:** `{'input': 0.09, 'output': 0.34, 'cached': 0.05}`

## Config block

```yaml
gemma4-31b-mtp:
  name: Gemma 4 31B-it — UD-Q8_K_XL, Q8_0 KV, 163840, MTP+ngram draft
  description: 'Type: MoE (gemma-4) | MTP speculative (~1.4x decode) | reasoning tiers

    Parameter: 31B total

    Source: https://huggingface.co/unsloth/gemma-4-31B-it-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: UD-Q8_K_XL + mtp-drafter.gguf

    KV Cache: Q8_0 @ 163840

    Context window: 163840 (VRAM-capped; native 262144)

    VRAM: 45.7 GiB of 47.8 (2.1 GiB free) — measured 2026-09-08

    Chat template: Gemma (--jinja; reasoning_effort top-level only)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/gemma-4-31B-it/current.gguf

    -md ${gguf-models}/gemma-4-31B-it/mtp-drafter.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -b 4096 -ub 1024

    -c 163840

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --spec-type draft-mtp,ngram-mod

    --spec-draft-n-max 4

    --top-k 64

    --min-p 0.0

    '
  filters:
    setParamsByID:
      ${MODEL_ID}:low:
        reasoning_effort: low
      ${MODEL_ID}:medium:
        reasoning_effort: medium
      ${MODEL_ID}:high:
        reasoning_effort: high
  ttl: 600
  pricing:
    input: 0.09
    output: 0.34
    cached: 0.05
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/gemma-4-31B-it/current.gguf` | Weights file (GGUF) this entry serves. |
| `-md` | `${gguf-models}/gemma-4-31B-it/mtp-drafter.gguf` | Drafter weights for speculative decoding (MTP drafter GGUF, DFlash drafter, etc.). |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `-c` | `163840` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `draft-mtp,ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `4` | Max tokens speculated per step. |
| `--top-k` | `64` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `filters` | Gateway request filters applied before proxying to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Measured (llama-bench)

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). tg@9k is shallow decode; tg@max is the deep-decode walk (~98% of -c). Do not mix with gateway measurements (see troubleshooting).

| Metric | Value |
|---|---|
| pp @9k | 2016 tok/s |
| pp @max (193k ctx) | 762 tok/s |
| tg @9k (shallow) | 48 tok/s |
| tg @max (193k ctx) | 22 tok/s |
