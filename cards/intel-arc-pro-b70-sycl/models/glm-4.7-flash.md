# glm-4.7-flash
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** GLM-4.7 Flash (UD-Q4_K_XL, SYCL, F16 KV, 131k)

## Model card summary (from the deployed config)

- Type: MoE with MLA attention (deepseek2 arch, 64 experts / 4 active, 47 blocks)
- Parameter: GGUF size label 64x2.6B, 4 experts active per token
- Source: https://huggingface.co/unsloth/GLM-4.7-Flash-GGUF
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Quant: UD-Q4_K_XL (15.9 GiB)
- KV Cache: F16 @ 131072 (6.6 GiB)
- Context window: 131072 (native 202752)
- Chat template: GLM (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.0605, 'output': 0.4}`

## Config block

```yaml
glm-4.7-flash:
  name: GLM-4.7 Flash (UD-Q4_K_XL, SYCL, F16 KV, 131k)
  description: 'Type: MoE with MLA attention (deepseek2 arch, 64 experts / 4 active, 47 blocks)

    Parameter: GGUF size label 64x2.6B, 4 experts active per token

    Source: https://huggingface.co/unsloth/GLM-4.7-Flash-GGUF

    Engine: SYCL (Level Zero)

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Quant: UD-Q4_K_XL (15.9 GiB)

    KV Cache: F16 @ 131072 (6.6 GiB)

    Context window: 131072 (native 202752)

    Chat template: GLM (--jinja; reasoning parsed)

    '
  cmd: '${llama-server-sycl-bin}

    -m ${gguf-models}/GLM-4.7-Flash/GLM-4.7-Flash-UD-Q4_K_XL.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -b 4096 -ub 1024

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --spec-type ngram-mod

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
| `-m` | `${gguf-models}/GLM-4.7-Flash/GLM-4.7-Flash-UD-Q4_K_XL.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
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

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). Card-specific semantics: llama-bench `tg` is depth-independent (near-empty prompt); only tg@max (bench-mx walk to ~98% of -c) is a true deep-decode number. MoE entries lose ~40% prefill in llama-bench vs the serving path (expert locality).

| Metric | Value |
|---|---|
| pp @9k | 383 tok/s |
| pp @max (126k ctx) | 32.3 tok/s |
| tg @9k (shallow) | 28.8 tok/s |
| tg @max (126k ctx) | 6.6 tok/s |

## Notes, incidents, upstream references

- Long-context prefill historically collapsed on this card (pp@max 32 tok/s, measured 2026-08-27/28). Root cause: MLA shape (576/512, GQA-20) fell back to the TILE kernel - fixed by ggml-org/llama.cpp#29171 (merged 2026-10-07: pp@8192 3.01x, pp@64000 5.64x on this card). The fleet branch carried the fix from 2026-09-20 (pp8192 +270% / pp64k +809% vs the prior build). Re-measure on your build before trusting either number.
- Related merged PRs: #28918 (MKL-FA softmax coalesce, pp@126k +49%), #28931 (MMVQ GLU fusions).
- `ggml-org/llama.cpp#29171` (merged 2026-10-07) - This entry's MLA shape (576/512, GQA-20) had no SYCL flash-attention path and fell back to a slow TILE kernel for prefill. This PR routes the shape through MKL-FA: pp@8192 3.01x, pp@64000 5.64x on this card; decode unchanged.
