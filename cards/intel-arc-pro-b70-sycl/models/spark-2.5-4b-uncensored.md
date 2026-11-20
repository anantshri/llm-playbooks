# spark-2.5-4b-uncensored
**Card:** Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)  |  **Engine:** llama.cpp

**Display name:** UNCENSORED Spark X2.5 4B (BF16, SYCL, F16 KV, 131k)

## Model card summary (from the deployed config)

- Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on
- Parameter: 4B
- Source: https://huggingface.co/toandev/Spark-X2.5-4B-Uncensored-GGUF (revision 8d13862e2cfa9e1e67b1f19bf17759f41bd583f2)
- Engine: SYCL (Level Zero)
- Inference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)
- Env: GGML_SYCL_FA_DPAS_F16=1 (dpas F16 arm, via ${llama-server-spark25u-bin}; ~+5% class per pentester sweep)
- Quant: Q8_0, author quant (4.73 GB) — NOTE: this host's spark-2.5-4b sibling is BF16, so quant is a confound in any A/B here
- KV Cache: F16 @ 131072
- Context window: 131072 (native 1048576)
- Safety gate: insights-db.paloaltonetworks.com — **Safe, scanned 2026-09-09,
- version 8d1386**; the repo's current main sha is
- 8d13862e2cfa9e1e67b1f19bf17759f41bd583f2, so the verdict covers the EXACT
- revision pinned here, not a sibling. License apache-2.0.
- Provenance: `toandev` is OFF the allowlist. User requested this exact repo
- by URL on 2026-10-03 -> repo-scoped approval, THIS REPO ONLY.
- Purpose: uncensored sibling of `spark-2.5-4b` for authorized pentesting
- work (user directive 2026-10-03). Flags cloned 1:1 from this host's own
- `spark-2.5-4b` so the MODEL is the only variable.
- Chat template: embedded (--jinja)

## llama-swap entry

- **ttl:** `600`
- **pricing:** `{'input': 0.05, 'output': 0.1}`

## Config block

```yaml
spark-2.5-4b-uncensored:
  name: UNCENSORED Spark X2.5 4B (BF16, SYCL, F16 KV, 131k)
  description: "Type: Dense hybrid attention (1 full + 3 SWA per group) | thinking default-on\nParameter:\
    \ 4B\nSource: https://huggingface.co/toandev/Spark-X2.5-4B-Uncensored-GGUF (revision 8d13862e2cfa9e1e67b1f19bf17759f41bd583f2)\n\
    Engine: SYCL (Level Zero)\nInference: anantshri/llama.cpp (fleet-opt-20261110 @ 1438246ad + 0001-li-tile\
    \ + 0001-dpas-f16 (incl. Q80_KG retile; binary = lc-q80-tune)\nEnv: GGML_SYCL_FA_DPAS_F16=1 (dpas\
    \ F16 arm, via ${llama-server-spark25u-bin}; ~+5% class per pentester sweep)\nQuant: Q8_0, author\
    \ quant (4.73 GB) — NOTE: this host's spark-2.5-4b sibling is BF16, so quant is a confound in any\
    \ A/B here\nKV Cache: F16 @ 131072\nContext window: 131072 (native 1048576)\nSafety gate: insights-db.paloaltonetworks.com\
    \ — **Safe, scanned 2026-09-09,\n  version 8d1386**; the repo's current main sha is\n  8d13862e2cfa9e1e67b1f19bf17759f41bd583f2,\
    \ so the verdict covers the EXACT\n  revision pinned here, not a sibling. License apache-2.0.\nProvenance:\
    \ `toandev` is OFF the allowlist. User requested this exact repo\n  by URL on 2026-10-03 -> repo-scoped\
    \ approval, THIS REPO ONLY.\nPurpose: uncensored sibling of `spark-2.5-4b` for authorized pentesting\n\
    \  work (user directive 2026-10-03). Flags cloned 1:1 from this host's own\n  `spark-2.5-4b` so the\
    \ MODEL is the only variable.\nChat template: embedded (--jinja)\n"
  cmd: '${llama-server-spark25u-bin}

    -m ${gguf-models}/Spark-X2.5-4B-Uncensored/Spark-X2.5-4B-Uncensored-Q8_0.gguf

    --host 127.0.0.1 --port ${PORT}

    -ngl 999

    -c 131072

    -ub 2048

    --cache-type-k f16 --cache-type-v f16

    --flash-attn on

    --jinja

    --top-k 0

    --min-p 0.0

    --reasoning-format deepseek

    '
  ttl: 600
  pricing:
    input: 0.05
    output: 0.1
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `-m` | `${gguf-models}/Spark-X2.5-4B-Uncensored/Spark-X2.5-4B-Uncensored-Q8_0.gguf` | Weights file (GGUF) this entry serves. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |
| `-ngl` | `999` | Layers offloaded to GPU; 999 = everything. |
| `-c` | `131072` | KV context window in tokens. Chosen by measured VRAM fit (see notes). |
| `-ub` | `2048` | Physical micro-batch per kernel - the tuned prefill lever (measured +13-18% prefill at 2048/512 on the heavy MoE entries, 2026-11). |
| `--cache-type-k` | `f16` | KV cache dtype for K (f16/q8_0/q4_0) - the per-entry KV policy. Past ~100k tokens KV dtype is the decode bottleneck (measured 2026-09-08). |
| `--cache-type-v` | `f16` | KV cache dtype for V - same policy as --cache-type-k. |
| `--flash-attn` | `on` | Flash-attention kernels on. |
| `--jinja` | _(flag)_ | Use the model's own Jinja chat template instead of a builtin. |
| `--top-k` | `0` | Default top-k sampling. |
| `--min-p` | `0.0` | Default min-p sampling. |
| `--reasoning-format` | `deepseek` | How reasoning blocks are exposed to clients (e.g. deepseek-style <think> parsing). |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `pricing` | Per-1M-token pricing used by the gateway stats page for cost estimates. |

## Notes, incidents, upstream references

- `ggml-org/llama.cpp#30606` (open) - D=256 FA vector decode routing on Battlemage (same arch class).
