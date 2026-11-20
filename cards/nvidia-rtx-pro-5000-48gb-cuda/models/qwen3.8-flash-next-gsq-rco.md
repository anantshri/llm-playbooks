# qwen3.8-flash-next-gsq-rco
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.8 Flash Next — GSQ-RCO CODER (expert-PRUNED 256/512, no draft, 256k)

## Model card summary (from the deployed config)

- Type: Expert-pruned MoE (qwen4exp; 256 of 512 routed experts/layer) | text-only
- ⚠️⚠️ WHICH MODEL IS THIS: **the CODER — expert-PRUNED, 256 of 512 routed
- experts retained.** The bare entry id `qwen3.8-flash-next-gsq-rco` carries
- no "coder" suffix, which is a naming trap, so verify from the artifact:
- - this file:  general.name = "Qwen3.8-Flash-Next GSQ-RCO Coder", expert_count = **256**
- - unpruned:   general.name = "Qwen3.8-Flash-Next GSQ-RCO",       expert_count = **512**
- The UNPRUNED siblings on this host are `qwen3.8-flash-next-q2-0` and
- `qwen3.8-flash-next-q2-0-abliterated`.
- ⚠️ Do NOT trust `general.size_label` — it reads `512x56B` on the Coder too
- (vendor label bug). `qwen4exp.expert_count` is the authoritative field.
- ⚠️ Nor the on-disk path: the directory is `Qwen3.8-Flash-Next-GSQ-RCO`
- with NO `-Coder` suffix, while the unpruned one is `...-GSQ-RCO-Unpruned`.
- Bare = Coder here, by accident of download order.
- Parameter: 176.9B original / ~6B active; 50% routed experts removed
- Source: ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-Coder-GGUF (target revision bba88856f8c3c0c9e40e45f82d05e228b814ac43; live cutover pending)
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672, kpool patch in-branch)
- Quant: GSQ-RCO Coder — mixed IQ4_NL/IQ2_S/IQ3_XXS/BF16 (IQ1_M tier name, zero IQ1_M tensors)
- KV Cache: q8_0 @ 262144 (~3.19 GiB; only 12 of 48 layers are full-attention)
- Context window: 262144 (native max)
- Speculative: none — exact greedy decoder, context maximised instead.
- VRAM: 35612 MiB of 48935 measured at load (bd_ctx_probe.sh, 2026-10-03)
- Vision: NONE — mmproj deliberately dropped (user directive 2026-10-03);
- frees ~1.10 GiB which is spent on context instead.
- No --n-cpu-moe: fully GPU-resident. Host-RAM offload costs ~0.8-1.0 GiB
- and ~5% prefill PER LAYER and was measured to buy no capability.

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/health`
- **pricing:** `{'input': 0.15, 'output': 0.47, 'cached': 0.016}`

## Config block

```yaml
qwen3.8-flash-next-gsq-rco:
  name: Qwen3.8 Flash Next — GSQ-RCO CODER (expert-PRUNED 256/512, no draft, 256k)
  description: "Type: Expert-pruned MoE (qwen4exp; 256 of 512 routed experts/layer) | text-only\n⚠️⚠️\
    \ WHICH MODEL IS THIS: **the CODER — expert-PRUNED, 256 of 512 routed\nexperts retained.** The bare\
    \ entry id `qwen3.8-flash-next-gsq-rco` carries\nno \"coder\" suffix, which is a naming trap, so verify\
    \ from the artifact:\n  - this file:  general.name = \"Qwen3.8-Flash-Next GSQ-RCO Coder\", expert_count\
    \ = **256**\n  - unpruned:   general.name = \"Qwen3.8-Flash-Next GSQ-RCO\",       expert_count = **512**\n\
    The UNPRUNED siblings on this host are `qwen3.8-flash-next-q2-0` and\n`qwen3.8-flash-next-q2-0-abliterated`.\n\
    ⚠️ Do NOT trust `general.size_label` — it reads `512x56B` on the Coder too\n(vendor label bug). `qwen4exp.expert_count`\
    \ is the authoritative field.\n⚠️ Nor the on-disk path: the directory is `Qwen3.8-Flash-Next-GSQ-RCO`\n\
    with NO `-Coder` suffix, while the unpruned one is `...-GSQ-RCO-Unpruned`.\nBare = Coder here, by\
    \ accident of download order.\nParameter: 176.9B original / ~6B active; 50% routed experts removed\n\
    Source: ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-Coder-GGUF (target revision bba88856f8c3c0c9e40e45f82d05e228b814ac43;\
    \ live cutover pending)\nEngine: CUDA (sm_120)\nInference: anantshri/llama.cpp (fleet-stable-20261021\
    \ @ 458f4d672, kpool patch in-branch)\nQuant: GSQ-RCO Coder — mixed IQ4_NL/IQ2_S/IQ3_XXS/BF16 (IQ1_M\
    \ tier name, zero IQ1_M tensors)\nKV Cache: q8_0 @ 262144 (~3.19 GiB; only 12 of 48 layers are full-attention)\n\
    Context window: 262144 (native max)\nSpeculative: none — exact greedy decoder, context maximised instead.\n\
    VRAM: 35612 MiB of 48935 measured at load (bd_ctx_probe.sh, 2026-10-03)\nVision: NONE — mmproj deliberately\
    \ dropped (user directive 2026-10-03);\nfrees ~1.10 GiB which is spent on context instead.\nNo --n-cpu-moe:\
    \ fully GPU-resident. Host-RAM offload costs ~0.8-1.0 GiB\nand ~5% prefill PER LAYER and was measured\
    \ to buy no capability.\n"
  cmd: '${llama-server-mtp-bin}

    -m ${gguf-models}/Qwen3.8-Flash-Next-GSQ-RCO/IQ1_M/Qwen3.8-Flash-Next-GSQ-RCO-IQ1_M-00001-of-00002.gguf

    --alias qwen3.8-flash-next-gsq-rco

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -lm mmap -lzm on

    --cache-type-k q8_0 --cache-type-v q8_0

    -c 262144

    -b 1024 -ub 256

    --jinja

    --top-k 20

    --min-p 0.0

    '
  checkEndpoint: /health
  ttl: 600
  pricing:
    input: 0.15
    output: 0.47
    cached: 0.016
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Qwen3.8-Flash-Next-GSQ-RCO/IQ1_M/Qwen3.8-Flash-Next-GSQ-RCO-IQ1_M-00001-of-00002.gguf` | Weights file (GGUF) this entry serves. |
| `--alias` | `qwen3.8-flash-next-gsq-rco` | Model name the backend reports - kept identical to the entry id clients request. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-lm` | `mmap` | Flash-Next trunk build flag (mmap-based weight loading variant) - build-specific, see that build's --help. |
| `-lzm` | `on` | Flash-Next trunk build flag (paired with -lm) - build-specific, see that build's --help. |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `1024` | Logical batch size (prompt-processing chunk). |
| `-ub` | `256` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--top-k` | `20` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Measured (llama-bench)

> llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47). tg@9k is shallow decode; tg@max is the deep-decode walk (~98% of -c). Do not mix with gateway measurements (see troubleshooting).

| Metric | Value |
|---|---|
| pp @9k | 2121 tok/s |
| pp @max (254k ctx) | 735 tok/s |
| tg @9k (shallow) | 57.4 tok/s |
| tg @max (254k ctx) | 24.1 tok/s |

## Notes, incidents, upstream references

- Expert-PRUNED Coder: 256 of 512 routed experts (qwen4exp.expert_count=256 - do not trust size_label). Old repetition/speed verdicts do not transfer to the refreshed weights (provisioning targets rev bba88856, 2026-10-19).
