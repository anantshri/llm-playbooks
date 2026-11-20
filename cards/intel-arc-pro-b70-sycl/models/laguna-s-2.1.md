# laguna-s-2.1
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Laguna S 2.1 (118B-A8B MoE, Q4_K_M, SYCL, F16 KV, 65k, n-cpu-moe 46)

## Model card summary (from the deployed config)

- Type: MoE (laguna arch, 256 experts / 10 active)
- Parameter: 118B total / 8B active
- Source: https://huggingface.co/poolside/Laguna-S-2.1-GGUF
- Engine: SYCL (Level Zero) — attention + GPU: 18 expert layers, RAM: 46 expert layers in page cache
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Quant: Q4_K_M (63.5 GiB)
- KV Cache: F16 @ 65536 (12 GiB)
- Context window: 65536 (native 1048576)
- Chat template: poolside-Laguna-S-2.1.jinja

## llama-swap entry

- **ttl:** `3600`
- **pricing:** `{'input': 0.09, 'output': 0.18, 'cached': 0.009}`

## Config block

```yaml
laguna-s-2.1:
  name: Laguna S 2.1 (118B-A8B MoE, Q4_K_M, SYCL, F16 KV, 65k, n-cpu-moe 46)
  description: 'Type: MoE (laguna arch, 256 experts / 10 active)

    Parameter: 118B total / 8B active

    Source: https://huggingface.co/poolside/Laguna-S-2.1-GGUF

    Engine: SYCL (Level Zero) — attention + GPU: 18 expert layers, RAM: 46 expert layers in page cache

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Quant: Q4_K_M (63.5 GiB)

    KV Cache: F16 @ 65536 (12 GiB)

    Context window: 65536 (native 1048576)

    Chat template: poolside-Laguna-S-2.1.jinja

    '
  cmd: '${llama-server-sycl-bin}

    -m ${gguf-models}/Laguna-S-2.1/laguna-s-2.1-Q4_K_M.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -np 1

    --n-cpu-moe 46

    -c 65536

    -b 16384 -ub 4096

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --chat-template-file /opt/inference/llama.cpp-src/models/templates/poolside-Laguna-S-2.1.jinja

    --reasoning-format deepseek

    --min-p 0.0

    '
  ttl: 3600
  pricing:
    input: 0.09
    output: 0.18
    cached: 0.009
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Laguna-S-2.1/laguna-s-2.1-Q4_K_M.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-np` | `1` | Concurrent sequence slots served by this backend. |
| `--n-cpu-moe` | `46` | Keep N MoE expert layers on CPU RAM - the VRAM-fit lever for oversized MoE models; costs host RAM and prefill speed. |
| `-c` | `65536` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `16384` | Logical batch size (prompt-processing chunk). |
| `-ub` | `4096` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--chat-template-file` | `/opt/inference/llama.cpp-src/models/templates/poolside-Laguna-S-2.1.jinja` | Custom Jinja chat template file, overriding the one embedded in the GGUF. |
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
| pp @9k | 409 tok/s |
| pp @max (60k ctx) | 392 tok/s |
| tg @9k (shallow) | 13.3 tok/s |
| tg @max (60k ctx) | 10.5 tok/s |

## Notes, incidents, upstream references

- 118B-A8B MoE run with --n-cpu-moe 46 (experts paged from host RAM). VRAM ceiling caps context at 65536 here; ~10-13 tok/s.
