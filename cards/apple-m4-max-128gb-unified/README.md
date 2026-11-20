# Apple M4 Max (128 GB unified memory)

**Backend:** llama.cpp Metal + MLX (mlx-lm); gateway llama-swap fork on :9567  
**VRAM:** 128 GB unified (shared with OS; swap risk under memory churn)

## Global gateway settings (from the deployed config)

| Key | Value | Meaning |
|---|---|---|
| `healthCheckTimeout` | `120` | Max seconds to wait for a cold backend to become healthy before failing the request. |
| `startPort` | `5800` | First internal proxy port; each loaded backend gets the next port. |
| `captureBuffer` | `100` | Bytes of request/response captured per call for the stats UI. |
| `logLevel` | `info` | Gateway log verbosity. |
| `store` | `{'path': '~/inference/llama-swap.sqlite'}` | Path to the gateway's sqlite persistence DB. |

## Swap groups

| Group | Members | Exclusive | Meaning |
|---|---|---|---|
| `residents` | `Spark-X2.5-4B-MLX-8bit` | False | Co-resides with other non-exclusive groups. |

## Macros (resolved into every entry's `cmd`)

| Macro | Path |
|---|---|
| `${bailing-server-bin}` | `~/inference/llama.cpp-bailing/build/bin/llama-server` |
| `${gguf-models}` | `~/inference/models/gguf` |
| `${llama-server-bin}` | `~/inference/bin/llama-server` |
| `${mlx-serve}` | `~/inference/bin/mlx-serve.sh` |
| `${tensorfold-bin}` | `~/inference/venv-tensorfold/bin/tensorfold` |

## Roster

| Entry | One-line | File |
|---|---|---|
| [`spark-2.5-4b`](models/spark-2.5-4b.md) | Spark X2.5 4B (Q8_0, Metal, F16 KV, 262k) | models/spark-2.5-4b.md |
| [`spark-2.6-4b`](models/spark-2.6-4b.md) | Spark X2.6 4B (Q8_0, Metal, F16 KV, 100k) | models/spark-2.6-4b.md |
| [`spark-x-pentester-2.6-4b`](models/spark-x-pentester-2.6-4b.md) | Spark X2.6 4B Pentester (Q8_0, Metal, F16 KV, 100k) | models/spark-x-pentester-2.6-4b.md |
| [`spark-2.5-4b-uncensored`](models/spark-2.5-4b-uncensored.md) | UNCENSORED Spark X2.5 4B (Q8_0, Metal, F16 KV, 262k) | models/spark-2.5-4b-uncensored.md |
| [`minicpm5-2b`](models/minicpm5-2b.md) | MiniCPM5-2B (Q8_0 GGUF, thinking + tools) | models/minicpm5-2b.md |
| [`Spark-X2.5-4B-MLX-8bit`](models/Spark-X2.5-4B-MLX-8bit.md) | Spark X2.5 4B (MLX 8-bit, native spark2_5) | models/Spark-X2.5-4B-MLX-8bit.md |
| [`Spark-X2.6-4B-MLX-8bit`](models/Spark-X2.6-4B-MLX-8bit.md) | Spark X2.6 4B (MLX 8-bit, native spark2_5) | models/Spark-X2.6-4B-MLX-8bit.md |
| [`MiniCPM5-2B-MLX-8bit`](models/MiniCPM5-2B-MLX-8bit.md) | MiniCPM5-2B (MLX 8-bit, llama arch) | models/MiniCPM5-2B-MLX-8bit.md |
| [`fm-system`](models/fm-system.md) | Apple Foundation Model (on-device, macOS 27 `fm serve`) | models/fm-system.md |

## Upstream llama.cpp PRs from this deployment

Authored against this card's stack (`author:anantshri`):

- `ggml-org/llama.cpp#30728` (open) - Metal GQA fa_vec decode.

## Measurement provenance

- llama.cpp rows are 2026-08-27 era; MLX A/B rows 2026-09-03 (taken on a dirty box - depressed, see troubleshooting).
- Clean-box re-measure 2026-09-30 showed one MLX row 4.8x off its dirty-box value; environment moved more than engine choice.

## Customizations

See [`customization/`](customization/) - one file per OS/hardware-level change.
