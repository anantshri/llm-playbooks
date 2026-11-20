# qwen-image-2.1
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** other

**Display name:** Qwen-Image-2.1 (bf16)

## Model card summary (from the deployed config)

- Type: Image generation (Qwen-Image-2.1 DiT, 7B) — unified T2I, native RGBA transparency
- Engine: CUDA (sm_120)
- Inference: sd-server
- Quant: bf16 safetensors diffusion + Qwen3-VL-8B-Instruct Q8_0 text encoder
- VRAM: ~24 GB
- Endpoints: /v1/images/generations, /sdapi/v1/txt2img
- Sampler: cfg-scale 6.0 + euler, per upstream docs/qwen_image_2.1.md.
- Transparency: prefix the prompt with "This is an RGBA image with
- transparency. <subject>. The image has alpha channel and the
- background is transparent." to get an alpha channel.
- Encoder: Qwen-Image-2.1 changed text encoders — 1.0 used Qwen2.5-VL-7B,
- 2.1 uses Qwen3-VL-8B. The VAE also changed and is NOT interchangeable
- with the Qwen-Image 1.0 VAE (same trap class as FLUX.1 vs FLUX.2).

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/`
- **pricing:** `{'input': 0, 'output': 0}`

## Config block

```yaml
qwen-image-2.1:
  name: Qwen-Image-2.1 (bf16)
  description: "Type: Image generation (Qwen-Image-2.1 DiT, 7B) — unified T2I, native RGBA transparency\n\
    Engine: CUDA (sm_120)\nInference: sd-server\nQuant: bf16 safetensors diffusion + Qwen3-VL-8B-Instruct\
    \ Q8_0 text encoder\nVRAM: ~24 GB\nEndpoints: /v1/images/generations, /sdapi/v1/txt2img\nSampler:\
    \ cfg-scale 6.0 + euler, per upstream docs/qwen_image_2.1.md.\nTransparency: prefix the prompt with\
    \ \"This is an RGBA image with\n  transparency. <subject>. The image has alpha channel and the\n \
    \ background is transparent.\" to get an alpha channel.\nEncoder: Qwen-Image-2.1 changed text encoders\
    \ — 1.0 used Qwen2.5-VL-7B,\n  2.1 uses Qwen3-VL-8B. The VAE also changed and is NOT interchangeable\n\
    \  with the Qwen-Image 1.0 VAE (same trap class as FLUX.1 vs FLUX.2).\n"
  cmd: '${sd-server-bin}

    --diffusion-model ${sd-diffusion}/Qwen-Image-2.1/current.safetensors

    --vae ${sd-vae}/qwen_image_2.1_vae.safetensors

    --llm ${sd-encoders}/Qwen3-VL-8B/current.gguf

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
