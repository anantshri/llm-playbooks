# glm-4.7-flash
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** GLM-4.7 Flash (UD-Q8_K_XL GGUF)

## Model card summary (from the deployed config)

- Type: MoE with MLA attention (deepseek2 arch)
- Parameter: 30B total / 3B active
- Source: https://huggingface.co/unsloth/GLM-4.7-Flash-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: UD-Q8_K_XL (35.6 GB)
- KV Cache: F16 @ 202752
- Context window: 202752 (model-card native)
- VRAM: 43.7 GiB of 47.8 (4.1 GiB free) — measured 2026-09-08 at F16
- Chat template: built-in GGUF template (no --jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.0605, 'output': 0.4}`

## Config block

```yaml
glm-4.7-flash:
  name: GLM-4.7 Flash (UD-Q8_K_XL GGUF)
  description: 'Type: MoE with MLA attention (deepseek2 arch)

    Parameter: 30B total / 3B active

    Source: https://huggingface.co/unsloth/GLM-4.7-Flash-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: UD-Q8_K_XL (35.6 GB)

    KV Cache: F16 @ 202752

    Context window: 202752 (model-card native)

    VRAM: 43.7 GiB of 47.8 (4.1 GiB free) — measured 2026-09-08 at F16

    Chat template: built-in GGUF template (no --jinja; reasoning parsed)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/GLM-4.7-Flash/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -b 4096 -ub 1024

    --spec-type ngram-mod

    -c 202752

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --reasoning-format deepseek

    --min-p 0.01

    '
  ttl: 600
  pricing:
    input: 0.0605
    output: 0.4
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/GLM-4.7-Flash/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--spec-type` | `ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `-c` | `202752` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--min-p` | `0.01` | Default min-p sampling. |

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
| pp @9k | 5393 tok/s |
| pp @max (197k ctx) | 563 tok/s |
| tg @9k (shallow) | 101 tok/s |
| tg @max (197k ctx) | 29 tok/s |
