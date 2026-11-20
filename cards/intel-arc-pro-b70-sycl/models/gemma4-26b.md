# gemma4-26b
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Gemma 4 26B-A4B (UD-Q6_K_XL, SYCL, F16 KV, 131k)

## Model card summary (from the deployed config)

- Type: MoE (128 experts / 8 active, sliding-window + global attn) | text-only here
- Parameter: 26B total / 4B active
- Source: https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Quant: UD-Q6_K_XL (20.5 GiB)
- KV Cache: F16 @ 131072 (2.2 GiB)
- Context window: 131072 (native 262144)
- Chat template: Gemma (--jinja; no reasoning parser)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.07, 'output': 0.34}`

## Config block

```yaml
gemma4-26b:
  name: Gemma 4 26B-A4B (UD-Q6_K_XL, SYCL, F16 KV, 131k)
  description: 'Type: MoE (128 experts / 8 active, sliding-window + global attn) | text-only here

    Parameter: 26B total / 4B active

    Source: https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF

    Engine: SYCL (Level Zero)

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Quant: UD-Q6_K_XL (20.5 GiB)

    KV Cache: F16 @ 131072 (2.2 GiB)

    Context window: 131072 (native 262144)

    Chat template: Gemma (--jinja; no reasoning parser)

    '
  cmd: '${llama-server-sycl-bin}

    -m ${gguf-models}/gemma-4-26B-A4B-it/gemma-4-26B-A4B-it-UD-Q6_K_XL.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -b 4096 -ub 2048

    # 2026-11-14: KV F16 -> q8_0, same intelligence gate as qwen3.8 above

    # (byte-identical battery outputs; ppl within noise). Frees ~4 GiB.

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --spec-type ngram-mod

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
| `-m` | `${gguf-models}/gemma-4-26B-A4B-it/gemma-4-26B-A4B-it-UD-Q6_K_XL.gguf` | Weights file (GGUF) this entry serves. |
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
| `--spec-type` | `ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
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

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). Card-specific semantics: llama-bench `tg` is depth-independent (near-empty prompt); only tg@max (bench-mx walk to ~98% of -c) is a true deep-decode number. MoE entries lose ~40% prefill in llama-bench vs the serving path (expert locality).

| Metric | Value |
|---|---|
| pp @9k | 1669 tok/s |
| pp @max (126k ctx) | 1117 tok/s |
| tg @9k (shallow) | 34.7 tok/s |
| tg @max (126k ctx) | 18.3 tok/s |
