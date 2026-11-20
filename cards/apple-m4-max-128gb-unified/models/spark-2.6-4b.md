# spark-2.6-4b
**Card:** Apple M4 Max (128 GB unified memory)  |  **Engine:** llama.cpp

**Display name:** Spark X2.6 4B (Q8_0, Metal, F16 KV, 100k)

## Model card summary (from the deployed config)

- Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on
- Parameter: 4B
- Source: https://huggingface.co/XHToken/Spark-X2.6-4B-GGUF (revision 16c77db5a782d75d6953770339f7ae06be7dd3f7)
- Engine: Metal
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: Q8_0, author quant (4.38 GB)
- KV Cache: F16 @ 102400
- Batch: -b 512 -ub 128 (cloned from spark-2.5-4b)
- Context window: 102400 (native 1048576)
- Chat template: embedded (--jinja)
- Reasoning: ON by default at high (template thinking_effort=high, budget 8192); reasoning_effort none/low override (budget 0/512)

## llama-swap entry

- **ttl:** `600`
- **capabilities:** `{'max_output_tokens': 32768, 'reasoning': {'default': 'high', 'efforts': {'none': 0, 'low': 512, 'high': 8192}}}`

## Config block

```yaml
spark-2.6-4b:
  name: Spark X2.6 4B (Q8_0, Metal, F16 KV, 100k)
  description: 'Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on

    Parameter: 4B

    Source: https://huggingface.co/XHToken/Spark-X2.6-4B-GGUF (revision 16c77db5a782d75d6953770339f7ae06be7dd3f7)

    Engine: Metal

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: Q8_0, author quant (4.38 GB)

    KV Cache: F16 @ 102400

    Batch: -b 512 -ub 128 (cloned from spark-2.5-4b)

    Context window: 102400 (native 1048576)

    Chat template: embedded (--jinja)

    Reasoning: ON by default at high (template thinking_effort=high, budget 8192); reasoning_effort none/low
    override (budget 0/512)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Spark-X2.6-4B/Spark-X2.6-4B-Q8_0.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 102400

    --cache-type-k f16 --cache-type-v f16

    -b 512 -ub 128

    --flash-attn on

    --jinja

    --reasoning on

    --reasoning-effort high

    --reasoning-budget 8192

    '
  capabilities:
    max_output_tokens: 32768
    reasoning:
      default: high
      efforts:
        none: 0
        low: 512
        high: 8192
  ttl: 600
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Spark-X2.6-4B/Spark-X2.6-4B-Q8_0.gguf` | Weights file (GGUF) this entry serves. |
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
| `--reasoning` | `on` | Server-side thinking on/off switch. |
| `--reasoning-effort` | `high` | Server-side default thinking effort (clients can override per request). |
| `--reasoning-budget` | `8192` | Max tokens the model may spend thinking before answering. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `capabilities` | Gateway metadata for what this entry supports (e.g. reasoning). Requests using an unsupported feature are rejected up-front instead of mis-served. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |

## Notes, incidents, upstream references

- Thinks at HIGH by default (--reasoning on, effort high, budget 8192) unlike the 2.5 sibling. Give max_tokens headroom or send reasoning_effort none/low. Staged 2026-10-21; loads at next gateway restart; smoke owed. No 2.6 uncensored exists.
