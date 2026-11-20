# qwen3.6-27b-131k
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.6 27B — Q8_0, KV F16, 131k, MTP — co-residency fidelity arm

## Model card summary (from the deployed config)

- Type: Dense hybrid (qwen35 trunk) | MTP speculative
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: Q8_0
- KV Cache: F16 @ 131072 (~6.4 GiB)
- Context window: 131072
- VRAM: 37774 MiB of 48935 measured at load (2026-11-10); pairing with the
- worker arm totals 47437 MiB (~1.5 GiB free)
- Chat template: Qwen (--jinja; reasoning parsed)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.3, 'output': 2.0, 'cached': 0.03}`

## Config block

```yaml
qwen3.6-27b-131k:
  name: Qwen3.6 27B — Q8_0, KV F16, 131k, MTP — co-residency fidelity arm
  description: "Type: Dense hybrid (qwen35 trunk) | MTP speculative\nParameter: 27B total / 27B active\
    \ (dense)\nSource: https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF\nEngine: CUDA (sm_120)\nInference:\
    \ anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)\nQuant: Q8_0\nKV Cache: F16 @ 131072 (~6.4\
    \ GiB)\nContext window: 131072\nVRAM: 37774 MiB of 48935 measured at load (2026-11-10); pairing with\
    \ the\n  worker arm totals 47437 MiB (~1.5 GiB free)\nChat template: Qwen (--jinja; reasoning parsed)\n"
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Qwen3.6-27B-MTP/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --spec-type draft-mtp

    --spec-draft-n-max 2

    --reasoning-format deepseek

    --chat-template-kwargs ''{"preserve_thinking":true}''

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
| `-m` | `${gguf-models}/Qwen3.6-27B-MTP/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `draft-mtp` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `2` | Max tokens speculated per step. |
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
