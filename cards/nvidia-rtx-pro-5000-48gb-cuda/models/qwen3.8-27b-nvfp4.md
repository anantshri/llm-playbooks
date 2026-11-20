# qwen3.8-27b-nvfp4
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** vLLM

**Display name:** Qwen3.8 27B (ModelOpt NVFP4, vLLM, vision, 262k) — quant A/B

## Model card summary (from the deployed config)

- Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | Vision | NO MTP
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/nvidia/Qwen3.8-27B-NVFP4
- Engine: CUDA (sm_120)
- Inference: vllm-project/vllm (0.28.0)
- Quant: ModelOpt NVFP4 (MLP+lm_head 4-bit, FP8 attention; 21.9 GB)
- KV Cache: FP16 @ 262144
- Context window: 262144
- Chat template: Qwen (reasoning parsed)

## llama-swap entry

- **ttl:** `1800`
- **checkEndpoint:** `/health`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
qwen3.8-27b-nvfp4:
  name: Qwen3.8 27B (ModelOpt NVFP4, vLLM, vision, 262k) — quant A/B
  description: 'Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | Vision | NO MTP

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/nvidia/Qwen3.8-27B-NVFP4

    Engine: CUDA (sm_120)

    Inference: vllm-project/vllm (0.28.0)

    Quant: ModelOpt NVFP4 (MLP+lm_head 4-bit, FP8 attention; 21.9 GB)

    KV Cache: FP16 @ 262144

    Context window: 262144

    Chat template: Qwen (reasoning parsed)

    '
  cmd: '${vllm-bin} serve ${hf-models}/Qwen3.8-27B-NVFP4

    --served-model-name qwen3.8-27b-nvfp4

    --host 127.0.0.1 --port ${PORT}

    --dtype auto

    --max-model-len 262144

    --gpu-memory-utilization 0.92

    --kv-cache-dtype auto

    --no-enable-prefix-caching

    --enable-auto-tool-choice

    --tool-call-parser qwen3_xml

    --reasoning-parser qwen3

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
| `--served-model-name` | `qwen3.8-27b-nvfp4` | vLLM: model name exposed on the API - matches the entry id. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `--dtype` | `auto` | vLLM: activation/weights dtype at runtime. |
| `--max-model-len` | `262144` | vLLM: max context length served. |
| `--gpu-memory-utilization` | `0.92` | vLLM: fraction of VRAM the engine may claim (pre-allocates KV pool). |
| `--kv-cache-dtype` | `auto` | vLLM: KV dtype (auto = FP16; fp8 measured slower-or-equal on both axes on XPU). |
| `--no-enable-prefix-caching` | _(flag)_ | vLLM: disable prefix caching (measured interacting with MTP on this build). |
| `--enable-auto-tool-choice` | _(flag)_ | vLLM: allow tool-call parsing into structured tool_calls. |
| `--tool-call-parser` | `qwen3_xml` | vLLM: which tool-call format to parse (native qwen3_coder here - dodges the llama.cpp grammar limit). |
| `--reasoning-parser` | `qwen3` | vLLM: reasoning block parser. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |
