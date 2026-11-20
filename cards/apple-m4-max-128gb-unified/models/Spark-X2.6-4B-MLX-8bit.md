# Spark-X2.6-4B-MLX-8bit
**Card:** Apple M4 Max (128 GB unified memory)  |  **Engine:** MLX (mlx-lm)

**Display name:** Spark X2.6 4B (MLX 8-bit, native spark2_5)

## Model card summary (from the deployed config)

- Type: PLE dense | on-demand (MLX engine build of spark-2.6-4b)
- Parameter: 4.1B
- Source: self-quant from https://huggingface.co/XHToken/Spark-X2.6-4B (revision 2f1d8110eb60b18920d424d7bf47f4001a956f1e)
- Engine: Metal (MLX, RotatingKVCache SWA + SDPA full attention)
- Inference: ml-explore/mlx-lm (0.32.0, venv-mlx-spark)
- Quant: MLX 8-bit group 64 (g_proj kept high-precision by quant predicate)
- Context window: 1048576 (train); KV fp16, unbounded (no --max-kv-size in 0.32.0)
- Notes: thinking off via wrapper; on-demand ttl 600 (2.5 MLX remains the resident).

## llama-swap entry

- **ttl:** `600`

## Config block

```yaml
Spark-X2.6-4B-MLX-8bit:
  name: Spark X2.6 4B (MLX 8-bit, native spark2_5)
  description: 'Type: PLE dense | on-demand (MLX engine build of spark-2.6-4b)

    Parameter: 4.1B

    Source: self-quant from https://huggingface.co/XHToken/Spark-X2.6-4B (revision 2f1d8110eb60b18920d424d7bf47f4001a956f1e)

    Engine: Metal (MLX, RotatingKVCache SWA + SDPA full attention)

    Inference: ml-explore/mlx-lm (0.32.0, venv-mlx-spark)

    Quant: MLX 8-bit group 64 (g_proj kept high-precision by quant predicate)

    Context window: 1048576 (train); KV fp16, unbounded (no --max-kv-size in 0.32.0)

    Notes: thinking off via wrapper; on-demand ttl 600 (2.5 MLX remains the resident).

    '
  cmd: '${mlx-serve} Spark-X2.6-4B-MLX-8bit --host 127.0.0.1 --port ${PORT} --temp 1.0 --top-p 0.95 --top-k
    0 --min-p 0.0

    '
  env:
  - MLX_PYTHON=~/inference/venv-mlx-spark/bin/python
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
| `--top-k` | `0` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## env parameters

| Variable | Value here | What it does |
|---|---|---|
| `MLX_PYTHON` | `~/inference/venv-mlx-spark/bin/python` | Override the mlx-lm venv python for this entry (a model needing a newer mlx-lm than the fleet-pinned 0.31.3). |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
