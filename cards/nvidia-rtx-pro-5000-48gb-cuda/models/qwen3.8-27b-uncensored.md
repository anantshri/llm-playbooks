# qwen3.8-27b-uncensored
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.8 27B Uncensored (self-quant Q8_0, vision, 262k)

## Model card summary (from the deployed config)

- Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | abliterated | Vision
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/orcarouter/Qwen3.8-27B-Uncensored
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: self-quant Q8_0 (~30 GB) + mmproj-F16
- KV Cache: Q8_0 @ 262144
- Context window: 262144
- Chat template: Qwen (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
qwen3.8-27b-uncensored:
  name: Qwen3.8 27B Uncensored (self-quant Q8_0, vision, 262k)
  description: 'Type: Dense hybrid (qwen35, 48 GDN linear + 16 full-attn) | abliterated | Vision

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/orcarouter/Qwen3.8-27B-Uncensored

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: self-quant Q8_0 (~30 GB) + mmproj-F16

    KV Cache: Q8_0 @ 262144

    Context window: 262144

    Chat template: Qwen (--jinja; reasoning parsed)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Qwen3.8-27B-Uncensored/current.gguf

    --mmproj ${gguf-models}/Qwen3.8-27B-Uncensored/mmproj-F16.gguf

    --image-min-tokens 1024

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 262144

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --spec-type ngram-mod

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
| `-m` | `${gguf-models}/Qwen3.8-27B-Uncensored/current.gguf` | Weights file (GGUF) this entry serves. |
| `--mmproj` | `${gguf-models}/Qwen3.8-27B-Uncensored/mmproj-F16.gguf` | Vision projector file - enables image input for this entry. |
| `--image-min-tokens` | `1024` | Minimum tokens budgeted per image (vision quality floor). |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
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

## Measured (llama-bench)

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). tg@9k is shallow decode; tg@max is the deep-decode walk (~98% of -c). Do not mix with gateway measurements (see troubleshooting).

| Metric | Value |
|---|---|
| pp @9k | 2293 tok/s |
| pp @max (254k ctx) | 797 tok/s |
| tg @9k (shallow) | 34 tok/s |
| tg @max (254k ctx) | 17.9 tok/s |

## Notes, incidents, upstream references

- Q8_0 here. Source repo has no insights-DB scan record - deployed under a documented override; flagged, not cleared.
