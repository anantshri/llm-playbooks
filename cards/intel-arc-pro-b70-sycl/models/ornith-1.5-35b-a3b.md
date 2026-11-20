# ornith-1.5-35b-a3b
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Ornith 1.5 35B-A3B (Q8_0, SYCL, F16 KV, vision, n-cpu-moe 12, 131k)

## Model card summary (from the deployed config)

- Type: MoE (qwen35moe, 256 experts / 8 active, hybrid attn) | Vision
- Parameter: 35B total / 3B active
- Source: https://huggingface.co/ornith-ai/Ornith-1.5-35B-A3B-GGUF
- Engine: SYCL (Level Zero) — GPU: 29 / RAM: 12 of 41 expert layers
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Quant: Q8_0 (33.5 GiB) + mmproj-F16 (0.9 GiB)
- KV Cache: F16 @ 131072 (2.4 GiB)
- Context window: 131072 (native 262144)
- Chat template: Qwen (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.1, 'output': 0.9, 'cached': 0.05}`

## Config block

```yaml
ornith-1.5-35b-a3b:
  name: Ornith 1.5 35B-A3B (Q8_0, SYCL, F16 KV, vision, n-cpu-moe 12, 131k)
  description: 'Type: MoE (qwen35moe, 256 experts / 8 active, hybrid attn) | Vision

    Parameter: 35B total / 3B active

    Source: https://huggingface.co/ornith-ai/Ornith-1.5-35B-A3B-GGUF

    Engine: SYCL (Level Zero) — GPU: 29 / RAM: 12 of 41 expert layers

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Quant: Q8_0 (33.5 GiB) + mmproj-F16 (0.9 GiB)

    KV Cache: F16 @ 131072 (2.4 GiB)

    Context window: 131072 (native 262144)

    Chat template: Qwen (--jinja; reasoning parsed)

    '
  cmd: '${llama-server-sycl-bin}

    -m ${gguf-models}/ornith-1.5-35b-a3b/current.gguf

    --mmproj ${gguf-models}/ornith-1.5-35b-a3b/mmproj-F16.gguf

    --image-min-tokens 1024

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -np 1

    --n-cpu-moe 12

    -c 131072

    -b 4096 -ub 1024

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --reasoning-format deepseek

    --chat-template-kwargs ''{"preserve_thinking":true}''

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
| `-m` | `${gguf-models}/ornith-1.5-35b-a3b/current.gguf` | Weights file (GGUF) this entry serves. |
| `--mmproj` | `${gguf-models}/ornith-1.5-35b-a3b/mmproj-F16.gguf` | Vision projector file - enables image input for this entry. |
| `--image-min-tokens` | `1024` | Minimum tokens budgeted per image (vision quality floor). |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-np` | `1` | Concurrent sequence slots served by this backend. |
| `--n-cpu-moe` | `12` | Keep N MoE expert layers on CPU RAM - the VRAM-fit lever for oversized MoE models; costs host RAM and prefill speed. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--chat-template-kwargs` | `'{"preserve_thinking":true}'` | JSON kwargs passed to the chat template (deprecated flag form). Used here to force thinking off server-side. |
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

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). Card-specific semantics: llama-bench `tg` is depth-independent (near-empty prompt); only tg@max (bench-mx walk to ~98% of -c) is a true deep-decode number. MoE entries lose ~40% prefill in llama-bench vs the serving path (expert locality).

| Metric | Value |
|---|---|
| pp @9k | 1295 tok/s |
| pp @max (126k ctx) | 808 tok/s |
| tg @9k (shallow) | 41.8 tok/s |
| tg @max (126k ctx) | 21.6 tok/s |

## Notes, incidents, upstream references

- MoE with --n-cpu-moe 12 of 41 experts on CPU; 131k context here.
