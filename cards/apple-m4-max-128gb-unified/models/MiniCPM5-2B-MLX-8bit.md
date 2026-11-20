# MiniCPM5-2B-MLX-8bit
**Card:** Apple M4 Max (128 GB unified memory)  |  **Engine:** MLX (mlx-lm)

**Display name:** MiniCPM5-2B (MLX 8-bit, llama arch)

## Model card summary (from the deployed config)

- Type: Dense (llama arch, 42 layers, GQA 16/2) | engine A/B candidate vs minicpm5-2b (GGUF)
- Parameter: 2.5B
- Source: self-quant from https://huggingface.co/openbmb/MiniCPM5-2B
- Engine: Metal (MLX)
- Inference: ml-explore/mlx-lm (0.31.3, venv-mlx)
- Quant: MLX 8-bit group 64 (8.501 bpw, 2.5 GB)
- Context window: 131072 (native)
- Notes: thinking always on (template property, both engines); ttl 600, NOT resident.

## llama-swap entry

- **ttl:** `600`

## Config block

```yaml
MiniCPM5-2B-MLX-8bit:
  name: MiniCPM5-2B (MLX 8-bit, llama arch)
  description: 'Type: Dense (llama arch, 42 layers, GQA 16/2) | engine A/B candidate vs minicpm5-2b (GGUF)

    Parameter: 2.5B

    Source: self-quant from https://huggingface.co/openbmb/MiniCPM5-2B

    Engine: Metal (MLX)

    Inference: ml-explore/mlx-lm (0.31.3, venv-mlx)

    Quant: MLX 8-bit group 64 (8.501 bpw, 2.5 GB)

    Context window: 131072 (native)

    Notes: thinking always on (template property, both engines); ttl 600, NOT resident.

    '
  cmd: '${mlx-serve} MiniCPM5-2B-MLX-8bit --host 127.0.0.1 --port ${PORT} --temp 1.0 --top-p 0.95 --min-p
    0.0

    '
  ttl: 600
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `--temp` | `1.0` | Default sampling temperature (client per-request values override). |
| `--top-p` | `0.95` | Default top-p sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
