# thomson-1.0-small
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Thomson 1.0 Small (Q8_0 GGUF + vision, F16 KV, 131k)

## Model card summary (from the deployed config)

- Type: MoE (qwen3_5_moe, 40 layers, hybrid 3:1 linear/full-attn, 10 KV layers) | Vision | legal/tax/journalism specialist
- Parameter: 35B total / 3B active
- Source: https://huggingface.co/thomsonreuters/Thomson-1.0-Small
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: self-converted Q8_0 + mmproj-F16 (~0.9 GB)
- KV Cache: F16 @ 131072 (~2.5 GiB)
- Context window: 131072 (native 262144)
- Chat template: Qwen (--jinja; reasoning parsed)
- VRAM: ~47 GB of 48 (footprint unchanged vs q8_0 @262k — F16 KV at half ctx is the same block size)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.08, 'output': 0.45, 'cached': 0.04}`

## Config block

```yaml
thomson-1.0-small:
  name: Thomson 1.0 Small (Q8_0 GGUF + vision, F16 KV, 131k)
  description: 'Type: MoE (qwen3_5_moe, 40 layers, hybrid 3:1 linear/full-attn, 10 KV layers) | Vision
    | legal/tax/journalism specialist

    Parameter: 35B total / 3B active

    Source: https://huggingface.co/thomsonreuters/Thomson-1.0-Small

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: self-converted Q8_0 + mmproj-F16 (~0.9 GB)

    KV Cache: F16 @ 131072 (~2.5 GiB)

    Context window: 131072 (native 262144)

    Chat template: Qwen (--jinja; reasoning parsed)

    VRAM: ~47 GB of 48 (footprint unchanged vs q8_0 @262k — F16 KV at half ctx is the same block size)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Thomson-1.0-Small/current.gguf

    --mmproj ${gguf-models}/Thomson-1.0-Small/mmproj-F16.gguf

    --image-min-tokens 1024

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --reasoning-format deepseek

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.08
    output: 0.45
    cached: 0.04
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Thomson-1.0-Small/current.gguf` | Weights file (GGUF) this entry serves. |
| `--mmproj` | `${gguf-models}/Thomson-1.0-Small/mmproj-F16.gguf` | Vision projector file - enables image input for this entry. |
| `--image-min-tokens` | `1024` | Minimum tokens budgeted per image (vision quality floor). |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Measured (llama-bench)

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). tg@9k is shallow decode; tg@max is the deep-decode walk (~98% of -c). Do not mix with gateway measurements (see troubleshooting).

| Metric | Value |
|---|---|
| pp @9k | 6627 tok/s |
| pp @max (254k ctx) | 3336 tok/s |
| tg @9k (shallow) | 144.8 tok/s |
| tg @max (254k ctx) | 64.1 tok/s |
