# orcacyber-nano-1.0
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** OrcaCyber Nano 1.0 (Mix-15.7G, SYCL, F16 KV, MTP+ngram, vision, 131k)

## Model card summary (from the deployed config)

- Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | Vision | MTP | cybersecurity defense fine-tune
- Parameter: 27B total / 27B active (dense)
- Source: https://huggingface.co/orcarouter/OrcaCyber-Nano-1.0
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Quant: Mix-15.7G (14.6 GiB) + mmproj-F16 (0.86 GiB)
- KV Cache: F16 @ 131072
- Context window: 131072 (native 262144)
- Chat template: Qwen (--jinja; reasoning parsed; thinking aliases :nothink · :medium)

## llama-swap entry

- **ttl:** `600`
- **filters:** `{'setParamsByID': {'${MODEL_ID}:nothink': {'chat_template_kwargs.enable_thinking': False}, '${MODEL_ID}:medium': {'chat_template_kwargs.reasoning_effort': 'medium'}}}`
- **pricing:** `{'input': 0.42, 'output': 3.0, 'cached': 0.085}`

## Config block

```yaml
orcacyber-nano-1.0:
  name: OrcaCyber Nano 1.0 (Mix-15.7G, SYCL, F16 KV, MTP+ngram, vision, 131k)
  description: 'Type: Dense hybrid (48 Gated-DeltaNet linear + 16 full-attn of 64) | Vision | MTP | cybersecurity
    defense fine-tune

    Parameter: 27B total / 27B active (dense)

    Source: https://huggingface.co/orcarouter/OrcaCyber-Nano-1.0

    Engine: SYCL (Level Zero)

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Quant: Mix-15.7G (14.6 GiB) + mmproj-F16 (0.86 GiB)

    KV Cache: F16 @ 131072

    Context window: 131072 (native 262144)

    Chat template: Qwen (--jinja; reasoning parsed; thinking aliases :nothink · :medium)

    '
  cmd: '${llama-server-sycl-bin}

    -m ${gguf-models}/OrcaCyber-Nano-1.0/OrcaCyber-Nano-1.0-AI-Defense-Mix-15.7G.gguf

    --mmproj ${gguf-models}/OrcaCyber-Nano-1.0/mmproj-OrcaCyber-Nano-1.0-AI-Defense-F16.gguf

    --image-min-tokens 1024

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -b 4096 -ub 1024

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --spec-type draft-mtp,ngram-mod

    --spec-draft-n-max 3

    --spec-draft-p-min 0.3

    --reasoning-format deepseek

    --chat-template-kwargs ''{"preserve_thinking":true}''

    --top-k 20

    --min-p 0.0

    '
  filters:
    setParamsByID:
      ${MODEL_ID}:nothink:
        chat_template_kwargs.enable_thinking: false
      ${MODEL_ID}:medium:
        chat_template_kwargs.reasoning_effort: medium
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
| `-m` | `${gguf-models}/OrcaCyber-Nano-1.0/OrcaCyber-Nano-1.0-AI-Defense-Mix-15.7G.gguf` | Weights file (GGUF) this entry serves. |
| `--mmproj` | `${gguf-models}/OrcaCyber-Nano-1.0/mmproj-OrcaCyber-Nano-1.0-AI-Defense-F16.gguf` | Vision projector file - enables image input for this entry. |
| `--image-min-tokens` | `1024` | Minimum tokens budgeted per image (vision quality floor). |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `1024` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--spec-type` | `draft-mtp,ngram-mod` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `3` | Max tokens speculated per step. |
| `--spec-draft-p-min` | `0.3` | Draft acceptance probability threshold - tunes acceptance vs wasted drafts. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--chat-template-kwargs` | `'{"preserve_thinking":true}'` | JSON kwargs passed to the chat template (deprecated flag form). Used here to force thinking off server-side. |
| `--top-k` | `20` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `filters` | Gateway request filters applied before proxying to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Notes, incidents, upstream references

- Decode -29% vs the swift sibling entry (IQ-heavy quant mix on SYCL, measured 2026-10-19). Vision + MTP. Thinking defaults to xhigh.
- No supply-chain scan record - deployed under a documented, repo-scoped override (2026-10-19).
