# tf-qwen3.8-27b-nvfp4
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** other

**Display name:** Qwen3.8 27B (ModelOpt NVFP4, TensorFold) — engine A/B

## Model card summary (from the deployed config)

- Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | Vision weights present, not wired
- Parameter: 27B
- Source: https://huggingface.co/nvidia/Qwen3.8-27B-NVFP4
- Engine: CUDA (sm_120)
- Inference: ashhart/TensorFold (0.3.6.3, fork anantshri/TensorFold branch the RTX PRO 5000 box)
- Quant: ModelOpt NVFP4 (MLP+lm_head 4-bit, FP8 attention; 21.1 GB resident)
- KV Cache: bf16
- Context window: 65536 (native 262144 — 64k is this engine's hard ceiling here; see NOTES)
- Reasoning: auto-parsed into reasoning_content (no flag needed)
- Notes: Engine A/B against qwen3.8-27b-nvfp4 (vLLM) on the identical checkpoint.

## llama-swap entry

- **ttl:** `1800`
- **checkEndpoint:** `/health`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
tf-qwen3.8-27b-nvfp4:
  name: Qwen3.8 27B (ModelOpt NVFP4, TensorFold) — engine A/B
  description: 'Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | Vision weights present,
    not wired

    Parameter: 27B

    Source: https://huggingface.co/nvidia/Qwen3.8-27B-NVFP4

    Engine: CUDA (sm_120)

    Inference: ashhart/TensorFold (0.3.6.3, fork anantshri/TensorFold branch the RTX PRO 5000 box)

    Quant: ModelOpt NVFP4 (MLP+lm_head 4-bit, FP8 attention; 21.1 GB resident)

    KV Cache: bf16

    Context window: 65536 (native 262144 — 64k is this engine''s hard ceiling here; see NOTES)

    Reasoning: auto-parsed into reasoning_content (no flag needed)

    Notes: Engine A/B against qwen3.8-27b-nvfp4 (vLLM) on the identical checkpoint.

    '
  cmd: '${tensorfold-serve} ${hf-models}/Qwen3.8-27B-NVFP4

    --backend cuda

    --name tf-qwen3.8-27b-nvfp4

    --host 127.0.0.1 --port ${PORT}

    --context 65536

    --no-drafts

    --no-update-check

    '
  checkEndpoint: /health
  ttl: 1800
  pricing:
    input: 0.42
    output: 3.0
    cached: 0.085
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `--backend` | `cuda` | Engine/build-specific flag - see this engine build's --help. |
| `--name` | `tf-qwen3.8-27b-nvfp4` | Model name the image server reports. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `--context` | `65536` | Engine/build-specific flag - see this engine build's --help. |
| `--no-drafts` | _(flag)_ | Engine/build-specific flag - see this engine build's --help. |
| `--no-update-check` | _(flag)_ | Engine/build-specific flag - see this engine build's --help. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |
