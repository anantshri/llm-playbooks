# NVIDIA RTX PRO 5000 (Blackwell, sm_120, 48 GB)

**Backend:** llama.cpp CUDA + vLLM 0.28-era CUDA; gateway llama-swap fork on :8080  
**VRAM:** 48 GB GDDR7

## Global gateway settings (from the deployed config)

| Key | Value | Meaning |
|---|---|---|
| `healthCheckTimeout` | `600` | Max seconds to wait for a cold backend to become healthy before failing the request. |
| `startPort` | `5800` | First internal proxy port; each loaded backend gets the next port. |
| `captureBuffer` | `100` | Bytes of request/response captured per call for the stats UI. |
| `logLevel` | `info` | Gateway log verbosity. |
| `store` | `{'path': '/opt/inference/llama-swap.sqlite'}` | Path to the gateway's sqlite persistence DB. |

## Swap groups

| Group | Members | Exclusive | Meaning |
|---|---|---|---|
| `gpu-exclusive` | `qwen3.6-27b`, `qwen3.6-27b-uncensored`, `qwen3.8-27b`, `qwen3.8-27b-uncensored`, `qwen3.8-27b-nvfp4`, `tf-qwen3.8-27b-nvfp4`, `swift-qwen3.8-27b`, `orcacyber-nano-1.0`, `laguna-xs-2.1`, `ornith-1.0-35b`, `ornith-1.5-35b-a3b`, `ornith-1.5-9b`, `btl4-35b`, `btl4-compact`, `qwen3.6-35b-a3b`, `qwen3.6-35b-a3b-q8`, `glm-4.7-flash`, `gemma4-26b`, `gemma4-31b-mtp`, `muse-glimmer-30b`, `gemma4-31b-200k-nospec`, `qwen3.8-flash-next`, `thomson-1.0-small`, `qwen3.8-27b-gsq`, `spark-2.5-4b`, `spark-2.6-4b`, `flux2-dev`, `flux2-klein-9b`, `flux1-dev-nsfw`, `qwen-image-2.1`, `qwen-image-2.1-edit`, `qwen3.8-flash-next-gsq-rco`, `qwen3.8-flash-next-gsq-rco-mtp`, `qwen3.8-flash-next-q2-0`, `qwen3.8-flash-next-q2-0-mtp`, `qwen3.8-flash-next-q2-0-abliterated`, `qwen3.8-flash-next-q2-0-abliterated-mtp`, `minimax-h3`, `minimax-h3-ref`, `cyber-frost-3.8`, `spark-x-pentester-2.6-4b`, `edge0-145b` | True | Loading a member evicts other groups. |
| `cores-coordinator` | `qwen3.6-27b-131k-q8kv`, `qwen3.6-27b-131k`, `qwen3.6-27b-200k-q8kv` | False | Co-resides with other non-exclusive groups. |
| `cores-worker` | `spark-x-pentester-2.6-4b-131k`, `spark-2.5-4b-uncensored-131k` | False | Co-resides with other non-exclusive groups. |

## Macros (resolved into every entry's `cmd`)

| Macro | Path |
|---|---|
| `${edge0-bin}` | `/opt/inference/bin/edge0-serve.sh` |
| `${edge0-models}` | `/opt/inference/models/edge0` |
| `${gguf-models}` | `/opt/inference/models/gguf` |
| `${h3-server-bin}` | `/opt/inference/venv-h3/bin/python /opt/inference/bin/h3-server.py` |
| `${hf-models}` | `/opt/inference/models/hf` |
| `${llama-server-bin}` | `/opt/inference/bin/llama-server` |
| `${llama-server-mtp-bin}` | `/opt/inference/bin/llama-server` |
| `${param2-llama-server-bin}` | `/opt/inference/llama.cpp-param2/build/bin/llama-server` |
| `${sd-checkpoints}` | `/opt/inference/models/sd/checkpoints` |
| `${sd-diffusion}` | `/opt/inference/models/sd/diffusion` |
| `${sd-encoders}` | `/opt/inference/models/sd/text-encoders` |
| `${sd-server-bin}` | `/opt/inference/bin/sd-server` |
| `${sd-vae}` | `/opt/inference/models/sd/vae` |
| `${tensorfold-serve}` | `/opt/inference/bin/tensorfold-serve.sh` |
| `${vllm-bin}` | `/opt/inference/bin/vllm-serve.sh` |

