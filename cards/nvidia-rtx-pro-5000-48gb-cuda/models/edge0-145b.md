# edge0-145b
**Card:** NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)  |  **Engine:** other

**Display name:** Edge0 145B (NVFP4 native, edge0-strata engine, 131k)

## Model card summary (from the deployed config)

- Type: latent sparse MoE (48 layers, 512 experts / 10 routed + 1 shared, latent width 1792, hybrid QSA attention + fp8 PLE n-gram table) | thinking default-on | text-only
- Parameter: 145B total / 10.1B active
- Source: https://huggingface.co/Edge0/Edge0-145B-NVFP4
- Engine: CUDA (sm_120) — edge0-strata (NVFP4 experts; VRAM-resident cache, remainder streamed over PCIe from host RAM)
- Inference: Edge0-AI/Edge0 (edge0-infer @ da9cc526)
- Quant: native-trained NVFP4 (47.8 GiB expert pool; fp8 PLE table SSD-read; pack ~105 GiB total)
- KV Cache: int8 @ 131072 (32k resident; remainder pinned host RAM)
- Context window: 131072 (native 262144)
- Chat template: Qwen (engine built-in renderer; thinking default-on)
- VRAM: expert cache auto-sized from free VRAM (~43 GiB class sole-resident) + ~4 GiB fixed skeleton/MTP/buffers (vram-reserve 700 MiB default)

## llama-swap entry

- **ttl:** `1800`
- **checkEndpoint:** `/health`

## Config block

```yaml
edge0-145b:
  name: Edge0 145B (NVFP4 native, edge0-strata engine, 131k)
  description: 'Type: latent sparse MoE (48 layers, 512 experts / 10 routed + 1 shared, latent width 1792,
    hybrid QSA attention + fp8 PLE n-gram table) | thinking default-on | text-only

    Parameter: 145B total / 10.1B active

    Source: https://huggingface.co/Edge0/Edge0-145B-NVFP4

    Engine: CUDA (sm_120) — edge0-strata (NVFP4 experts; VRAM-resident cache, remainder streamed over
    PCIe from host RAM)

    Inference: Edge0-AI/Edge0 (edge0-infer @ da9cc526)

    Quant: native-trained NVFP4 (47.8 GiB expert pool; fp8 PLE table SSD-read; pack ~105 GiB total)

    KV Cache: int8 @ 131072 (32k resident; remainder pinned host RAM)

    Context window: 131072 (native 262144)

    Chat template: Qwen (engine built-in renderer; thinking default-on)

    VRAM: expert cache auto-sized from free VRAM (~43 GiB class sole-resident) + ~4 GiB fixed skeleton/MTP/buffers
    (vram-reserve 700 MiB default)

    '
  cmd: '${edge0-bin}

    --model ${edge0-models}/Edge0-145B-NVFP4

    --alias edge0-145b

    --host 127.0.0.1 --port ${PORT}

    '
  checkEndpoint: /health
  ttl: 1800
```

## cmd parameters (this entry's engine invocation)

argv[0]: wrapper/engine binary (see the card macros table)

| Flag | Value here | What it does |
|---|---|---|
| `--model` | `${edge0-models}/Edge0-145B-NVFP4` | Weights/checkpoint this entry serves. |
| `--alias` | `edge0-145b` | Model name the backend reports - kept identical to the entry id clients request. |
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |

## Notes, incidents, upstream references

- 145B-total / ~10B-active latent sparse MoE, native NVFP4, in-pack MTP, 256k ctx. Engine is CUDA 13 + Metal only (no SYCL backend); pack is Strata format, not GGUF (~105 GiB on disk).
- License: community license permits internal use; external/commercial serving needs a separate license.
- Card holds ~85-90% of the expert pool VRAM-resident. Staged 2026-11-13; cold load must beat healthCheckTimeout 600; GPU smoke pending.
