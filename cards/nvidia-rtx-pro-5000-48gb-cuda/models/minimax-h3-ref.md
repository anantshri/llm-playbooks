# minimax-h3-ref
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** other

**Display name:** MiniMax-H3 (video+audio, ref2va omni-references, int8 + group offload)

## Model card summary (from the deployed config)

- Type: Joint video+audio diffusion (MiniMaxH3ModularPipeline) | ref2va omni-reference partition
- Parameter: 33B dense single-stream transformer (transformer_ref partition)
- Source: https://huggingface.co/MiniMaxAI/MiniMax-H3
- Engine: CUDA (sm_120)
- Inference: diffusers 0.40.0 + torchao int8 (venv-h3)
- Quant: int8 weight-only (transformer_ref + Qwen3-VL conditioner)
- Context window: n/a (24 fps, 5-15 s; num_frames required)
- Chat template: n/a — POST /v1/video/generations
- VRAM: ~11 GiB idle / ~21 GiB peak; ~87 GB host RAM

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/health`
- **pricing:** `{'input': 0.0, 'output': 0.0}`

## Config block

```yaml
minimax-h3-ref:
  name: MiniMax-H3 (video+audio, ref2va omni-references, int8 + group offload)
  description: 'Type: Joint video+audio diffusion (MiniMaxH3ModularPipeline) | ref2va omni-reference partition

    Parameter: 33B dense single-stream transformer (transformer_ref partition)

    Source: https://huggingface.co/MiniMaxAI/MiniMax-H3

    Engine: CUDA (sm_120)

    Inference: diffusers 0.40.0 + torchao int8 (venv-h3)

    Quant: int8 weight-only (transformer_ref + Qwen3-VL conditioner)

    Context window: n/a (24 fps, 5-15 s; num_frames required)

    Chat template: n/a — POST /v1/video/generations

    VRAM: ~11 GiB idle / ~21 GiB peak; ~87 GB host RAM

    '
  cmd: '${h3-server-bin}

    --model ${hf-models}/MiniMax-H3

    --workflow ref2va

    --host 127.0.0.1 --port ${PORT}

    --device cuda

    --served-name minimax-h3-ref

    --output-dir /opt/inference/work/h3-out

    '
  checkEndpoint: /health
  ttl: 600
  pricing:
    input: 0.0
    output: 0.0
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `--model` | `${hf-models}/MiniMax-H3` | Weights/checkpoint this entry serves. |
| `--workflow` | `ref2va` | Image workflow definition (comfy-style graph). |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `--device` | `cuda` | GPU selection for the image server. |
| `--served-name` | `minimax-h3-ref` | Model name the backend reports - kept identical to the entry id. |
| `--output-dir` | `/opt/inference/work/h3-out` | Where generated images are written. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |
