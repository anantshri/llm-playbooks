# flux1-dev-nsfw
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** other

**Display name:** FLUX.1-dev NSFW (Q4_K_S GGUF)

## Model card summary (from the deployed config)

- Type: Image generation (FLUX.1-dev) — mature NSFW LoRA ecosystem
- Engine: CUDA (sm_120)
- Inference: sd-server
- Quant: Q4_K_S GGUF diffusion + clip_l + t5xxl encoders
- VRAM: ~20 GB
- Endpoints: /v1/images/generations, /sdapi/v1/txt2img

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/`
- **pricing:** `{'input': 0, 'output': 0}`

## Config block

```yaml
flux1-dev-nsfw:
  name: FLUX.1-dev NSFW (Q4_K_S GGUF)
  description: 'Type: Image generation (FLUX.1-dev) — mature NSFW LoRA ecosystem

    Engine: CUDA (sm_120)

    Inference: sd-server

    Quant: Q4_K_S GGUF diffusion + clip_l + t5xxl encoders

    VRAM: ~20 GB

    Endpoints: /v1/images/generations, /sdapi/v1/txt2img

    '
  cmd: '${sd-server-bin}

    --diffusion-model ${sd-diffusion}/FLUX.1-dev/current.gguf

    --vae ${sd-vae}/flux1_ae.safetensors

    --clip_l ${sd-encoders}/FLUX.1/clip_l.safetensors

    --t5xxl ${sd-encoders}/FLUX.1/t5xxl-current.gguf

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
| `--diffusion-model` | `${sd-diffusion}/FLUX.1-dev/current.gguf` | Diffusion weights (FLUX / Qwen-Image backbone). |
| `--vae` | `${sd-vae}/flux1_ae.safetensors` | VAE decoder weights. |
| `--clip_l` | `${sd-encoders}/FLUX.1/clip_l.safetensors` | CLIP-L text encoder weights. |
| `--t5xxl` | `${sd-encoders}/FLUX.1/t5xxl-current.gguf` | T5-XXL text encoder weights. |
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
