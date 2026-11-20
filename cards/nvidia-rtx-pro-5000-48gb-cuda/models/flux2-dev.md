# flux2-dev
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** other

**Display name:** FLUX.2-dev (Q4_K_S GGUF)

## Model card summary (from the deployed config)

- Type: Image generation (FLUX.2-dev diffusion) — highest-quality FLUX.2, uncensored base
- Engine: CUDA (sm_120)
- Inference: sd-server
- Quant: Q4_K_S GGUF diffusion + Mistral-Small-3.2-24B LLM text encoder
- VRAM: ~30 GB
- Endpoints: /v1/images/generations, /sdapi/v1/txt2img

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/`
- **pricing:** `{'input': 0, 'output': 0}`

## Config block

```yaml
flux2-dev:
  name: FLUX.2-dev (Q4_K_S GGUF)
  description: 'Type: Image generation (FLUX.2-dev diffusion) — highest-quality FLUX.2, uncensored base

    Engine: CUDA (sm_120)

    Inference: sd-server

    Quant: Q4_K_S GGUF diffusion + Mistral-Small-3.2-24B LLM text encoder

    VRAM: ~30 GB

    Endpoints: /v1/images/generations, /sdapi/v1/txt2img

    '
  cmd: '${sd-server-bin}

    --diffusion-model ${sd-diffusion}/FLUX.2-dev/current.gguf

    --vae ${sd-vae}/flux2_ae.safetensors

    --llm ${sd-encoders}/Mistral-Small-3.2-24B/current.gguf

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
| `--diffusion-model` | `${sd-diffusion}/FLUX.2-dev/current.gguf` | Diffusion weights (FLUX / Qwen-Image backbone). |
| `--vae` | `${sd-vae}/flux2_ae.safetensors` | VAE decoder weights. |
| `--llm` | `${sd-encoders}/Mistral-Small-3.2-24B/current.gguf` | LLM text encoder used by this image family (Qwen-Image uses a Qwen3 VL GGUF). |
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
