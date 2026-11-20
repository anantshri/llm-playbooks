# qwen3.6-35b-a3b
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.6 35B-A3B — UD-Q4_K_M weights, F16 KV, 204800

## Model card summary (from the deployed config)

- Type: MoE (qwen3_5_moe)
- Parameter: 35B total / 3B active
- Source: https://huggingface.co/unsloth/Qwen3.6-35B-A3B-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: UD-Q4_K_M
- KV Cache: F16 @ 204800
- Context window: 204800
- VRAM: 25.0 GiB of 47.8 (22.8 GiB free) — measured 2026-09-08 at F16
- Chat template: built-in GGUF template (no --jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.1, 'output': 0.9, 'cached': 0.05}`

## Config block

```yaml
qwen3.6-35b-a3b:
  name: Qwen3.6 35B-A3B — UD-Q4_K_M weights, F16 KV, 204800
  description: 'Type: MoE (qwen3_5_moe)

    Parameter: 35B total / 3B active

    Source: https://huggingface.co/unsloth/Qwen3.6-35B-A3B-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: UD-Q4_K_M

    KV Cache: F16 @ 204800

    Context window: 204800

    VRAM: 25.0 GiB of 47.8 (22.8 GiB free) — measured 2026-09-08 at F16

    Chat template: built-in GGUF template (no --jinja; reasoning parsed)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Qwen3.6-35B-A3B/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    --spec-type ngram-mod

    -c 204800

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --reasoning-format deepseek

    --top-k 20

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.1
    output: 0.9
    cached: 0.05
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Qwen3.6-35B-A3B/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `--spec-type` | `ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `-c` | `204800` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--top-k` | `20` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Measured (llama-bench)

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). tg@9k is shallow decode; tg@max is the deep-decode walk (~98% of -c). Do not mix with gateway measurements (see troubleshooting).

| Metric | Value |
|---|---|
| pp @9k | 6530 tok/s |
| pp @max (198k ctx) | 3764 tok/s |
| tg @9k (shallow) | 168 tok/s |
| tg @max (198k ctx) | 74 tok/s |
