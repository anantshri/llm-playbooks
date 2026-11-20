# qwen3.8-flash-next-q2-0
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** llama.cpp

**Display name:** Qwen3.8 Flash Next — Q2_0 UNPRUNED (all 512 experts, no draft, 256k max ctx)

## Model card summary (from the deployed config)

- Type: Sparse MoE (qwen4exp; all 512 routed experts, 10 active) | text-only
- ⚠️ WHICH MODEL IS THIS: **UNPRUNED — all 512 routed experts**
- (`qwen4exp.expert_count = 512`). The expert-PRUNED Coder is the
- separate entry `qwen3.8-flash-next-gsq-rco` (expert_count 256).
- Parameter: 176.9B total / ~6B active
- Source: ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF @ ed59f92082
- Engine: CUDA (sm_120)
- Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672, kpool patch in-branch)
- Quant: GSQ-RCO Q2_0 2.40 bpw (unpruned, all 512 experts)
- KV Cache: q8_0 @ 262144 (~3.19 GiB; only 12 of 48 layers are full-attention)
- Context window: 262144 (native max)
- Speculative: none — exact greedy decoder, context maximised instead.
- VRAM: 43316 MiB of 48935 measured at load (bd_ctx_probe.sh, 2026-10-03)
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
qwen3.8-flash-next-q2-0:
  name: Qwen3.8 Flash Next — Q2_0 UNPRUNED (all 512 experts, no draft, 256k max ctx)
  description: 'Type: Sparse MoE (qwen4exp; all 512 routed experts, 10 active) | text-only

    ⚠️ WHICH MODEL IS THIS: **UNPRUNED — all 512 routed experts**

    (`qwen4exp.expert_count = 512`). The expert-PRUNED Coder is the

    separate entry `qwen3.8-flash-next-gsq-rco` (expert_count 256).

    Parameter: 176.9B total / ~6B active

    Source: ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF @ ed59f92082

    Engine: CUDA (sm_120)

    Inference: anantshri/llama.cpp (fleet-stable-20261021 @ 458f4d672, kpool patch in-branch)

    Quant: GSQ-RCO Q2_0 2.40 bpw (unpruned, all 512 experts)

    KV Cache: q8_0 @ 262144 (~3.19 GiB; only 12 of 48 layers are full-attention)

    Context window: 262144 (native max)

    Speculative: none — exact greedy decoder, context maximised instead.

    VRAM: 43316 MiB of 48935 measured at load (bd_ctx_probe.sh, 2026-10-03)

    Vision: NONE — mmproj deliberately dropped (user directive 2026-10-03);

    frees ~1.10 GiB which is spent on context instead.

    No --n-cpu-moe: fully GPU-resident. Host-RAM offload costs ~0.8-1.0 GiB

    and ~5% prefill PER LAYER and was measured to buy no capability.

    '
  cmd: '${llama-server-mtp-bin}

    -m ${gguf-models}/Qwen3.8-Flash-Next-GSQ-RCO-Unpruned/Q2_0/Qwen3.8-Flash-Next-GSQ-RCO-Q2_0-00001-of-00002.gguf

    --alias qwen3.8-flash-next-q2-0

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
| `-m` | `${gguf-models}/Qwen3.8-Flash-Next-GSQ-RCO-Unpruned/Q2_0/Qwen3.8-Flash-Next-GSQ-RCO-Q2_0-00001-of-00002.gguf` | Weights file (GGUF) this entry serves. |
| `--alias` | `qwen3.8-flash-next-q2-0` | Model name the backend reports - kept identical to the entry id clients request. |
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
