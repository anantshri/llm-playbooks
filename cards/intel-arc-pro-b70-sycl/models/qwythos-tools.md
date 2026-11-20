# qwythos-tools
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** Qwythos-9B (Q8_0, SYCL, F16 KV, non-thinking — agent tools, 131k)

## Model card summary (from the deployed config)

- Type: Dense hybrid (qwen3_5) | abliterated (Claude-Mythos) | non-thinking agent-tools
- Parameter: 9B total / 9B active (dense)
- Source: https://huggingface.co/Securelayer7/Qwythos-9B-Claude-Mythos-Uncensored-Abliterated-1M
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Quant: self-converted Q8_0 (9.1 GB)
- KV Cache: F16 @ 131072 (~2.6 GiB)
- Context window: 131072 (1M-capable via YaRN)
- Chat template: Qwen (--jinja; thinking forced off)
- Sampling: card (temp 0.6, top_p 0.95, top_k 20, min_p 0, rep-penalty 1.05)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.1, 'output': 0.15}`

## Config block

```yaml
qwythos-tools:
  name: Qwythos-9B (Q8_0, SYCL, F16 KV, non-thinking — agent tools, 131k)
  description: 'Type: Dense hybrid (qwen3_5) | abliterated (Claude-Mythos) | non-thinking agent-tools

    Parameter: 9B total / 9B active (dense)

    Source: https://huggingface.co/Securelayer7/Qwythos-9B-Claude-Mythos-Uncensored-Abliterated-1M

    Engine: SYCL (Level Zero)

    Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl.
    Q80_KG retile; binary = lc-q80-tune)

    Quant: self-converted Q8_0 (9.1 GB)

    KV Cache: F16 @ 131072 (~2.6 GiB)

    Context window: 131072 (1M-capable via YaRN)

    Chat template: Qwen (--jinja; thinking forced off)

    Sampling: card (temp 0.6, top_p 0.95, top_k 20, min_p 0, rep-penalty 1.05)

    '
  cmd: '${llama-server-sycl-bin}

    -m ${gguf-models}/Qwythos-9B-Q8/current.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -b 4096 -ub 2048

    # 2026-11-14: KV F16 -> q8_0, same intelligence gate as qwen3.8 above

    # (byte-identical battery outputs; ppl within noise). Frees ~4 GiB.

    --cache-type-k q8_0 --cache-type-v q8_0

    --flash-attn on

    --jinja

    --chat-template-kwargs ''{"enable_thinking":false}''

    --temp 0.6

    --top-p 0.95

    --top-k 20

    --min-p 0.0

    --repeat-penalty 1.05

    '
  ttl: 600
  pricing:
    input: 0.1
    output: 0.15
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Qwythos-9B-Q8/current.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-b` | `4096` | Logical batch size (prompt-processing chunk). |
| `-ub` | `2048` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--chat-template-kwargs` | `'{"enable_thinking":false}'` | JSON kwargs passed to the chat template (deprecated flag form). Used here to force thinking off server-side. |
| `--temp` | `0.6` | Default sampling temperature (client per-request values override). |
| `--top-p` | `0.95` | Default top-p sampling. |
| `--top-k` | `20` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |
| `--repeat-penalty` | `1.05` | Default repetition penalty. Abliterated entries loop without 1.1+ (model-card requirement). |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |
