# gemma4-31b-200k-nospec
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Gemma 4 31B-it — UD-Q8_K_XL, Q8_0 KV, 200000, NO spec decode (long-ctx sibling of gemma4-31b-mtp)

## Model card summary (from the deployed config)

- Type: MoE (gemma-4) | long-context variant of gemma4-31b-mtp — no MTP, no spec decode
- Source: https://huggingface.co/unsloth/gemma-4-31B-it-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: UD-Q8_K_XL
- KV Cache: Q8_0 @ 200000
- Context window: 200000 (vs 163840 on the drafter-carrying base)
- VRAM: ~44.8 GB, ~3 GB free

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.09, 'output': 0.34, 'cached': 0.05}`

## Config block

```yaml
gemma4-31b-200k-nospec:
  name: Gemma 4 31B-it — UD-Q8_K_XL, Q8_0 KV, 200000, NO spec decode (long-ctx sibling of gemma4-31b-mtp)
  description: 'Type: MoE (gemma-4) | long-context variant of gemma4-31b-mtp — no MTP, no spec decode

    Source: https://huggingface.co/unsloth/gemma-4-31B-it-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: UD-Q8_K_XL

    KV Cache: Q8_0 @ 200000

    Context window: 200000 (vs 163840 on the drafter-carrying base)

    VRAM: ~44.8 GB, ~3 GB free

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/gemma-4-31B-it/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -b 4096 -ub 1024

    -c 200000

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --top-k 64

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.09
    output: 0.34
    cached: 0.05
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/gemma-4-31B-it/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `-c` | `200000` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--top-k` | `64` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |
