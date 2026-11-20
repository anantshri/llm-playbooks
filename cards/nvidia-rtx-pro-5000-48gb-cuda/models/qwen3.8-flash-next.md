# qwen3.8-flash-next
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.8 Flash Next (125B-A6B sparse MoE, UD-Q4_K_XL, 262K)

## Model card summary (from the deployed config)

- Type: Sparse MoE (Gated DeltaNet 3:1 + QSA sparse attention budget 2048; 512 experts top-10 + shared + 51B n-gram PLE embedding table)
- Parameter: 125B total / 6B active
- Source: https://huggingface.co/unsloth/Qwen3.8-Flash-Next-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: UD-Q4_K_XL (111 GB)
- KV Cache: Q8_0 @ 262144
- Context window: 262144
- Chat template: Qwen (--jinja)
- VRAM: peak 46.1 of 48 (nothing co-resides)

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/health`
- **pricing:** `{'input': 0.15, 'output': 0.47, 'cached': 0.016}`

## Config block

```yaml
qwen3.8-flash-next:
  name: Qwen3.8 Flash Next (125B-A6B sparse MoE, UD-Q4_K_XL, 262K)
  description: 'Type: Sparse MoE (Gated DeltaNet 3:1 + QSA sparse attention budget 2048; 512 experts top-10
    + shared + 51B n-gram PLE embedding table)

    Parameter: 125B total / 6B active

    Source: https://huggingface.co/unsloth/Qwen3.8-Flash-Next-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: UD-Q4_K_XL (111 GB)

    KV Cache: Q8_0 @ 262144

    Context window: 262144

    Chat template: Qwen (--jinja)

    VRAM: peak 46.1 of 48 (nothing co-resides)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Qwen3.8-Flash-Next/UD-Q4_K_XL/Qwen3.8-Flash-Next-UD-Q4_K_XL-00001-of-00004.gguf

    --alias qwen3.8-flash-next

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    --n-cpu-moe 26

    --cache-type-k q8_0 --cache-type-v q8_0

    -c 262144

    --jinja

    --top-k 20

    --min-p 0.0

    --reasoning-format deepseek

    '
  checkEndpoint: /health
  ttl: 600
  pricing:
    input: 0.15
    output: 0.47
    cached: 0.016
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Qwen3.8-Flash-Next/UD-Q4_K_XL/Qwen3.8-Flash-Next-UD-Q4_K_XL-00001-of-00004.gguf` | Weights file (GGUF) this entry serves. |
| `--alias` | `qwen3.8-flash-next` | Model name the backend reports - kept identical to the entry id clients request. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `--n-cpu-moe` | `26` | Keep N MoE expert layers on CPU RAM - the VRAM-fit lever for oversized MoE models; costs host RAM and prefill speed. |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--top-k` | `20` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Notes, incidents, upstream references

- --n-cpu-moe 26 + q8_0 KV host-RAM offload, ~73 GB host RAM. NIAH 28/28 to 196k (2026-08-28).
- Text-only since 2026-10-03 (mmproj dropped to buy context). MTP variants are NOT lossless - silent depth-specific failures; never for exact-format output.
