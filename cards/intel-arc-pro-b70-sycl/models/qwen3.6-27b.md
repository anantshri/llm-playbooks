# qwen3.6-27b
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.6 27B (UD-Q4_K_XL, SYCL, F16 KV, MTP, 131k)

## Model card summary (from the deployed config)

- Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | MTP | no vision
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Env: GGML_SYCL_FA_DPAS_F16=1 + GGML_SYCL_FA_DPAS_Q80_KG=4 (dpas decode arms, via ${llama-server-qwen36-bin})
- Quant: UD-Q4_K_XL (16.7 GiB)
- KV Cache: q8_0 @ 131072 (~4.5 GiB; was F16 8.0 GiB — 2026-11-14, intelligence-gated; 16 main full-attn layers x ~285 MiB)
- Context window: 131072 (native 262144)
- Chat template: Qwen (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.3, 'output': 2.0, 'cached': 0.03}`

## Config block

```yaml
qwen3.6-27b:
  name: Qwen3.6 27B (UD-Q4_K_XL, SYCL, F16 KV, MTP, 131k)
  description: 'Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | MTP | no vision

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF

    Engine: SYCL (Level Zero)

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Env: GGML_SYCL_FA_DPAS_F16=1 + GGML_SYCL_FA_DPAS_Q80_KG=4 (dpas decode arms, via ${llama-server-qwen36-bin})

    Quant: UD-Q4_K_XL (16.7 GiB)

    KV Cache: q8_0 @ 131072 (~4.5 GiB; was F16 8.0 GiB — 2026-11-14, intelligence-gated; 16 main full-attn
    layers x ~285 MiB)

    Context window: 131072 (native 262144)

    Chat template: Qwen (--jinja; reasoning parsed)

    '
  cmd: '${llama-server-qwen36-bin}

    -m ${gguf-models}/Qwen3.6-27B-MTP/Qwen3.6-27B-UD-Q4_K_XL.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -b 4096 -ub 2048

    # 2026-11-14: KV F16 -> q8_0, same intelligence gate as qwen3.8 above

    # (byte-identical battery outputs; ppl within noise). Frees ~4 GiB.

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --spec-type draft-mtp

    --spec-draft-n-max 4

    --reasoning-format deepseek

    --chat-template-kwargs ''{"preserve_thinking":true}''

    --top-k 20

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.3
    output: 2.0
    cached: 0.03
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Qwen3.6-27B-MTP/Qwen3.6-27B-UD-Q4_K_XL.gguf` | Weights file (GGUF) this entry serves. |
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
| `--spec-type` | `draft-mtp` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `4` | Max tokens speculated per step. |
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
| pp @9k | 1014 tok/s |
| pp @max (126k ctx) | 387 tok/s |
| tg @9k (shallow) | 29.7 tok/s |
| tg @max (126k ctx) | 8.5 tok/s |

## Notes, incidents, upstream references

- ggml-org/llama.cpp#32486 dpas-F16 arm: served MTP TG @~125k +37.0% median (2026-11-14).
- `ggml-org/llama.cpp#32486` (open) - dpas F16-KV decode arm: served MTP TG @~125k +37.0% (2026-11-14).
