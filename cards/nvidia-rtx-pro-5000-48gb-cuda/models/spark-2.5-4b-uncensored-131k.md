# spark-2.5-4b-uncensored-131k
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** UNCENSORED Spark X2.5 4B (Q8_0, CUDA, F16 KV, 131k) — co-residency worker arm

## Model card summary (from the deployed config)

- Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on
- Parameter: 4B
- Source: https://huggingface.co/toandev/Spark-X2.5-4B-Uncensored-GGUF (revision 8d13862e2cfa9e1e67b1f19bf17759f41bd583f2)
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: Q8_0, author quant (4.73 GB)
- KV Cache: F16 @ 131072 (~2-4 GiB, SWA-hybrid)
- Context window: 131072 (native 1048576)
- Safety gate: insights-db.paloaltonetworks.com — Safe, scanned 2026-09-09,
- version 8d1386 — verdict covers the EXACT revision pinned here.
- Provenance: toandev OFF the allowlist — repo-scoped approval, THIS REPO ONLY
- (user directive 2026-10-03).
- Purpose: cores-worker arm for the q36-unc25 bench combo (user directive
- 2026-11-11). Flags cloned 1:1 from spark-2.5-4b-uncensored.
- Chat template: embedded (--jinja)
- VRAM: 9644 MiB measured at load (2026-11-12, co-resident with the
- qwen3.6-27b-131k-q8kv coordinator arm)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.05, 'output': 0.1}`

## Config block

```yaml
spark-2.5-4b-uncensored-131k:
  name: UNCENSORED Spark X2.5 4B (Q8_0, CUDA, F16 KV, 131k) — co-residency worker arm
  description: "Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on\nParameter:\
    \ 4B\nSource: https://huggingface.co/toandev/Spark-X2.5-4B-Uncensored-GGUF (revision 8d13862e2cfa9e1e67b1f19bf17759f41bd583f2)\n\
    Engine: CUDA (sm_120)\nInference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)\nQuant:\
    \ Q8_0, author quant (4.73 GB)\nKV Cache: F16 @ 131072 (~2-4 GiB, SWA-hybrid)\nContext window: 131072\
    \ (native 1048576)\nSafety gate: insights-db.paloaltonetworks.com — Safe, scanned 2026-09-09,\n  version\
    \ 8d1386 — verdict covers the EXACT revision pinned here.\nProvenance: toandev OFF the allowlist —\
    \ repo-scoped approval, THIS REPO ONLY\n  (user directive 2026-10-03).\nPurpose: cores-worker arm\
    \ for the q36-unc25 bench combo (user directive\n  2026-11-11). Flags cloned 1:1 from spark-2.5-4b-uncensored.\n\
    Chat template: embedded (--jinja)\nVRAM: 9644 MiB measured at load (2026-11-12, co-resident with the\n\
    \  qwen3.6-27b-131k-q8kv coordinator arm)\n"
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Spark-X2.5-4B-Uncensored/Spark-X2.5-4B-Uncensored-Q8_0.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --top-k 0

    --min-p 0.0

    --reasoning-format deepseek

    '
  ttl: 600
  pricing:
    input: 0.05
    output: 0.1
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Spark-X2.5-4B-Uncensored/Spark-X2.5-4B-Uncensored-Q8_0.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--top-k` | `0` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |
