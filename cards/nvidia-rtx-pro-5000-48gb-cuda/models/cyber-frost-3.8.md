# cyber-frost-3.8
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** CYBER-FROST-3.8 (Blackfrost security fine-tune of Flash-Next, self-quant Q4_K_M, 262K)

## Model card summary (from the deployed config)

- Type: Sparse MoE (Gated DeltaNet 3:1 + QSA sparse attention budget 2048; 512 experts top-10 + shared + 51B n-gram PLE embedding table) | security fine-tune | text-only
- Parameter: ~180B total / ~6B active
- Source: https://huggingface.co/Blackfrost-AI/CYBER-FROST-3.8-BF16 (rev 7fc56ac4, self-quantized here)
- Base: Qwen/Qwen3.8-Flash-Next @ de4b8e4d
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)
- Quant: self-quant Q4_K_M (110.96 GiB) — NOT unsloth UD; no imatrix
- KV Cache: Q8_0 @ 262144
- Context window: 262144 (card exercised only 32768 — longer is unqualified)
- Chat template: Blackfrost (--jinja); reasoning auto-parsed
- Reasoning: ALWAYS ON — template clobbers enable_thinking; send max_tokens >= 1500 or content comes back null
- Sampling: card defaults temp 1.0 / top-p 0.95 / top-k 20
- VRAM: 44956 of 48935 MiB measured at --n-cpu-moe 26 (~73 GB host RAM)

## llama-swap entry

- **ttl:** `600`
- **checkEndpoint:** `/health`
- **pricing:** `{'input': 0.15, 'output': 0.47, 'cached': 0.016}`

## Config block

```yaml
cyber-frost-3.8:
  name: CYBER-FROST-3.8 (Blackfrost security fine-tune of Flash-Next, self-quant Q4_K_M, 262K)
  description: 'Type: Sparse MoE (Gated DeltaNet 3:1 + QSA sparse attention budget 2048; 512 experts top-10
    + shared + 51B n-gram PLE embedding table) | security fine-tune | text-only

    Parameter: ~180B total / ~6B active

    Source: https://huggingface.co/Blackfrost-AI/CYBER-FROST-3.8-BF16 (rev 7fc56ac4, self-quantized here)

    Base: Qwen/Qwen3.8-Flash-Next @ de4b8e4d

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672)

    Quant: self-quant Q4_K_M (110.96 GiB) — NOT unsloth UD; no imatrix

    KV Cache: Q8_0 @ 262144

    Context window: 262144 (card exercised only 32768 — longer is unqualified)

    Chat template: Blackfrost (--jinja); reasoning auto-parsed

    Reasoning: ALWAYS ON — template clobbers enable_thinking; send max_tokens >= 1500 or content comes
    back null

    Sampling: card defaults temp 1.0 / top-p 0.95 / top-k 20

    VRAM: 44956 of 48935 MiB measured at --n-cpu-moe 26 (~73 GB host RAM)

    '
  cmd: '${llama-server-bin}

    -m ${gguf-models}/CYBER-FROST-3.8/current.gguf

    --alias cyber-frost-3.8

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    --n-cpu-moe 26

    --cache-type-k q8_0 --cache-type-v q8_0

    -c 262144

    --jinja

    --temp 1.0

    --top-p 0.95

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
| `-m` | `${gguf-models}/CYBER-FROST-3.8/current.gguf` | Weights file (GGUF) this entry serves. |
| `--alias` | `cyber-frost-3.8` | Model name the backend reports - kept identical to the entry id clients request. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `--n-cpu-moe` | `26` | Keep N MoE expert layers on CPU RAM - the VRAM-fit lever for oversized MoE models; costs host RAM and prefill speed. |
| `--cache-type-k` | `q8_0` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `q8_0` | KV cache dtype for V - same policy as --cache-type-k. |
| `-c` | `262144` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--temp` | `1.0` | Default sampling temperature (client per-request values override). |
| `--top-p` | `0.95` | Default top-p sampling. |
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

- Thinking is unconditional: max_tokens >=1500 or content comes back null. --n-cpu-moe 26, ~73 GB host RAM. Provenance override repo-scoped.
