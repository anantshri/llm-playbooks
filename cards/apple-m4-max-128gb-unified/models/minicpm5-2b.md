# minicpm5-2b
**Card:** Apple M4 Max (128 GB unified memory)  |  **Engine:** llama.cpp

**Display name:** MiniCPM5-2B (Q8_0 GGUF, thinking + tools)

## Model card summary (from the deployed config)

- Type: Dense (llama arch, 42 layers, GQA 16/2) | thinking default-on | tools
- Parameter: 2.5B total / 2.5B active (dense)
- Source: https://huggingface.co/openbmb/MiniCPM5-2B-GGUF
- Engine: Metal
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: Q8_0, author quant (2.5 GB)
- KV Cache: F16 @ 131072
- Batch: -b 512 -ub 128 (measured laptop profile)
- Context window: 131072 (native)
- Chat template: MiniCPM (--jinja; reasoning parsed)
- Sampling: temp 1.0 / top_p 0.95 (card) — min_p pinned 0.0
- Measured: 121.0 tok/s decode; clean tool call (2026-09-08, shallow)
- Reasoning: reasoning_effort low/med/high -> budget 512/2048/8192 | default uncapped

## llama-swap entry

- **ttl:** `600`
- **capabilities:** `{'max_output_tokens': 32768, 'reasoning': {'default': 'low', 'efforts': {'low': 512, 'medium': 2048, 'high': 8192}}}`

## Config block

```yaml
minicpm5-2b:
  name: MiniCPM5-2B (Q8_0 GGUF, thinking + tools)
  description: 'Type: Dense (llama arch, 42 layers, GQA 16/2) | thinking default-on | tools

    Parameter: 2.5B total / 2.5B active (dense)

    Source: https://huggingface.co/openbmb/MiniCPM5-2B-GGUF

    Engine: Metal

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: Q8_0, author quant (2.5 GB)

    KV Cache: F16 @ 131072

    Batch: -b 512 -ub 128 (measured laptop profile)

    Context window: 131072 (native)

    Chat template: MiniCPM (--jinja; reasoning parsed)

    Sampling: temp 1.0 / top_p 0.95 (card) — min_p pinned 0.0

    Measured: 121.0 tok/s decode; clean tool call (2026-09-08, shallow)

    Reasoning: reasoning_effort low/med/high -> budget 512/2048/8192 | default uncapped

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/MiniCPM5-2B/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    --cache-type-k f16 --cache-type-v f16

    -b 512 -ub 128

    --flash-attn on

    --jinja

    --reasoning-format deepseek

    --temp 1.0 --top-p 0.95 --min-p 0.0

    '
  capabilities:
    max_output_tokens: 32768
    reasoning:
      default: low
      efforts:
        low: 512
        medium: 2048
        high: 8192
  ttl: 600
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/MiniCPM5-2B/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `-b` | `512` | Logical batch size (prompt-processing chunk). |
| `-ub` | `128` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--temp` | `1.0` | Default sampling temperature (client per-request values override). |
| `--top-p` | `0.95` | Default top-p sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `capabilities` | Gateway metadata for what this entry supports (e.g. reasoning). Requests using an unsupported feature are rejected up-front instead of mis-served. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |

## Notes, incidents, upstream references

- 2B, 127.7 tok/s, thinking default-on, tools verified (2026-08-27 era measurement).
