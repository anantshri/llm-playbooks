# qwen-image-2.1-edit
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** other

**Display name:** Qwen-Image-2.1 Edit (bf16)

## Model card summary (from the deployed config)

- Type: Image editing (Qwen-Image-2.1 DiT, 7B) — reference-image editing
- Engine: CUDA (sm_120)
- Inference: sd-server
- Quant: bf16 safetensors diffusion + Qwen3-VL-8B-Instruct Q8_0 + F16 mmproj
- VRAM: ~25 GB
- Endpoints: /v1/images/edits, /sdapi/v1/img2img
- Capability: up to 10 reference images, local edits via circles/painted
- annotations/masks, identity preservation for people and products,
- transparent-layer editing and subject extraction.
- Difference from qwen-image-2.1: adds --llm_vision (the Qwen3-VL vision
- tower). Same weights otherwise — the mmproj is what makes reference
- images legible to the text encoder, so t2i does not need it.

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/`
- **pricing:** `{'input': 0, 'output': 0}`

## Config block

```yaml
qwen-image-2.1-edit:
  name: Qwen-Image-2.1 Edit (bf16)
  description: "Type: Image editing (Qwen-Image-2.1 DiT, 7B) — reference-image editing\nEngine: CUDA (sm_120)\n\
    Inference: sd-server\nQuant: bf16 safetensors diffusion + Qwen3-VL-8B-Instruct Q8_0 + F16 mmproj\n\
    VRAM: ~25 GB\nEndpoints: /v1/images/edits, /sdapi/v1/img2img\nCapability: up to 10 reference images,\
    \ local edits via circles/painted\n  annotations/masks, identity preservation for people and products,\n\
    \  transparent-layer editing and subject extraction.\nDifference from qwen-image-2.1: adds --llm_vision\
    \ (the Qwen3-VL vision\n  tower). Same weights otherwise — the mmproj is what makes reference\n  images\
    \ legible to the text encoder, so t2i does not need it.\n"
  cmd: '${sd-server-bin}

    --diffusion-model ${sd-diffusion}/Qwen-Image-2.1/current.safetensors

    --vae ${sd-vae}/qwen_image_2.1_vae.safetensors

    --llm ${sd-encoders}/Qwen3-VL-8B/current.gguf

    --llm_vision ${sd-encoders}/Qwen3-VL-8B/mmproj-current.gguf

    --cfg-scale 6.0

    --sampling-method euler

    --listen-ip 127.0.0.1 --listen-port ${PORT}

    '
  checkEndpoint: /
  ttl: 600
  pricing:
    input: 0
    output: 0
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `--diffusion-model` | `${sd-diffusion}/Qwen-Image-2.1/current.safetensors` | Diffusion weights (FLUX / Qwen-Image backbone). |
| `--vae` | `${sd-vae}/qwen_image_2.1_vae.safetensors` | VAE decoder weights. |
| `--llm` | `${sd-encoders}/Qwen3-VL-8B/current.gguf` | LLM text encoder used by this image family (Qwen-Image uses a Qwen3 VL GGUF). |
| `--llm_vision` | `${sd-encoders}/Qwen3-VL-8B/mmproj-current.gguf` | Vision projector for the LLM text encoder. |
| `--cfg-scale` | `6.0` | Classifier-free guidance scale. |
| `--sampling-method` | `euler` | Diffusion sampler. |
| `--listen-ip` | `127.0.0.1` | Bind address (image server). |
| `--listen-port` | `${PORT}` | Bind port (image server). |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |
