# qwen3.8-flash-next-gsq-rco
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.8 Flash Next — GSQ-RCO CODER (expert-PRUNED 256/512, SYCL, text-only, 128k)

## Model card summary (from the deployed config)

- Type: Expert-pruned MoE (qwen4exp, 256 of 512 routed experts/layer, 10 active + shared, QSA/GDN) | code/agentic
- ⚠️⚠️ WHICH MODEL IS THIS: **the CODER — expert-PRUNED, 256 of 512 routed
- experts.** The bare entry id carries no "coder" suffix. Verify from the
- artifact: general.name = "Qwen3.8-Flash-Next GSQ-RCO Coder",
- qwen4exp.expert_count = **256** (the unpruned build reports 512).
- ⚠️ Do NOT trust general.size_label — it reads `512x56B` on the Coder too
- (vendor label bug); expert_count is authoritative. The on-disk directory
- is `Qwen3.8-Flash-Next-GSQ-RCO` with NO `-Coder` suffix.
- ⚠️ The unpruned Q2_0 does NOT fit this card (36.6 GiB vs ~32.6 usable),
- so on the Arc B70 box there is no unpruned sibling to confuse this with.
- Parameter: 176.9B original / ~6B active; 50% routed experts removed
- Source: https://huggingface.co/ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-Coder-GGUF (target revision bba88856f8c3c0c9e40e45f82d05e228b814ac43; live cutover pending)
- Engine: SYCL (Level Zero), F32 attention scores
- Quant: GSQ-RCO mixed IQ4_NL/IQ2_S/IQ3_XXS/BF16 (IQ1_M tier name; zero IQ1_M tensors); 29.6 GB weights + 28.8 GB disk-backed n-gram shard
- KV Cache: Q8_0 @ 131072 (~1.59 GiB — only 12 of 48 layers are full-attention,
- so KV is cheap on this arch: 12.75 KiB/token at q8_0, 6.75 at q4_0)
- Context window: 131072 (native 262144) — MEASURED CEILING for q8_0 KV on the
- B70, not a guess: the Arc B70 box_kv_probe.sh walked 262144/196608/131072 on
- 2026-10-03 and q8_0 loads at 131072 but fails at 196608 and 262144.
- Vision: NONE — mmproj dropped 2026-10-03 (user: "for the Arc B70 box we dont need
- vision"), which is what paid for 65536 -> 131072.
- ⚠️ No MTP on this host (user directive 2026-10-03), and it would not fit
- anyway: the draft head costs a FLAT 3496 MiB (measured on the RTX PRO 5000 box, not
- the 2.60 GiB previously recorded), and 29.1 + 3.41 GiB leaves zero margin
- against ~32.6 usable. MTP also does not ADD context — the head consumes
- VRAM that would otherwise hold KV.
- ⚠️ q4_0 KV would reach 196608 (also measured; q4_0 loads at 196608/163840,
- fails at 262144). That is +50% context for lower attention fidelity —
- a deliberate tradeoff, left for the user to pick. Neither dtype reaches
- the native 262144 on this card.
- Chat template: Qwen (--jinja); xhigh reasoning and tools validated

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/health`
- **pricing:** `{'input': 0.15, 'output': 0.47, 'cached': 0.016}`

## Config block

```yaml
qwen3.8-flash-next-gsq-rco:
  name: Qwen3.8 Flash Next — GSQ-RCO CODER (expert-PRUNED 256/512, SYCL, text-only, 128k)
  description: "Type: Expert-pruned MoE (qwen4exp, 256 of 512 routed experts/layer, 10 active + shared,\
    \ QSA/GDN) | code/agentic\n⚠️⚠️ WHICH MODEL IS THIS: **the CODER — expert-PRUNED, 256 of 512 routed\n\
    experts.** The bare entry id carries no \"coder\" suffix. Verify from the\nartifact: general.name\
    \ = \"Qwen3.8-Flash-Next GSQ-RCO Coder\",\nqwen4exp.expert_count = **256** (the unpruned build reports\
    \ 512).\n⚠️ Do NOT trust general.size_label — it reads `512x56B` on the Coder too\n(vendor label bug);\
    \ expert_count is authoritative. The on-disk directory\nis `Qwen3.8-Flash-Next-GSQ-RCO` with NO `-Coder`\
    \ suffix.\n⚠️ The unpruned Q2_0 does NOT fit this card (36.6 GiB vs ~32.6 usable),\nso on the Arc\
    \ B70 box there is no unpruned sibling to confuse this with.\nParameter: 176.9B original / ~6B active;\
    \ 50% routed experts removed\nSource: https://huggingface.co/ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-Coder-GGUF\
    \ (target revision bba88856f8c3c0c9e40e45f82d05e228b814ac43; live cutover pending)\nEngine: SYCL (Level\
    \ Zero), F32 attention scores\nQuant: GSQ-RCO mixed IQ4_NL/IQ2_S/IQ3_XXS/BF16 (IQ1_M tier name; zero\
    \ IQ1_M tensors); 29.6 GB weights + 28.8 GB disk-backed n-gram shard\nKV Cache: Q8_0 @ 131072 (~1.59\
    \ GiB — only 12 of 48 layers are full-attention,\n  so KV is cheap on this arch: 12.75 KiB/token at\
    \ q8_0, 6.75 at q4_0)\nContext window: 131072 (native 262144) — MEASURED CEILING for q8_0 KV on the\n\
    \  B70, not a guess: the Arc B70 box_kv_probe.sh walked 262144/196608/131072 on\n  2026-10-03 and\
    \ q8_0 loads at 131072 but fails at 196608 and 262144.\nVision: NONE — mmproj dropped 2026-10-03 (user:\
    \ \"for the Arc B70 box we dont need\n  vision\"), which is what paid for 65536 -> 131072.\n⚠️ No\
    \ MTP on this host (user directive 2026-10-03), and it would not fit\n  anyway: the draft head costs\
    \ a FLAT 3496 MiB (measured on the RTX PRO 5000 box, not\n  the 2.60 GiB previously recorded), and\
    \ 29.1 + 3.41 GiB leaves zero margin\n  against ~32.6 usable. MTP also does not ADD context — the\
    \ head consumes\n  VRAM that would otherwise hold KV.\n⚠️ q4_0 KV would reach 196608 (also measured;\
    \ q4_0 loads at 196608/163840,\n  fails at 262144). That is +50% context for lower attention fidelity\
    \ —\n  a deliberate tradeoff, left for the user to pick. Neither dtype reaches\n  the native 262144\
    \ on this card.\nChat template: Qwen (--jinja); xhigh reasoning and tools validated\n"
  cmd: '${llama-server-flashnext-bin}

    -m ${gguf-models}/Qwen3.8-Flash-Next-GSQ-RCO/IQ1_M/Qwen3.8-Flash-Next-GSQ-RCO-IQ1_M-00001-of-00002.gguf

    --alias qwen3.8-flash-next-gsq-rco

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -np 1 -c 131072

    # 2026-11-10: -b 1024/-ub 256 -> -b 2048/-ub 512 — measured on fleet-opt-20261110:

    # +13.0% pp@8k / +18.4% pp@32k / +18.5% pp@131k (674 t/s derived), tg flat,

    # 131k VRAM-verified (llama-bench rc=0 with VRAM check). Continues the ub ladder

    # direction (128->256 was +45% in the 2026-10-03 capacity test).

    -b 2048 -ub 512

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    -lm mmap -lzm on

    --jinja

    --reasoning-effort xhigh

    --top-k 20 --min-p 0.0

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
| `-np` | `1` | Concurrent sequence slots served by this backend. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `2048` | Logical batch size (prompt-processing chunk). |
| `-ub` | `512` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `-lm` | `mmap` | Flash-Next trunk build flag (mmap-based weight loading variant) - build-specific, see that build's --help. |
| `-lzm` | `on` | Flash-Next trunk build flag (paired with -lm) - build-specific, see that build's --help. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--reasoning-effort` | `xhigh` | Server-side default thinking effort (clients can override per request). |
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

## Notes, incidents, upstream references

- `ggml-org/llama.cpp#29107` (merged 2026-10-14) - Persistent IQ3 reorder: IQ3 tg256@9k 12.30 -> 21.59 tok/s on this card.
- `ggml-org/llama.cpp#30604` (merged 2026-10-21) - Fixes the NaN/garbage crash this IQ3 quant class hit after the reorder (fused-dequant GEMM misread reordered bytes). 42/42 clean under concurrent load.
