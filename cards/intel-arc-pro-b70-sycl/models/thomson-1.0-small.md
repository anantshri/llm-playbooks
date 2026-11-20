# thomson-1.0-small
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Thomson 1.0 Small (Q4_K_M, SYCL, F16 KV, in-VRAM, 131k)

## Model card summary (from the deployed config)

- Type: MoE (qwen3_5_moe, 256 experts / 8 active, hybrid attn) | no vision here
- Parameter: 35B total / 3B active
- Source: https://huggingface.co/thomsonreuters/Thomson-1.0-Small
- Engine: SYCL (Level Zero) — fully on GPU
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Quant: self-converted Q4_K_M (~20.5 GiB)
- KV Cache: F16 @ 131072 (~2.5 GiB)
- Context window: 131072 (native 262144)
- Chat template: Qwen (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.08, 'output': 0.45, 'cached': 0.04}`

## Config block

```yaml
thomson-1.0-small:
  name: Thomson 1.0 Small (Q4_K_M, SYCL, F16 KV, in-VRAM, 131k)
  description: 'Type: MoE (qwen3_5_moe, 256 experts / 8 active, hybrid attn) | no vision here

    Parameter: 35B total / 3B active

    Source: https://huggingface.co/thomsonreuters/Thomson-1.0-Small

    Engine: SYCL (Level Zero) — fully on GPU

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Quant: self-converted Q4_K_M (~20.5 GiB)

    KV Cache: F16 @ 131072 (~2.5 GiB)

    Context window: 131072 (native 262144)

    Chat template: Qwen (--jinja; reasoning parsed)

    '
  cmd: '${llama-server-sycl-bin}

    -m ${gguf-models}/Thomson-1.0-Small/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -np 1

    -c 131072

    -b 4096 -ub 2048

    # 2026-11-14: KV F16 -> q8_0, same intelligence gate as qwen3.8 above

    # (byte-identical battery outputs; ppl within noise). Frees ~4 GiB.

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --reasoning-format deepseek

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.08
    output: 0.45
    cached: 0.04
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Thomson-1.0-Small/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-np` | `1` | Concurrent sequence slots served by this backend. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `2048` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Measured (llama-bench)

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). Card-specific semantics: llama-bench `tg` is depth-independent (near-empty prompt); only tg@max (bench-mx walk to ~98% of -c) is a true deep-decode number. MoE entries lose ~40% prefill in llama-bench vs the serving path (expert locality).

| Metric | Value |
|---|---|
| pp @9k | 2081 tok/s |
| pp @max (126k ctx) | 1006 tok/s |
| tg @9k (shallow) | 80.6 tok/s |
| tg @max (126k ctx) | 28.3 tok/s |

## Notes, incidents, upstream references

- Legal/tax/journalism specialist. Q4_K_M fully in VRAM. License is polyform-strict: internal use only, no redistribution.