## Roster

| Entry | One-line | File |
|---|---|---|
| [`qwen3.6-27b`](models/qwen3.6-27b.md) | Qwen3.6 27B — Q8_0 weights, F16 KV, 262k, MTP draft | models/qwen3.6-27b.md |
| [`qwen3.6-27b-131k-q8kv`](models/qwen3.6-27b-131k-q8kv.md) | Qwen3.6 27B — Q8_0, KV q8_0, 131k, MTP — co-residency coordinator arm | models/qwen3.6-27b-131k-q8kv.md |
| [`qwen3.6-27b-200k-q8kv`](models/qwen3.6-27b-200k-q8kv.md) | Qwen3.6 27B — Q8_0, KV q8_0, 200k, MTP — co-residency coordinator arm (production) | models/qwen3.6-27b-200k-q8kv.md |
| [`qwen3.6-27b-131k`](models/qwen3.6-27b-131k.md) | Qwen3.6 27B — Q8_0, KV F16, 131k, MTP — co-residency fidelity arm | models/qwen3.6-27b-131k.md |
| [`qwen3.6-27b-uncensored`](models/qwen3.6-27b-uncensored.md) | Qwen3.6 27B Uncensored (DavidAU Heretic, Q8 + vision) | models/qwen3.6-27b-uncensored.md |
| [`qwen3.8-27b`](models/qwen3.8-27b.md) | Qwen3.8 27B (UD-Q8_K_XL GGUF, MTP, vision, 262k) | models/qwen3.8-27b.md |
| [`swift-qwen3.8-27b`](models/swift-qwen3.8-27b.md) | Swift Qwen3.8 27B (Q8_0 GGUF, MTP, vision, 262k) | models/swift-qwen3.8-27b.md |
| [`orcacyber-nano-1.0`](models/orcacyber-nano-1.0.md) | OrcaCyber Nano 1.0 (Mix-15.7G GGUF, MTP, vision, 262k) | models/orcacyber-nano-1.0.md |
| [`qwen3.8-27b-gsq`](models/qwen3.8-27b-gsq.md) | Qwen3.8 27B GSQ-RCO IQ3_S (MTP, vision, 262k) — quant A/B | models/qwen3.8-27b-gsq.md |
| [`qwen3.8-27b-nvfp4`](models/qwen3.8-27b-nvfp4.md) | Qwen3.8 27B (ModelOpt NVFP4, vLLM, vision, 262k) — quant A/B | models/qwen3.8-27b-nvfp4.md |
| [`tf-qwen3.8-27b-nvfp4`](models/tf-qwen3.8-27b-nvfp4.md) | Qwen3.8 27B (ModelOpt NVFP4, TensorFold) — engine A/B | models/tf-qwen3.8-27b-nvfp4.md |
| [`qwen3.8-27b-uncensored`](models/qwen3.8-27b-uncensored.md) | Qwen3.8 27B Uncensored (self-quant Q8_0, vision, 262k) | models/qwen3.8-27b-uncensored.md |
| [`laguna-xs-2.1`](models/laguna-xs-2.1.md) | Laguna XS 2.1 (33B-A3B MoE, Q8_0, 262K) | models/laguna-xs-2.1.md |
| [`qwen3.8-flash-next`](models/qwen3.8-flash-next.md) | Qwen3.8 Flash Next (125B-A6B sparse MoE, UD-Q4_K_XL, 262K) | models/qwen3.8-flash-next.md |
| [`qwen3.8-flash-next-gsq-rco`](models/qwen3.8-flash-next-gsq-rco.md) | Qwen3.8 Flash Next — GSQ-RCO CODER (expert-PRUNED 256/512, no draft, 256k) | models/qwen3.8-flash-next-gsq-rco.md |
| [`qwen3.8-flash-next-gsq-rco-mtp`](models/qwen3.8-flash-next-gsq-rco-mtp.md) | Qwen3.8 Flash Next — GSQ-RCO CODER (expert-PRUNED 256/512) + MTP n4 (256k) | models/qwen3.8-flash-next-gsq-rco-mtp.md |
| [`qwen3.8-flash-next-q2-0`](models/qwen3.8-flash-next-q2-0.md) | Qwen3.8 Flash Next — Q2_0 UNPRUNED (all 512 experts, no draft, 256k max ctx) | models/qwen3.8-flash-next-q2-0.md |
| [`qwen3.8-flash-next-q2-0-mtp`](models/qwen3.8-flash-next-q2-0-mtp.md) | Qwen3.8 Flash Next — Q2_0 UNPRUNED (all 512 experts) + MTP n5 (192k) | models/qwen3.8-flash-next-q2-0-mtp.md |
| [`qwen3.8-flash-next-q2-0-abliterated`](models/qwen3.8-flash-next-q2-0-abliterated.md) | Qwen3.8 Flash Next — Q2_0 UNPRUNED ABLITERATED (all 512 experts, no draft, 256k max ctx) | models/qwen3.8-flash-next-q2-0-abliterated.md |
| [`qwen3.8-flash-next-q2-0-abliterated-mtp`](models/qwen3.8-flash-next-q2-0-abliterated-mtp.md) | Qwen3.8 Flash Next — Q2_0 UNPRUNED ABLITERATED (all 512 experts) + MTP n5 (192k) | models/qwen3.8-flash-next-q2-0-abliterated-mtp.md |
| [`cyber-frost-3.8`](models/cyber-frost-3.8.md) | CYBER-FROST-3.8 (Blackfrost security fine-tune of Flash-Next, self-quant Q4_K_M, 262K) | models/cyber-frost-3.8.md |
| [`minimax-h3`](models/minimax-h3.md) | MiniMax-H3 (video+audio, t2va/fl2va, int8 + group offload) | models/minimax-h3.md |
| [`minimax-h3-ref`](models/minimax-h3-ref.md) | MiniMax-H3 (video+audio, ref2va omni-references, int8 + group offload) | models/minimax-h3-ref.md |
| [`ornith-1.0-35b`](models/ornith-1.0-35b.md) | Ornith 1.0 35B (Q8_0 GGUF, 200k ctx) | models/ornith-1.0-35b.md |
| [`btl4-35b`](models/btl4-35b.md) | BTL-4 35B (Q8_0 GGUF + vision, Ornith-1.0-35B fine-tune) | models/btl4-35b.md |
| [`btl4-compact`](models/btl4-compact.md) | BTL-4 Compact 35B-A3B (IQ2_XXS 2.3 bpw, author quant, 262k) | models/btl4-compact.md |
| [`ornith-1.5-35b-a3b`](models/ornith-1.5-35b-a3b.md) | Ornith 1.5 35B-A3B (Q8_0 GGUF + vision, 262k ctx) | models/ornith-1.5-35b-a3b.md |
| [`ornith-1.5-9b`](models/ornith-1.5-9b.md) | Ornith 1.5 9B (Q8_0 GGUF + vision, 262k ctx) | models/ornith-1.5-9b.md |
| [`qwen3.6-35b-a3b`](models/qwen3.6-35b-a3b.md) | Qwen3.6 35B-A3B — UD-Q4_K_M weights, F16 KV, 204800 | models/qwen3.6-35b-a3b.md |
| [`qwen3.6-35b-a3b-q8`](models/qwen3.6-35b-a3b-q8.md) | Qwen3.6 35B-A3B — Q8_0 weights, F16 KV, 204800 (Q8-weight variant of qwen3.6-35b-a3b) | models/qwen3.6-35b-a3b-q8.md |
| [`thomson-1.0-small`](models/thomson-1.0-small.md) | Thomson 1.0 Small (Q8_0 GGUF + vision, F16 KV, 131k) | models/thomson-1.0-small.md |
| [`glm-4.7-flash`](models/glm-4.7-flash.md) | GLM-4.7 Flash (UD-Q8_K_XL GGUF) | models/glm-4.7-flash.md |
| [`gemma4-26b`](models/gemma4-26b.md) | Gemma 4 26B A4B (UD-Q8_K_XL GGUF) | models/gemma4-26b.md |
| [`gemma4-31b-mtp`](models/gemma4-31b-mtp.md) | Gemma 4 31B-it — UD-Q8_K_XL, Q8_0 KV, 163840, MTP+ngram draft | models/gemma4-31b-mtp.md |
| [`muse-glimmer-30b`](models/muse-glimmer-30b.md) | Muse Glimmer 30B (Q8_0 GGUF, vision + DFlash, 262k) | models/muse-glimmer-30b.md |
| [`gemma4-31b-200k-nospec`](models/gemma4-31b-200k-nospec.md) | Gemma 4 31B-it — UD-Q8_K_XL, Q8_0 KV, 200000, NO spec decode (long-ctx sibling of gemma4-31b-mtp) | models/gemma4-31b-200k-nospec.md |
| [`spark-2.5-4b`](models/spark-2.5-4b.md) | Spark X2.5 4B (BF16, CUDA, F16 KV, 262k) | models/spark-2.5-4b.md |
| [`spark-2.6-4b`](models/spark-2.6-4b.md) | Spark X2.6 4B (BF16, CUDA, F16 KV, 262k) | models/spark-2.6-4b.md |
| [`spark-x-pentester-2.6-4b`](models/spark-x-pentester-2.6-4b.md) | Spark X2.6 4B Pentester (Q8_0, CUDA, F16 KV, 262k) | models/spark-x-pentester-2.6-4b.md |
| [`spark-x-pentester-2.6-4b-131k`](models/spark-x-pentester-2.6-4b-131k.md) | Spark X2.6 4B Pentester (Q8_0, F16 KV, 131k) — co-residency worker arm | models/spark-x-pentester-2.6-4b-131k.md |
| [`spark-2.5-4b-uncensored`](models/spark-2.5-4b-uncensored.md) | UNCENSORED Spark X2.5 4B (BF16, CUDA, F16 KV, 262k) | models/spark-2.5-4b-uncensored.md |
| [`spark-2.5-4b-uncensored-131k`](models/spark-2.5-4b-uncensored-131k.md) | UNCENSORED Spark X2.5 4B (Q8_0, CUDA, F16 KV, 131k) — co-residency worker arm | models/spark-2.5-4b-uncensored-131k.md |
| [`flux2-dev`](models/flux2-dev.md) | FLUX.2-dev (Q4_K_S GGUF) | models/flux2-dev.md |
| [`flux2-klein-9b`](models/flux2-klein-9b.md) | FLUX.2-klein-9B (GGUF) | models/flux2-klein-9b.md |
| [`flux1-dev-nsfw`](models/flux1-dev-nsfw.md) | FLUX.1-dev NSFW (Q4_K_S GGUF) | models/flux1-dev-nsfw.md |
| [`qwen-image-2.1`](models/qwen-image-2.1.md) | Qwen-Image-2.1 (bf16) | models/qwen-image-2.1.md |
| [`qwen-image-2.1-edit`](models/qwen-image-2.1-edit.md) | Qwen-Image-2.1 Edit (bf16) | models/qwen-image-2.1-edit.md |
| [`param2-17b`](models/param2-17b.md) | Param-2 17B-A2.4B Thinking (Q8_0 self-quant, CUDA, F16 KV, 32k) | models/param2-17b.md |
| [`param2-17b-f16`](models/param2-17b-f16.md) | Param-2 17B-A2.4B Thinking (F16, CUDA, F16 KV, 32k) | models/param2-17b-f16.md |
| [`edge0-145b`](models/edge0-145b.md) | Edge0 145B (NVFP4 native, edge0-strata engine, 131k) | models/edge0-145b.md |

## Upstream llama.cpp PRs from this deployment

Authored against this card's stack (`author:anantshri`):

- `ggml-org/llama.cpp#30728` (open) - Metal GQA fa_vec decode (co-authored from this fleet's Metal A/B work).

## Measurement provenance

- llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47).
- tg@9k is shallow decode; tg@max is the deep-decode walk (~98% of -c). Do not mix with gateway measurements (see troubleshooting).

## Customizations

See [`customization/`](customization/) - one file per OS/hardware-level change.
