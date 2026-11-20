# Gateway systemd unit (deployable)

```ini
[Unit]
Description=llama-swap proxy (llama-server swap on the GPU box)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=user
Group=user
WorkingDirectory=/opt/inference

ExecStart=/opt/inference/bin/llama-swap-anantshri \
    --config /opt/inference/config/llama-swap.yaml \
    --listen 0.0.0.0:8080

Restart=on-failure
RestartSec=5

# Inherit GPU + caches
Environment=HF_HOME=/opt/inference/.cache/huggingface
Environment=PATH=/opt/inference/bin:/opt/inference/venv-vllm/bin:/usr/local/bin:/usr/bin:/bin

# Backend processes are heavy - raise file handle limit
LimitNOFILE=65536

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

- The vLLM venv on `PATH` exists on the CUDA box; the Arc box unit is identical
  minus that entry.
- `LimitNOFILE=65536`: backends open many handles; the default limit bit us early.
- Config and unit mirrored in git; deployed copies md5-verified.
