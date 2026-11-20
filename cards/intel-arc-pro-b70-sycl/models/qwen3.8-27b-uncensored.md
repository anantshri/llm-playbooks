# qwen3.8-27b-uncensored
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.8 27B Uncensored — Q4_K_M, q8_0 KV, MTP+ngram, 229376, text-only (abliterated)

## Model card summary (from the deployed config)

- Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | abliterated | Vision
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/orcarouter/Qwen3.8-27B-Uncensored-GGUF
- Engine: SYCL (Level Zero) — fully in VRAM, no expert split
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 incl. Q80_KG retile; binary = lc-q80-tune)
- Env: GGML_SYCL_FA_DPAS_F16=1 + GGML_SYCL_FA_DPAS_Q80_KG=4 (dpas decode arms, via ${llama-server-unc-bin})
- Quant: Q4_K_M author quant (15.66 GiB)
- Vision: REMOVED 2026-11-15 (mmproj dropped — text-only; frees 0.87 GiB + image-burst reserve)
- KV Cache: q8_0 @ 229376 (~4.9 GiB; was F16 @ 131072 = 8.6 GiB)
- Context window: 229376 (native 262144; 224k = ladder-measured comfort ceiling, text-only)
- Chat template: Qwen (--jinja; reasoning parsed, thinking default-on)
- VRAM: 25.3 GiB of 31.9 (6.6 GiB free) — measured 2026-09-09
- Sampling: top_k 20 / min_p 0.0 (card names none) — no rep-penalty, see NOTES
- Measured: 28.0 tok/s decode; clean 5-arg tool call (shallow, 2026-09-09)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
qwen3.8-27b-uncensored:
  name: Qwen3.8 27B Uncensored — Q4_K_M, q8_0 KV, MTP+ngram, 229376, text-only (abliterated)
  description: 'Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | abliterated | Vision

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/orcarouter/Qwen3.8-27B-Uncensored-GGUF

    Engine: SYCL (Level Zero) — fully in VRAM, no expert split

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 incl.
    Q80_KG retile; binary = lc-q80-tune)

    Env: GGML_SYCL_FA_DPAS_F16=1 + GGML_SYCL_FA_DPAS_Q80_KG=4 (dpas decode arms, via ${llama-server-unc-bin})

    Quant: Q4_K_M author quant (15.66 GiB)

    Vision: REMOVED 2026-11-15 (mmproj dropped — text-only; frees 0.87 GiB + image-burst reserve)

    KV Cache: q8_0 @ 229376 (~4.9 GiB; was F16 @ 131072 = 8.6 GiB)

    Context window: 229376 (native 262144; 224k = ladder-measured comfort ceiling, text-only)

    Chat template: Qwen (--jinja; reasoning parsed, thinking default-on)

    VRAM: 25.3 GiB of 31.9 (6.6 GiB free) — measured 2026-09-09

    Sampling: top_k 20 / min_p 0.0 (card names none) — no rep-penalty, see NOTES

    Measured: 28.0 tok/s decode; clean 5-arg tool call (shallow, 2026-09-09)

    '
  cmd: '${llama-server-unc-bin}

    -m ${gguf-models}/Qwen3.8-27B-Uncensored/current.gguf

    # 2026-11-15: vision removed (mmproj dropped) and KV F16 -> q8_0, -c

    # 131072 -> 229376 (224k). Ladder-measured: no-vision resident at 224k

    # = 29.4 GiB (2.5 GiB free); 256k native = 0.9 GiB free = brick-wall,

    # stays off. q8_0 engages the dpas q8_0 arms; KG=4 re-tile via env.

    # Intelligence gate precedent: byte-identical battery outputs

    # q8_0-vs-F16 on the stock sibling (2026-11-14).

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 229376

    -b 4096 -ub 1024

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --spec-type draft-mtp,ngram-mod

    --spec-draft-n-max 2

    --spec-draft-p-min 0.5

    --jinja

    --spec-type draft-mtp,ngram-mod

    --spec-draft-n-max 2

    --spec-draft-p-min 0.5

    --reasoning-format deepseek

    --chat-template-kwargs ''{"preserve_thinking":true}''

    --top-k 20

    --min-p 0.0

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
| `-m` | `${gguf-models}/Qwen3.8-27B-Uncensored/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `229376` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--spec-type` | `draft-mtp,ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `2` | Max tokens speculated per step. |
| `--spec-draft-p-min` | `0.5` | Draft acceptance probability threshold - tunes acceptance vs wasted drafts. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `draft-mtp,ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `2` | Max tokens speculated per step. |
| `--spec-draft-p-min` | `0.5` | Draft acceptance probability threshold - tunes acceptance vs wasted drafts. |
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

## Notes, incidents, upstream references

- 2026-11-15: moved to q8_0 KV at -c 229376 (224k measured comfort ceiling), vision/mmproj removed (text-only); base tg128@131k sweep F16 15.78 -> q8_0+KG4 22.22 t/s (+40.8%).
- Source repo has no insights-DB supply-chain scan record - deployed under a documented override; flagged, not cleared.
- `ggml-org/llama.cpp#32486` (open) - dpas Q8_0-KG=4 decode arm: tg128@131k 15.78 -> 22.22 t/s (+40.8%, 2026-11-15).
