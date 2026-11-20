# muse-glimmer-30b
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Muse Glimmer 30B (Q8_0 GGUF, vision + DFlash, 262k)

## Model card summary (from the deployed config)

- Type: Dense MULTIMODAL (muse-glimmer: 52 layers, 32Q/2KV GQA 16:1, hybrid [Local,Local,Local,Global] sliding-window 2048 — only 13/52 layers Global)
- Parameter: 29.6B dense + ViT-G/14 perception encoder ~1.8B (50 layers, 4096 visual tokens/image)
- Source: https://huggingface.co/unsloth/Muse-Glimmer-30B-GGUF
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: Q8_0 (27.6 GB) + mmproj-Q8_0 (1.9 GB) + dflash-kquant drafter (1.5 GB)
- KV Cache: F16 @ 262144 (~3.5 GB)
- Context window: 262144 (override)
- Chat template: Muse (--jinja; reasoning_strength default high, no parser)
- VRAM: ~33 GB (~15 GB headroom)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.3, 'output': 1.1, 'cached': 0.04}`

## Config block

```yaml
muse-glimmer-30b:
  name: Muse Glimmer 30B (Q8_0 GGUF, vision + DFlash, 262k)
  description: 'Type: Dense MULTIMODAL (muse-glimmer: 52 layers, 32Q/2KV GQA 16:1, hybrid [Local,Local,Local,Global]
    sliding-window 2048 — only 13/52 layers Global)

    Parameter: 29.6B dense + ViT-G/14 perception encoder ~1.8B (50 layers, 4096 visual tokens/image)

    Source: https://huggingface.co/unsloth/Muse-Glimmer-30B-GGUF

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: Q8_0 (27.6 GB) + mmproj-Q8_0 (1.9 GB) + dflash-kquant drafter (1.5 GB)

    KV Cache: F16 @ 262144 (~3.5 GB)

    Context window: 262144 (override)

    Chat template: Muse (--jinja; reasoning_strength default high, no parser)

    VRAM: ~33 GB (~15 GB headroom)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/Muse-Glimmer-30B/current.gguf

    --mmproj ${gguf-models}/Muse-Glimmer-30B/mmproj-Muse-Glimmer-30B-Q8_0.gguf

    -md ${gguf-models}/Muse-Glimmer-30B/dflash-kquant.gguf

    --spec-type draft-dflash

    --spec-draft-n-max 16

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -np 1

    -c 262144

    --override-kv muse-glimmer.context_length=int:262144

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --temp 1.0 --top-p 0.95 --top-k 64

    --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.3
    output: 1.1
    cached: 0.04
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Muse-Glimmer-30B/current.gguf` | Weights file (GGUF) this entry serves. |
| `--mmproj` | `${gguf-models}/Muse-Glimmer-30B/mmproj-Muse-Glimmer-30B-Q8_0.gguf` | Vision projector file - enables image input for this entry. |
| `-md` | `${gguf-models}/Muse-Glimmer-30B/dflash-kquant.gguf` | Drafter weights for speculative decoding (MTP drafter GGUF, DFlash drafter, etc.). |
| `--spec-type` | `draft-dflash` | Speculative decoding mode: draft (separate drafter), mtp (in-pack MTP heads), ngram-mod (n-gram drafting for verbatim-repetitive output). |
| `--spec-draft-n-max` | `16` | Max tokens speculated per step. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-np` | `1` | Concurrent sequence slots served by this backend. |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--override-kv` | `muse-glimmer.context_length=int:262144` | Override a GGUF metadata key at load (here: the model's declared context length). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--temp` | `1.0` | Default sampling temperature (client per-request values override). |
| `--top-p` | `0.95` | Default top-p sampling. |
| `--top-k` | `64` | Default top-k sampling. |
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
| pp @9k | 2316 tok/s |
| pp @max (244k ctx) | 1633 tok/s |
| tg @9k (shallow) | 88 tok/s |
| tg @max (244k ctx) | 82.4 tok/s |
