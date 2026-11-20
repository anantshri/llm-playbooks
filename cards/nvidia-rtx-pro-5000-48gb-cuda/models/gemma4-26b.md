# gemma4-26b
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Gemma 4 26B A4B (UD-Q8_K_XL GGUF)

## Model card summary (from the deployed config)

- Type: MoE (gemma-4 A4B)
- Parameter: 26B total / 4B active
- Source: https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: UD-Q8_K_XL
- KV Cache: F16 @ 262144
- Context window: 262144
- VRAM: 33.9 GiB of 47.8 (13.9 GiB free) — measured 2026-09-08 at F16
- Chat template: Gemma (--jinja)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.07, 'output': 0.34}`

## Config block

```yaml
gemma4-26b:
  name: Gemma 4 26B A4B (UD-Q8_K_XL GGUF)
  description: 'Type: MoE (gemma-4 A4B)

    Parameter: 26B total / 4B active

    Source: https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: UD-Q8_K_XL

    KV Cache: F16 @ 262144

    Context window: 262144

    VRAM: 33.9 GiB of 47.8 (13.9 GiB free) — measured 2026-09-08 at F16

    Chat template: Gemma (--jinja)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/gemma-4-26B-A4B-it/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -b 4096 -ub 2048

    --spec-type ngram-mod

    -c 262144

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --top-k 64

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.07
    output: 0.34
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/gemma-4-26B-A4B-it/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `2048` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--spec-type` | `ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--top-k` | `64` | Default top-k sampling. |
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
| pp @9k | 8666 tok/s |
| pp @max (254k ctx) | 2621 tok/s |
| tg @9k (shallow) | 126.6 tok/s |
| tg @max (254k ctx) | 55.1 tok/s |

## Notes, incidents, upstream references

- n-gram speculative decoding: +209% verbatim-repetitive decode, -2.7% JSON (measured 2026-08-27).
