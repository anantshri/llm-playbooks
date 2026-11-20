# qwen3.8-27b-obliterated
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.8 27B OBLITERATED (Q6_K, SYCL, q8_0 KV, MTP, abliterated, 131k)

## Model card summary (from the deployed config)

- Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | abliterated | MTP | no vision
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/OBLITERATUS/Qwen3.8-27B-OBLITERATED
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Env: GGML_SYCL_FA_DPAS_Q80_KG=4 (dpas q8_0 KG=4 re-tile, via ${llama-server-obl-bin}; +9.8% tg128@131k)
- Quant: Q6_K (20.9 GiB)
- KV Cache: Q8_0 @ 131072 (3.75 GiB)
- Context window: 131072 (native 262144)
- Sampling: temp 0 (greedy), repeat-penalty 1.15 (anti-loop)
- Chat template: Qwen (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
qwen3.8-27b-obliterated:
  name: Qwen3.8 27B OBLITERATED (Q6_K, SYCL, q8_0 KV, MTP, abliterated, 131k)
  description: 'Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | abliterated | MTP | no vision

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/OBLITERATUS/Qwen3.8-27B-OBLITERATED

    Engine: SYCL (Level Zero)

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Env: GGML_SYCL_FA_DPAS_Q80_KG=4 (dpas q8_0 KG=4 re-tile, via ${llama-server-obl-bin}; +9.8% tg128@131k)

    Quant: Q6_K (20.9 GiB)

    KV Cache: Q8_0 @ 131072 (3.75 GiB)

    Context window: 131072 (native 262144)

    Sampling: temp 0 (greedy), repeat-penalty 1.15 (anti-loop)

    Chat template: Qwen (--jinja; reasoning parsed)

    '
  cmd: '${llama-server-obl-bin}

    -m ${gguf-models}/Qwen3.8-27B-OBLITERATED/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -b 4096 -ub 2048

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --temp 0

    --repeat-penalty 1.15

    --repeat-last-n 512

    --top-k 0

    --top-p 1.0

    --min-p 0.0

    --spec-type draft-mtp

    --spec-draft-n-max 3

    --reasoning-format deepseek

    --chat-template-kwargs ''{"preserve_thinking":true}''

    '
  ttl: 600
  pricing:
    input: 0.42
    output: 3.0
    cached: 0.085
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Qwen3.8-27B-OBLITERATED/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `2048` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--temp` | `0` | Default sampling temperature (client per-request values override). |
| `--repeat-penalty` | `1.15` | Default repetition penalty. Abliterated entries loop without 1.1+ (model-card requirement). |
| `--repeat-last-n` | `512` | Window (tokens) the repetition penalty looks back over. |
| `--top-k` | `0` | Default top-k sampling. |
| `--top-p` | `1.0` | Default top-p sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |
| `--spec-type` | `draft-mtp` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `3` | Max tokens speculated per step. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--chat-template-kwargs` | `'{"preserve_thinking":true}'` | JSON kwargs passed to the chat template (deprecated flag form). Used here to force thinking off server-side. |

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
| pp @9k | 988 tok/s |
| pp @max (126k ctx) | 387 tok/s |
| tg @9k (shallow) | 43.0 tok/s |
| tg @max (126k ctx) | 7.5 tok/s |

## Notes, incidents, upstream references

- Model card calls repetition_penalty 1.15 essential to prevent loops; not set server-side - send per-request.
- dpas Q8_0-KG=4 arm: +9.8% (2026-11-15).
- `ggml-org/llama.cpp#32486` (open) - dpas Q8_0-KG=4 decode arm: +9.8% (2026-11-15).
