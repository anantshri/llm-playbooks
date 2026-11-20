# ornith-1.0-35b
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Ornith 1.0 35B (Q8_0 GGUF, 200k ctx)

## Model card summary (from the deployed config)

- Type: MoE (qwen3_5_moe, 40 layers, 256 experts x 8 active, 2 KV heads — heavy GQA)
- Parameter: 35B total
- Source: https://huggingface.co/deepreinforce-ai/Ornith-1.0-35B-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: Q8_0
- KV Cache: F16 @ 200000
- Context window: 200000
- VRAM: ~38.6 GB, ~9 GB free
- Chat template: Qwen3 XML (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.3, 'output': 2.0, 'cached': 0.03}`

## Config block

```yaml
ornith-1.0-35b:
  name: Ornith 1.0 35B (Q8_0 GGUF, 200k ctx)
  description: 'Type: MoE (qwen3_5_moe, 40 layers, 256 experts x 8 active, 2 KV heads — heavy GQA)

    Parameter: 35B total

    Source: https://huggingface.co/deepreinforce-ai/Ornith-1.0-35B-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: Q8_0

    KV Cache: F16 @ 200000

    Context window: 200000

    VRAM: ~38.6 GB, ~9 GB free

    Chat template: Qwen3 XML (--jinja; reasoning parsed)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/ornith-1.0-35b/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    --spec-type ngram-mod

    -c 200000

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --reasoning-format deepseek

    --top-k 20

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.3
    output: 2.0
    cached: 0.03
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/ornith-1.0-35b/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `--spec-type` | `ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `-c` | `200000` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--top-k` | `20` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Measured (llama-bench)

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). tg@9k is shallow decode; tg@max is the deep-decode walk (~98% of -c). Do not mix with gateway measurements (see troubleshooting).

| Metric | Value |
|---|---|
| pp @9k | 6578 tok/s |
| pp @max (193k ctx) | 3847 tok/s |
| tg @9k (shallow) | 142 tok/s |
| tg @max (193k ctx) | 76 tok/s |
