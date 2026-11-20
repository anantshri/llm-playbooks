# Intel Arc Pro B70 (Battlemage / Xe2, 32 GB)

**Backend:** llama.cpp SYCL (Level Zero) + vLLM XPU; gateway llama-swap fork on :8080  
**VRAM:** 32 GB GDDR (~29.3 GiB usable after Mesa overhead)

## Global gateway settings (from the deployed config)

| Key | Value | Meaning |
|---|---|---|
| `healthCheckTimeout` | `600` | Max seconds to wait for a cold backend to become healthy before failing the request. |
| `captureBuffer` | `100` | Bytes of request/response captured per call for the stats UI. |
| `store` | `{'path': '/opt/inference/llama-swap.sqlite'}` | Path to the gateway's sqlite persistence DB. |

## Macros (resolved into every entry's `cmd`)

| Macro | Path |
|---|---|
| `${gguf-models}` | `/opt/inference/models/gguf` |
| `${hf-models}` | `/opt/inference/models/hf` |
| `${laguna-llama-server-bin}` | `/opt/inference/llama.cpp-laguna/build/bin/llama-server` |
| `${llama-server-bin}` | `/opt/inference/bin/llama-server-sycl-env` |
| `${llama-server-flashnext-bin}` | `/opt/inference/bin/llama-server-flashnext-env` |
| `${llama-server-obl-bin}` | `/opt/inference/bin/llama-server-obl-env` |
| `${llama-server-qwen36-bin}` | `/opt/inference/bin/llama-server-qwen36-env` |
| `${llama-server-qwen38-bin}` | `/opt/inference/bin/llama-server-qwen38-20261012-env` |
| `${llama-server-spark25-bin}` | `/opt/inference/bin/llama-server-spark25-env` |
| `${llama-server-spark25u-bin}` | `/opt/inference/bin/llama-server-spark25u-env` |
| `${llama-server-spark26-bin}` | `/opt/inference/bin/llama-server-spark26-env` |
| `${llama-server-sparkpt-bin}` | `/opt/inference/bin/llama-server-sparkpt-env` |
| `${llama-server-swift-bin}` | `/opt/inference/bin/llama-server-swift-env` |
| `${llama-server-sycl-bin}` | `/opt/inference/bin/llama-server-sycl-env` |
| `${llama-server-unc-bin}` | `/opt/inference/bin/llama-server-unc-env` |
| `${param2-llama-server-bin}` | `/opt/inference/bin/llama-server-param2-env` |
| `${vllm-bin}` | `/opt/inference/venv-vllm/bin/vllm` |

## Roster

| Entry | One-line | File |
|---|---|---|
| [`qwen3.8-27b`](models/qwen3.8-27b.md) | Qwen3.8 27B (UD-Q4_K_XL, SYCL, F16 KV, MTP, vision, 131k) | models/qwen3.8-27b.md |
| [`swift-qwen3.8-27b`](models/swift-qwen3.8-27b.md) | Swift Qwen3.8 27B (Q4_K_M, SYCL, F16 KV, MTP+ngram, vision, 131k) | models/swift-qwen3.8-27b.md |
| [`orcacyber-nano-1.0`](models/orcacyber-nano-1.0.md) | OrcaCyber Nano 1.0 (Mix-15.7G, SYCL, F16 KV, MTP+ngram, vision, 131k) | models/orcacyber-nano-1.0.md |
| [`qwen3.8-27b-vllm`](models/qwen3.8-27b-vllm.md) | Qwen3.8 27B (GPTQ INT4 + BF16 MTP4, vLLM/XPU) — quality-verified 09-04 | models/qwen3.8-27b-vllm.md |
| [`qwen3.8-27b-obliterated`](models/qwen3.8-27b-obliterated.md) | Qwen3.8 27B OBLITERATED (Q6_K, SYCL, q8_0 KV, MTP, abliterated, 131k) | models/qwen3.8-27b-obliterated.md |
| [`qwen3.8-27b-uncensored`](models/qwen3.8-27b-uncensored.md) | Qwen3.8 27B Uncensored — Q4_K_M, q8_0 KV, MTP+ngram, 229376, text-only (abliterated) | models/qwen3.8-27b-uncensored.md |
| [`ornith-1.5-35b-a3b`](models/ornith-1.5-35b-a3b.md) | Ornith 1.5 35B-A3B (Q8_0, SYCL, F16 KV, vision, n-cpu-moe 12, 131k) | models/ornith-1.5-35b-a3b.md |
| [`thomson-1.0-small`](models/thomson-1.0-small.md) | Thomson 1.0 Small (Q4_K_M, SYCL, F16 KV, in-VRAM, 131k) | models/thomson-1.0-small.md |
| [`qwen3.6-27b`](models/qwen3.6-27b.md) | Qwen3.6 27B (UD-Q4_K_XL, SYCL, F16 KV, MTP, 131k) | models/qwen3.6-27b.md |
| [`gemma4-26b`](models/gemma4-26b.md) | Gemma 4 26B-A4B (UD-Q6_K_XL, SYCL, F16 KV, 131k) | models/gemma4-26b.md |
| [`glm-4.7-flash`](models/glm-4.7-flash.md) | GLM-4.7 Flash (UD-Q4_K_XL, SYCL, F16 KV, 131k) | models/glm-4.7-flash.md |
| [`qwen3.8-flash-next-gsq-rco`](models/qwen3.8-flash-next-gsq-rco.md) | Qwen3.8 Flash Next — GSQ-RCO CODER (expert-PRUNED 256/512, SYCL, text-only, 128k) | models/qwen3.8-flash-next-gsq-rco.md |
| [`laguna-s-2.1`](models/laguna-s-2.1.md) | Laguna S 2.1 (118B-A8B MoE, Q4_K_M, SYCL, F16 KV, 65k, n-cpu-moe 46) | models/laguna-s-2.1.md |
| [`spark-2.5-4b`](models/spark-2.5-4b.md) | Spark X2.5 4B (BF16, SYCL, F16 KV, 131k) | models/spark-2.5-4b.md |
| [`spark-2.6-4b`](models/spark-2.6-4b.md) | Spark X2.6 4B (BF16, SYCL, F16 KV, 131k) | models/spark-2.6-4b.md |
| [`spark-x-pentester-2.6-4b`](models/spark-x-pentester-2.6-4b.md) | Spark X2.6 4B Pentester (Q8_0, SYCL, F16 KV, 131k) | models/spark-x-pentester-2.6-4b.md |
| [`spark-2.5-4b-uncensored`](models/spark-2.5-4b-uncensored.md) | UNCENSORED Spark X2.5 4B (BF16, SYCL, F16 KV, 131k) | models/spark-2.5-4b-uncensored.md |
| [`qwythos-tools`](models/qwythos-tools.md) | Qwythos-9B (Q8_0, SYCL, F16 KV, non-thinking — agent tools, 131k) | models/qwythos-tools.md |
| [`qwythos-think`](models/qwythos-think.md) | Qwythos-9B (Q8_0, SYCL, F16 KV, thinking — reasoning, no tools, 131k) | models/qwythos-think.md |
| [`btl4-compact`](models/btl4-compact.md) | BTL-4 Compact 35B-A3B (IQ2_XXS, SYCL, F16 KV, in-VRAM, 262k) | models/btl4-compact.md |
| [`param2-17b`](models/param2-17b.md) | Param-2 17B-A2.4B Thinking (Q4_K_M self-quant, SYCL, F16 KV, 32k) | models/param2-17b.md |
| [`param2-17b-q8`](models/param2-17b-q8.md) | Param-2 17B-A2.4B Thinking (Q8_0 self-quant, SYCL, F16 KV, 32k) | models/param2-17b-q8.md |

