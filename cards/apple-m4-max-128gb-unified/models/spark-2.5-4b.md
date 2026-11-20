# spark-2.5-4b
**Card:** Apple M4 Max (128 GB unified memory)  |  **Engine:** llama.cpp

**Display name:** Spark X2.5 4B (Q8_0, Metal, F16 KV, 262k)

## Model card summary (from the deployed config)

- Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on
- Parameter: 4B
- Source: https://huggingface.co/XHToken/Spark-X2.5-4B-GGUF
- Engine: Metal
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: Q8_0, author quant (4.07 GB) — was BF16 to 09-13
- KV Cache: F16 @ 102400
- Batch: -b 512 -ub 128 (measured laptop profile)
- Context window: 102400 (native 1048576; reduced from 262144 on 2026-10-03 per user directive)
- Chat template: embedded (--jinja)
- Reasoning: off by default; reasoning_effort low/med/high opts in (budget 512/2048/8192)

## llama-swap entry

- **ttl:** `600`
- **capabilities:** `{'max_output_tokens': 32768, 'reasoning': {'default': 'none', 'efforts': {'none': 0, 'low': 512, 'medium': 2048, 'high': 8192}}}`

## Config block

```yaml
spark-2.5-4b:
  name: Spark X2.5 4B (Q8_0, Metal, F16 KV, 262k)
  description: 'Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on

    Parameter: 4B

    Source: https://huggingface.co/XHToken/Spark-X2.5-4B-GGUF

    Engine: Metal

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: Q8_0, author quant (4.07 GB) — was BF16 to 09-13

    KV Cache: F16 @ 102400

    Batch: -b 512 -ub 128 (measured laptop profile)

    Context window: 102400 (native 1048576; reduced from 262144 on 2026-10-03 per user directive)

    Chat template: embedded (--jinja)

    Reasoning: off by default; reasoning_effort low/med/high opts in (budget 512/2048/8192)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Spark-X2.5-4B/Spark-X2.5-4B-Q8_0.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 102400

    --cache-type-k f16 --cache-type-v f16

    -b 512 -ub 128

    --flash-attn on

    --jinja

    --chat-template-kwargs ''{"enable_thinking":false}''

    '
  capabilities:
    max_output_tokens: 32768
    reasoning:
      default: none
      efforts:
        none: 0
        low: 512
        medium: 2048
        high: 8192
  ttl: 600
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Spark-X2.5-4B/Spark-X2.5-4B-Q8_0.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `102400` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `-b` | `512` | Logical batch size (prompt-processing chunk). |
| `-ub` | `128` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--chat-template-kwargs` | `'{"enable_thinking":false}'` | JSON kwargs passed to the chat template (deprecated flag form). Used here to force thinking off server-side. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `capabilities` | Gateway metadata for what this entry supports (e.g. reasoning). Requests using an unsupported feature are rejected up-front instead of mis-served. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |

## Measured (llama-bench)

> llama.cpp rows are 2026-08-27 era; MLX A/B rows 2026-09-03 (taken on a dirty box - depressed, see troubleshooting). Clean-box re-measure 2026-09-30 showed one MLX row 4.8x off its dirty-box value; environment moved more than engine choice.

| Metric | Value |
|---|---|
| pp @9k | 1030.5 tok/s |
| pp @max (256k ctx) | — tok/s |
| tg @9k (shallow) | 31.3 tok/s |
| tg @max (256k ctx) | — tok/s |

## Notes, incidents, upstream references

- Q8_0 (not BF16) at 102400 ctx as of 2026-10-03. Thinking forced OFF via --chat-template-kwargs (deprecated form), so trivial arithmetic fails out of the box - send reasoning_effort when needed.
