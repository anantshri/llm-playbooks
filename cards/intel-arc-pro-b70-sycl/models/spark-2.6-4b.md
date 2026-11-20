# spark-2.6-4b
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Spark X2.6 4B (BF16, SYCL, F16 KV, 131k)

## Model card summary (from the deployed config)

- Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on
- Parameter: 4B
- Source: https://huggingface.co/XHToken/Spark-X2.6-4B-GGUF (revision 16c77db5a782d75d6953770339f7ae06be7dd3f7)
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Env: GGML_SYCL_FA_DPAS_F16=1 (dpas F16 arm, via ${llama-server-spark26-bin}; ~+5% class per pentester sweep)
- Quant: BF16 (8.23 GB)
- KV Cache: F16 @ 131072
- Context window: 131072 (native 1048576)
- Chat template: embedded (--jinja)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.05, 'output': 0.1}`

## Config block

```yaml
spark-2.6-4b:
  name: Spark X2.6 4B (BF16, SYCL, F16 KV, 131k)
  description: 'Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on

    Parameter: 4B

    Source: https://huggingface.co/XHToken/Spark-X2.6-4B-GGUF (revision 16c77db5a782d75d6953770339f7ae06be7dd3f7)

    Engine: SYCL (Level Zero)

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Env: GGML_SYCL_FA_DPAS_F16=1 (dpas F16 arm, via ${llama-server-spark26-bin}; ~+5% class per pentester
    sweep)

    Quant: BF16 (8.23 GB)

    KV Cache: F16 @ 131072

    Context window: 131072 (native 1048576)

    Chat template: embedded (--jinja)

    '
  cmd: '${llama-server-spark26-bin}

    -m ${gguf-models}/Spark-X2.6-4B/Spark-X2.6-4B.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -ub 2048

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --top-k 0

    --min-p 0.0

    --reasoning-format deepseek

    '
  ttl: 600
  pricing:
    input: 0.05
    output: 0.1
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Spark-X2.6-4B/Spark-X2.6-4B.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-ub` | `2048` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--top-k` | `0` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Notes, incidents, upstream references

- `ggml-org/llama.cpp#30606` (open) - Routes this arch's D=256 single-query decode to the FA vector kernel on Battlemage.
