# qwen3.8-27b-vllm
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** vLLM

**Display name:** Qwen3.8 27B (GPTQ INT4 + BF16 MTP4, vLLM/XPU) — quality-verified 09-04

## Model card summary (from the deployed config)

- Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | MTP4 | NO vision
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/Qwen/Qwen3.8-27B
- Engine: XPU/SYCL (torch 2.13.0+xpu)
- Inference: vllm-project/vllm (0.28.0)
- Quant: self-quant GPTQ INT4 g128, MTP tensors held BF16 (19 GiB)
- KV Cache: FP16 @ 102400 (7.99 GiB)
- Context window: 102400 (native 262144)
- Speculative: native MTP (n=4)
- Measured: 52.4 tok/s @8.1k / 50.1 @97.9k (2.07-2.84x vs llama.cpp sibling)
- Quality battery 2026-09-04: needles 9/9 @15k-99k all depths; canaries 14/15;
- spec parity 10/10 byte-identical; concurrency 2/4/8 + 4x8k clean.
- Detail: logs/the Arc B70 box/2026-09-04-quality-battery.md

## llama-swap entry

- **ttl:** `1800`
- **checkEndpoint:** `/health`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
qwen3.8-27b-vllm:
  name: Qwen3.8 27B (GPTQ INT4 + BF16 MTP4, vLLM/XPU) — quality-verified 09-04
  description: 'Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | MTP4 | NO vision

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/Qwen/Qwen3.8-27B

    Engine: XPU/SYCL (torch 2.13.0+xpu)

    Inference: vllm-project/vllm (0.28.0)

    Quant: self-quant GPTQ INT4 g128, MTP tensors held BF16 (19 GiB)

    KV Cache: FP16 @ 102400 (7.99 GiB)

    Context window: 102400 (native 262144)

    Speculative: native MTP (n=4)

    Measured: 52.4 tok/s @8.1k / 50.1 @97.9k (2.07-2.84x vs llama.cpp sibling)

    Quality battery 2026-09-04: needles 9/9 @15k-99k all depths; canaries 14/15;

    spec parity 10/10 byte-identical; concurrency 2/4/8 + 4x8k clean.

    Detail: logs/the Arc B70 box/2026-09-04-quality-battery.md

    '
  cmd: '${vllm-bin} serve ${hf-models}/Qwen3.8-27B-GPTQ-Int4-MTP

    --served-model-name qwen3.8-27b-vllm

    --host 127.0.0.1 --port ${PORT}

    --quantization gptq

    --dtype float16

    --max-model-len 102400

    --gpu-memory-utilization 0.95

    --kv-cache-dtype auto

    --max-num-seqs 8

    --no-enable-prefix-caching

    --language-model-only

    --speculative-config ''{"method":"mtp","num_speculative_tokens":4}''

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
| `--served-model-name` | `qwen3.8-27b-vllm` | vLLM: model name exposed on the API - matches the entry id. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `--quantization` | `gptq` | vLLM: weight quantization scheme (GPTQ INT4 g128 = the proven XPU recipe). |
| `--dtype` | `float16` | vLLM: activation/weights dtype at runtime. |
| `--max-model-len` | `102400` | vLLM: max context length served. |
| `--gpu-memory-utilization` | `0.95` | vLLM: fraction of VRAM the engine may claim (pre-allocates KV pool). |
| `--kv-cache-dtype` | `auto` | vLLM: KV dtype (auto = FP16; fp8 measured slower-or-equal on both axes on XPU). |
| `--max-num-seqs` | `8` | vLLM: max concurrent sequences. |
| `--no-enable-prefix-caching` | _(flag)_ | vLLM: disable prefix caching (measured interacting with MTP on this build). |
| `--language-model-only` | _(flag)_ | Skip loading the vision projector - text-only (the mmproj VRAM was traded for context). |
| `--speculative-config` | `'{"method":"mtp","num_speculative_tokens":4}'` | vLLM: speculative decoding configuration (MTP). |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Notes, incidents, upstream references

- vLLM XPU entry. On this card vLLM measured ~2x llama.cpp on the same harness (opposite verdict vs the CUDA box). --kv-cache-dtype auto (FP16); fp8 KV measured slower-or-equal on both axes.
- Escape hatch for the llama.cpp grammar limit (huge client tool schemas crash every llama.cpp entry).
