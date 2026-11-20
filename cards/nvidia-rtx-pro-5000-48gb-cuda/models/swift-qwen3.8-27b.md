# swift-qwen3.8-27b
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Swift Qwen3.8 27B (Q8_0 GGUF, MTP, vision, 262k)

## Model card summary (from the deployed config)

- Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | Vision | MTP
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/ukisai/Swift-Qwen3.8-27B-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: Q8_0 (27.1 GiB) + mmproj-F16 (0.9 GiB)
- KV Cache: Q8_0 @ 262144
- Context window: 262144 (1M via YaRN)
- Chat template: Qwen (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
swift-qwen3.8-27b:
  name: Swift Qwen3.8 27B (Q8_0 GGUF, MTP, vision, 262k)
  description: 'Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | Vision | MTP

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/ukisai/Swift-Qwen3.8-27B-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: Q8_0 (27.1 GiB) + mmproj-F16 (0.9 GiB)

    KV Cache: Q8_0 @ 262144

    Context window: 262144 (1M via YaRN)

    Chat template: Qwen (--jinja; reasoning parsed)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Swift-Qwen3.8-27B/Swift-Qwen3.8-27B-Q8_0.gguf

    --mmproj ${gguf-models}/Swift-Qwen3.8-27B/mmproj-Swift-Qwen3.8-27B-F16.gguf

    --image-min-tokens 1024

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 262144

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --spec-type draft-mtp,ngram-mod

    --spec-draft-n-max 3

    --reasoning-format deepseek

    --chat-template-kwargs ''{"preserve_thinking":true}''

    --top-k 20

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.42
    output: 3.0
    cached: 0.085
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Swift-Qwen3.8-27B/Swift-Qwen3.8-27B-Q8_0.gguf` | Weights file (GGUF) this entry serves. |
| `--mmproj` | `${gguf-models}/Swift-Qwen3.8-27B/mmproj-Swift-Qwen3.8-27B-F16.gguf` | Vision projector file - enables image input for this entry. |
| `--image-min-tokens` | `1024` | Minimum tokens budgeted per image (vision quality floor). |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `draft-mtp,ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `3` | Max tokens speculated per step. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--chat-template-kwargs` | `'{"preserve_thinking":true}'` | JSON kwargs passed to the chat template (deprecated flag form). Used here to force thinking off server-side. |
| `--top-k` | `20` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |
