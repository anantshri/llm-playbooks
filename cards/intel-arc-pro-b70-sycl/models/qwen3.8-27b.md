# qwen3.8-27b
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.8 27B (UD-Q4_K_XL, SYCL, F16 KV, MTP, vision, 131k)

## Model card summary (from the deployed config)

- Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | Vision | MTP
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/unsloth/Qwen3.8-27B-GGUF
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16; XMX max 8)
- Env: GGML_SYCL_FA_DPAS_F16=1 + GGML_SYCL_FA_DPAS_Q80_KG=4 (dpas decode arms, via ${llama-server-qwen38-bin})
- Quant: UD-Q4_K_XL (16.7 GiB) + mmproj-F16 (0.86 GiB)
- KV Cache: q8_0 @ 131072 (~4.5 GiB; was F16 8.0 GiB — 2026-11-14, intelligence-gated)
- Context window: 131072 (native 262144)
- Chat template: Qwen (--jinja; reasoning parsed; thinking aliases :nothink · :medium)

## llama-swap entry

- **ttl:** `600`
- **filters:** `{'setParamsByID': {'${MODEL_ID}:nothink': {'chat_template_kwargs.enable_thinking': False}, '${MODEL_ID}:medium': {'chat_template_kwargs.reasoning_effort': 'medium'}}}`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
qwen3.8-27b:
  name: Qwen3.8 27B (UD-Q4_K_XL, SYCL, F16 KV, MTP, vision, 131k)
  description: 'Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | Vision | MTP

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/unsloth/Qwen3.8-27B-GGUF

    Engine: SYCL (Level Zero)

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16; XMX
    max 8)

    Env: GGML_SYCL_FA_DPAS_F16=1 + GGML_SYCL_FA_DPAS_Q80_KG=4 (dpas decode arms, via ${llama-server-qwen38-bin})

    Quant: UD-Q4_K_XL (16.7 GiB) + mmproj-F16 (0.86 GiB)

    KV Cache: q8_0 @ 131072 (~4.5 GiB; was F16 8.0 GiB — 2026-11-14, intelligence-gated)

    Context window: 131072 (native 262144)

    Chat template: Qwen (--jinja; reasoning parsed; thinking aliases :nothink · :medium)

    '
  cmd: '${llama-server-qwen38-bin}

    -m ${gguf-models}/Qwen3.8-27B/Qwen3.8-27B-UD-Q4_K_XL.gguf

    --mmproj ${gguf-models}/Qwen3.8-27B/mmproj-F16.gguf

    --image-min-tokens 1024

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -b 4096 -ub 1024

    # 2026-11-14: KV F16 -> q8_0. Intelligence gate PASSED: greedy outputs

    # byte-identical to F16 across a 9-task ~118k battery (needles, verbatim

    # code copy, multi-hop arithmetic, aggregation, state update, registry,

    # summary — intel-f16/intel-q8_0.json); wiki ppl final estimate

    # 6.2696 (q8_0) vs 6.2694/6.2695 (F16, two runs) — within noise.

    # q8_0 also engages the fattn-dpas q8_0 arms (incl. the fully-utilized

    # NR16=2 verify arm). KV bytes/token halve: 8.0 -> ~4.5 GiB @131k.

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --spec-type draft-mtp,ngram-mod

    --spec-draft-n-max 3

    --spec-draft-p-min 0.5

    --reasoning-format deepseek

    --chat-template-kwargs ''{"preserve_thinking":true}''

    --top-k 20

    --min-p 0.0

    '
  filters:
    setParamsByID:
      ${MODEL_ID}:nothink:
        chat_template_kwargs.enable_thinking: false
      ${MODEL_ID}:medium:
        chat_template_kwargs.reasoning_effort: medium
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
| `-m` | `${gguf-models}/Qwen3.8-27B/Qwen3.8-27B-UD-Q4_K_XL.gguf` | Weights file (GGUF) this entry serves. |
| `--mmproj` | `${gguf-models}/Qwen3.8-27B/mmproj-F16.gguf` | Vision projector file - enables image input for this entry. |
| `--image-min-tokens` | `1024` | Minimum tokens budgeted per image (vision quality floor). |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `draft-mtp,ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `3` | Max tokens speculated per step. |
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
| `filters` | Gateway request filters applied before proxying to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Measured (llama-bench)

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). Card-specific semantics: llama-bench `tg` is depth-independent (near-empty prompt); only tg@max (bench-mx walk to ~98% of -c) is a true deep-decode number. MoE entries lose ~40% prefill in llama-bench vs the serving path (expert locality).

| Metric | Value |
|---|---|
| pp @9k | 1013 tok/s |
| pp @max (126k ctx) | 403 tok/s |
| tg @9k (shallow) | 24.0 tok/s |
| tg @max (126k ctx) | 10.3 tok/s |

## Notes, incidents, upstream references

- Weights UD-Q4_K_XL 16.7 GiB + mmproj-F16 0.86 GiB; KV q8_0 @131k ~4.5 GiB (was F16 8.0 GiB - 2026-11-14).
- ggml-org/llama.cpp#32486 dpas arms: served MTP TG at ~125k +39.4% median (2026-11-14); env knobs in the dedicated wrapper, XMX max 8.
- Vision + MTP. The only non-llama.cpp text path on this card is the vllm sibling entry - use it when clients send huge tool schemas (llama.cpp grammar limit).
- `ggml-org/llama.cpp#32486` (open) - F16-KV + re-tiled Q8_0 decode arms for the dpas attention kernel. The env knobs this entry's wrapper sets come from this work: served MTP TG @~125k +39.4% (2026-11-14).
