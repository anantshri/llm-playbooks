# Public inference-server configuration details

Deployed llama-swap configs, one file per model, per card, plus OS-level customizations and a troubleshooting catalogue. No fluff.

## Structure

- `technology/` - one folder per inference technology (gateway + engines), how it is used here and what was learned.
- `cards/` - one folder per GPU / config type. Each has `README.md` (card, stack, roster, macros, upstream PRs), `models/` (one file per deployed model entry) and `customization/` (one file per OS/hardware-level change).
- `troubleshooting/` - one file per incident class: symptom, how it was found, root cause, recovery, prevention.

## Model file anatomy

Every file in `cards/*/models/` has the same sections:

1. **llama-swap entry** - the gateway fields set for that entry (ttl, health check, group, capabilities, pricing).
2. **Config block** - the verbatim YAML stanza as deployed.
3. **cmd parameters** - every flag in this entry's engine invocation, with the actual deployed value and what it does (KV policy, spec-decode settings, CPU-offload levers, sampling defaults).
4. **env parameters** - per-entry environment variables (GPU pinning, venv overrides).
5. **Gateway parameter reference** - the common gateway fields (identical meaning across all models; kept short).
6. **Measured / Notes** - benchmark rows with dates, incidents, and llama.cpp PRs authored from this deployment (`author:anantshri`).

## Sources and provenance

- Config blocks are the deployed files, md5-verified against the running servers; only host names and local usernames are rewritten to neutral markers (`the Arc B70 box`, `the RTX PRO 5000 box`, `the M4 Max laptop`, `the L40S box`, `~/`).
- Measured numbers are `llama-bench` rows taken 2026-08-27/28, re-measured 2026-08-30, with 09-02-dated rows measured 2026-09-02 (build b81c99b47). Method notes live in each card README.
- `tg@9k` rows are shallow decode; `tg@max` is a walk to ~98% of the configured context and is the only deep-decode number. Do not mix these with HTTP-gateway measurements (see troubleshooting).
- Numbers were measured on the specific hardware described. They are not portable claims about the model or the card class.

## Upstream llama.cpp PRs from these deployments

All authored by `anantshri` against Intel Arc Pro B70 (SYCL) unless noted:

| PR | Status | What it fixed / gained |
|---|---|---|
| [#28918](https://github.com/ggml-org/llama.cpp/pull/28918) | merged 2026-09-21 | Coalesce MKL-FA softmax loads: softmax kernel 2407->440 ms (5.5x); pp@126k +49% |
| [#28931](https://github.com/ggml-org/llama.cpp/pull/28931) | merged 2026-09-23 | MMVQ GLU fusion for mixed quants + rms_norm+scale + ssm_conv+silu fusions; tg +1.5-1.8% |
| [#29107](https://github.com/ggml-org/llama.cpp/pull/29107) | merged 2026-10-14 | IQ3 persistent reorder: IQ3 tg256@9k 12.30->21.59 tok/s |
| [#29171](https://github.com/ggml-org/llama.cpp/pull/29171) | merged 2026-10-07 | GLM-4.7 Flash MLA prefill via MKL-FA: pp@8192 3.01x, pp@64000 5.64x |
| [#30604](https://github.com/ggml-org/llama.cpp/pull/30604) | merged 2026-10-21 | IQ3 crash fix after the reorder (NaN/garbage with MTP) |
| [#30606](https://github.com/ggml-org/llama.cpp/pull/30606) | open | BMG D=256 single-query decode -> FA vector kernel (Spark 4B class) |
| [#30976](https://github.com/ggml-org/llama.cpp/pull/30976) | merged 2026-10-29 | Vectorized reordered dequant stores: Q4_K +14.7-18.7%, Q6_K +28.7%, IQ3_S +21.1%, MXFP4-MoE +57-62% (primed prefill) |
| [#32486](https://github.com/ggml-org/llama.cpp/pull/32486) | open | F16-KV + re-tiled Q8_0 dpas decode arms: served MTP TG @~125k +37-39% |
| [#30728](https://github.com/ggml-org/llama.cpp/pull/30728) | open | Metal GQA fa_vec decode (Apple M4 Max card) |