## Upstream llama.cpp PRs from this deployment

Authored against this card's stack (`author:anantshri`):

- `ggml-org/llama.cpp#28918` (merged 2026-09-21) - Coalesce MKL-FA softmax loads. Softmax kernel 2407->440 ms (5.5x); pp@64k +33%, pp@126k +49% (Qwen3.8-27B UD-Q4_K_XL).
- `ggml-org/llama.cpp#28931` (merged 2026-09-23) - MMVQ GLU fusion for mixed quants + rms_norm+scale + ssm_conv+silu fusions. ~330 fewer kernel launches/token; tg +1.5-1.8%.
- `ggml-org/llama.cpp#29107` (merged 2026-10-14) - IQ3 persistent reorder. IQ3 tg256@9k 12.30->21.59 tok/s, @64k 9.76->14.92, @126k 7.88->10.93.
- `ggml-org/llama.cpp#29171` (merged 2026-10-07) - GLM-4.7 Flash MLA prefill via MKL-FA (576/512, GQA-20). pp@8192 3.01x (+201%), pp@64000 5.64x (+464%). Decode unchanged.
- `ggml-org/llama.cpp#30604` (merged 2026-10-21) - IQ3 crash fix: fused-dequant GEMM misread reordered IQ3 bytes -> NaN/garbage with GSQ-RCO IQ3_S + MTP. 42/42 clean under 5-min concurrent load.
- `ggml-org/llama.cpp#30606` (open) - Route BMG D=256 single-query decode to the FA vector kernel (Spark 4B-class shapes).
- `ggml-org/llama.cpp#30976` (merged 2026-10-29) - Vectorized reordered-layout dequant stores. Primed prefill: Q4_K +14.7-18.7%, Q6_K +28.7%, IQ3_S +21.1%, MXFP4-MoE +57-62%; decode <=0.1%; PPL unchanged.
- `ggml-org/llama.cpp#32486` (open) - F16-KV + re-tiled Q8_0 decode arms for fattn-dpas. Env-gated (GGML_SYCL_FA_DPAS_F16 / _Q80_KG); served MTP TG @~125k +37-39% (measured 2026-11-14).

## Measurement provenance

- llama-bench rows measured 2026-08-27/28, re-measured 2026-08-30; rows labelled 09-02 measured 2026-09-02 (build b81c99b47).
- Card-specific semantics: llama-bench `tg` is depth-independent (near-empty prompt); only tg@max (bench-mx walk to ~98% of -c) is a true deep-decode number. MoE entries lose ~40% prefill in llama-bench vs the serving path (expert locality).

## Customizations

See [`customization/`](customization/) - one file per OS/hardware-level change.
