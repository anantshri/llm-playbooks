# fm-system
**Card:** Apple M4 Max (128 GB unified memory)  |  **Engine:** Apple Foundation Model (fm serve)

**Display name:** Apple Foundation Model (on-device, macOS 27 `fm serve`)

## Model card summary (from the deployed config)

- Type: Apple on-device Foundation Model | no thinking | NO tools (accepted, ignored)
- Parameter: undisclosed (Apple documents an ~3B on-device model)
- Source: macOS 27.0 (26A428) system model via /usr/bin/fm — no HF repo, nothing downloaded
- Engine: Apple Neural Engine / Metal (OS-managed, opaque to llama-swap)
- Inference: Apple Foundation Models CLI — `fm serve`
- Quant: Apple on-device (not user-selectable)
- KV Cache: not exposed
- Context window: 8192 (measured 2026-09-25 — 7692-token prompt OK, ~9.6k refused)
- Chat template: Apple-internal (fm maps OpenAI messages/roles itself)
- Sampling: not exposed over HTTP — temperature/top_p accepted and ignored
- Measured: 2026-09-25 scratch-port probe — text and image_url input both OK, usage counts real, no tool_calls ever
- Notes: on-demand ttl 600; no unified-memory floor of its own (weights live in the OS daemon). Log: logs/the M4 Max laptop/2026-09-25-llama-swap-fc1b1da-apple-fm.md

## llama-swap entry

- **useModelName:** `system`
- **ttl:** `600`
- **checkEndpoint:** `/health`
- **capabilities:** `{'in': ['text', 'image'], 'out': ['text'], 'context': 8192, 'disableAuto': True}`
- **filters:** `{'setParams': {'stream?': False, 'stream_options?': {'include_usage': True}}}`

## Config block

```yaml
fm-system:
  name: Apple Foundation Model (on-device, macOS 27 `fm serve`)
  description: 'Type: Apple on-device Foundation Model | no thinking | NO tools (accepted, ignored)

    Parameter: undisclosed (Apple documents an ~3B on-device model)

    Source: macOS 27.0 (26A428) system model via /usr/bin/fm — no HF repo, nothing downloaded

    Engine: Apple Neural Engine / Metal (OS-managed, opaque to llama-swap)

    Inference: Apple Foundation Models CLI — `fm serve`

    Quant: Apple on-device (not user-selectable)

    KV Cache: not exposed

    Context window: 8192 (measured 2026-09-25 — 7692-token prompt OK, ~9.6k refused)

    Chat template: Apple-internal (fm maps OpenAI messages/roles itself)

    Sampling: not exposed over HTTP — temperature/top_p accepted and ignored

    Measured: 2026-09-25 scratch-port probe — text and image_url input both OK, usage counts real, no
    tool_calls ever

    Notes: on-demand ttl 600; no unified-memory floor of its own (weights live in the OS daemon). Log:
    logs/the M4 Max laptop/2026-09-25-llama-swap-fc1b1da-apple-fm.md

    '
  cmd: '/usr/bin/fm serve --host 127.0.0.1 --port ${PORT}

    '
  checkEndpoint: /health
  useModelName: system
  filters:
    setParams:
      stream?: false
      stream_options?:
        include_usage: true
  ttl: 600
  capabilities:
    in:
    - text
    - image
    out:
    - text
    context: 8192
    disableAuto: true
```

## cmd parameters (this entry's engine invocation)

argv[0]: `/usr/bin/fm`

| Flag | Value here | What it does |
|---|---|---|
| `--host` | `127.0.0.1` | Bind address. 127.0.0.1 - only the gateway reaches the backend. |
| `--port` | `${PORT}` | Bind port. ${PORT} is assigned by the gateway from its startPort block. |

## Gateway parameter reference (common to all entries)

| Parameter | Meaning |
|---|---|
| `name` | Display name shown by the gateway UI and logs. |
| `description` | Fleet-format structured description: model type, HF source, engine, quant + size, KV cache dtype/size, context window, chat template. |
| `checkEndpoint` | Path polled until healthy before traffic is routed to this backend. |
| `useModelName` | Model id forwarded to the backend when it must differ from the entry id. |
| `filters` | Gateway request filters applied before proxying to this backend. |
| `ttl` | Seconds of idle before the gateway stops this backend and frees its VRAM/RAM. |
| `capabilities` | Gateway metadata for what this entry supports (e.g. reasoning). Requests using an unsupported feature are rejected up-front instead of mis-served. |

## Notes, incidents, upstream references

- Apple on-device Foundation Model via macOS fm serve - no model files, no VRAM cost. 8192-token context (smallest on this card) and NO tool calling: tools[] is accepted and silently ignored. Text + image input.
