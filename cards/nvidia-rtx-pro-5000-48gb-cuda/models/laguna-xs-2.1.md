# laguna-xs-2.1
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Laguna XS 2.1 (33B-A3B MoE, Q8_0, 262K)

## Model card summary (from the deployed config)

- Type: MoE (laguna arch; 30/40 layers sliding-window @512) | reasoning
- Parameter: 33B total / 3B active
- Source: https://huggingface.co/poolside/Laguna-XS-2.1-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: self-quant Q8_0 (33 GB)
- KV Cache: F16 @ 262144
- Context window: 262144 (native, fits fully in VRAM)
- Chat template: Laguna (--jinja; reasoning parsed)
- VRAM: ~43.8 GB, ~4 GB free

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.06, 'output': 0.12, 'cached': 0.03}`

## Config block

```yaml
laguna-xs-2.1:
  name: Laguna XS 2.1 (33B-A3B MoE, Q8_0, 262K)
  description: 'Type: MoE (laguna arch; 30/40 layers sliding-window @512) | reasoning

    Parameter: 33B total / 3B active

    Source: https://huggingface.co/poolside/Laguna-XS-2.1-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: self-quant Q8_0 (33 GB)

    KV Cache: F16 @ 262144

    Context window: 262144 (native, fits fully in VRAM)

    Chat template: Laguna (--jinja; reasoning parsed)

    VRAM: ~43.8 GB, ~4 GB free

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Laguna-XS-2.1/Laguna-XS-2.1-Q8_0.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -np 1

    -b 4096 -ub 1024

    -c 262144

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --spec-type ngram-mod

    --reasoning-format deepseek

    --top-k 20

    --top-p 1.0

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.06
    output: 0.12
    cached: 0.03
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Laguna-XS-2.1/Laguna-XS-2.1-Q8_0.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-np` | `1` | Concurrent sequence slots served by this backend. |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--top-k` | `20` | Default top-k sampling. |
| `--top-p` | `1.0` | Default top-p sampling. |
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
| pp @9k | 8451 tok/s |
| pp @max (247k ctx) | 1906 tok/s |
| tg @9k (shallow) | 148 tok/s |
| tg @max (247k ctx) | 46 tok/s |

## Notes, incidents, upstream references

- Default agentic-coding pick on this card: 148 tok/s @9k, 262k ctx, tool calls dodge the llama.cpp grammar limit.
