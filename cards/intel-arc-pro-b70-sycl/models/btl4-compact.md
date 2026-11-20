# btl4-compact
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** BTL-4 Compact 35B-A3B (IQ2_XXS, SYCL, F16 KV, in-VRAM, 262k)

## Model card summary (from the deployed config)

- Type: MoE (qwen35moe, 256 experts / 8 active, hybrid attn) | agentic reasoning | text-only
- Parameter: 35.1B total / 2.1B active
- Source: https://huggingface.co/badtheorylabs/BTL-4-Compact
- Engine: SYCL (Level Zero) — fully in VRAM, no expert split
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Quant: IQ2_XXS 2.3 bpw, author quant (9.28 GiB)
- KV Cache: F16 @ 262144 (5.0 GiB)
- Context window: 262144 (native)
- Chat template: Qwen (--jinja; reasoning parsed)
- Sampling: temp 1.0 / top_p 0.95 / top_k 20 (card) — min_p pinned 0.0
- Notes: 2.3 bpw sibling of the RTX PRO 5000 box's btl4-35b (Q8_0); cross-check tool args there.

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.3125, 'output': 1.25, 'cached': 0.1562}`

## Config block

```yaml
btl4-compact:
  name: BTL-4 Compact 35B-A3B (IQ2_XXS, SYCL, F16 KV, in-VRAM, 262k)
  description: 'Type: MoE (qwen35moe, 256 experts / 8 active, hybrid attn) | agentic reasoning | text-only

    Parameter: 35.1B total / 2.1B active

    Source: https://huggingface.co/badtheorylabs/BTL-4-Compact

    Engine: SYCL (Level Zero) — fully in VRAM, no expert split

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Quant: IQ2_XXS 2.3 bpw, author quant (9.28 GiB)

    KV Cache: F16 @ 262144 (5.0 GiB)

    Context window: 262144 (native)

    Chat template: Qwen (--jinja; reasoning parsed)

    Sampling: temp 1.0 / top_p 0.95 / top_k 20 (card) — min_p pinned 0.0

    Notes: 2.3 bpw sibling of the RTX PRO 5000 box''s btl4-35b (Q8_0); cross-check tool args there.

    '
  cmd: '${llama-server-sycl-bin}

    -m ${gguf-models}/BTL-4-Compact/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -np 1

    -c 262144

    -b 4096 -ub 2048

    # 2026-11-14: KV F16 -> q8_0, same intelligence gate as qwen3.8 above

    # (byte-identical battery outputs; ppl within noise). Frees ~4 GiB.

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --reasoning-format deepseek

    --temp 1.0 --top-p 0.95 --top-k 20 --min-p 0.0

    '
  ttl: 600
  pricing:
    input: 0.3125
    output: 1.25
    cached: 0.1562
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/BTL-4-Compact/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-np` | `1` | Concurrent sequence slots served by this backend. |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `2048` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |
| `--temp` | `1.0` | Default sampling temperature (client per-request values override). |
| `--top-p` | `0.95` | Default top-p sampling. |
| `--top-k` | `20` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Notes, incidents, upstream references

- 2.3 bpw IQ2_XXS (9.28 GiB) fully in VRAM at 262k - the capacity tier for 35B-A3B on a 32 GB card.
- Tool-argument fidelity at 2-3 bpw was rejected twice in A/B on this class - not a tool-calling entry.
