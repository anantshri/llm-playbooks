# btl4-compact
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** BTL-4 Compact 35B-A3B (IQ2_XXS 2.3 bpw, author quant, 262k)

## Model card summary (from the deployed config)

- Type: MoE (qwen35moe, 256 experts / 8 active, hybrid attn) | agentic reasoning | text-only
- Parameter: 35.1B total / 2.1B active
- Source: https://huggingface.co/badtheorylabs/BTL-4-Compact
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: IQ2_XXS 2.3 bpw, author quant (9.28 GiB)
- KV Cache: Q8_0 @ 262144 (2.7 GiB)
- Context window: 262144 (native)
- Chat template: Qwen (--jinja; reasoning parsed)
- VRAM: 13.2 GB of 48 (measured, ample headroom)
- Sampling: temp 1.0 / top_p 0.95 / top_k 20 (card) — min_p pinned 0.0
- Measured: 217.3 tok/s decode, 187.1 prefill (2026-09-08, shallow)
- Notes: low-fidelity A/B control for btl4-35b (Q8_0 + vision); prefer that entry for tool args.

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.3125, 'output': 1.25, 'cached': 0.1562}`

## Config block

```yaml
btl4-compact:
  name: BTL-4 Compact 35B-A3B (IQ2_XXS 2.3 bpw, author quant, 262k)
  description: 'Type: MoE (qwen35moe, 256 experts / 8 active, hybrid attn) | agentic reasoning | text-only

    Parameter: 35.1B total / 2.1B active

    Source: https://huggingface.co/badtheorylabs/BTL-4-Compact

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: IQ2_XXS 2.3 bpw, author quant (9.28 GiB)

    KV Cache: Q8_0 @ 262144 (2.7 GiB)

    Context window: 262144 (native)

    Chat template: Qwen (--jinja; reasoning parsed)

    VRAM: 13.2 GB of 48 (measured, ample headroom)

    Sampling: temp 1.0 / top_p 0.95 / top_k 20 (card) — min_p pinned 0.0

    Measured: 217.3 tok/s decode, 187.1 prefill (2026-09-08, shallow)

    Notes: low-fidelity A/B control for btl4-35b (Q8_0 + vision); prefer that entry for tool args.

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/BTL-4-Compact/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 262144

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
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
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
