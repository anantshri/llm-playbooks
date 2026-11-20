# Level Zero loader pin (deployable)

Debian does not ship a current Intel compute stack; everything comes from Intel
GitHub releases, with **loader and driver from matching releases**.

```bash
# from the provisioning phase (versions pinned in env):
export LEVEL_ZERO_VER=1.32.0
export COMPUTE_RUNTIME_VER=26.31.39395.13

# Level Zero loader + SDK headers, from the SAME upstream release:
fetch_release_assets oneapi-src/level-zero "v${LEVEL_ZERO_VER}" \
    '^libze1_.*amd64\.deb$' \
    '^libze-dev_.*amd64\.deb$'

# compute-runtime (OpenCL ICD + Level Zero GPU driver + ocloc + gmmlib):
fetch_release_assets intel/compute-runtime "${COMPUTE_RUNTIME_VER}" \
    '^libigdgmm12_.*amd64\.deb$' \
    '^intel-opencl-icd_.*amd64\.deb$' \
    '^libze-intel-gpu1_.*amd64\.deb$' \
    '^intel-ocloc_.*amd64\.deb$'
```

Why it matters (three measured/observed facts):

1. Without the loader, `sycl-ls` shows only opencl backends, large USM
   allocations fail on Battlemage, and every SYCL model load dies with
   `UR_RESULT_ERROR_OUT_OF_RESOURCES` - that misdiagnosis cost most of a day
   (2026-08-18).
2. `libze-dev` matters beyond headers: llama.cpp's optional
   `GGML_SYCL_SUPPORT_LEVEL_ZERO_API` only compiles in when `ze_api.h` is
   present; the CMake silently drops the define otherwise.
3. Loader version changes the GPU allocation path: pinning the loader measured
   model load 17.0 s -> 12.9 s (2026-09-02). Re-A/B load time before unpinning.
