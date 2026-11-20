# NVIDIA fan control (RTX PRO 5000 Blackwell)

Vendored from https://github.com/zmarty/nvidia-fan-control @ `3f0172b`
(pinned 2026-05-24; upstream re-checked 2026-10-13, unchanged). Vendored
because upstream has one commit and no releases - pin beats depend.

## Service - `/etc/systemd/system/nvidia-fan-control.service` (deployable, post-patch)

```ini
[Unit]
Description=NVIDIA Aggressive Fan Control for AI Workloads
After=nvidia-persistenced.service
Wants=nvidia-persistenced.service

[Service]
Type=simple
ExecStart=/opt/nvidia-fan-control/venv/bin/python3 /opt/nvidia-fan-control/nvidia-fan-control.py --mode aggressive --interval 1
ExecStop=/bin/kill -SIGTERM $MAINPID
Restart=on-failure
RestartSec=5

# Run as root (required for fan control)
User=root

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=nvidia-fan-control

[Install]
WantedBy=multi-user.target
```

## Diff vs upstream (two deliberate changes)

```diff
--- upstream/nvidia-fan-control.service
+++ this deployment
-ExecStart=/usr/bin/python3 /opt/nvidia-fan-control/nvidia-fan-control.py --mode quiet --interval 1
+ExecStart=/opt/nvidia-fan-control/venv/bin/python3 /opt/nvidia-fan-control/nvidia-fan-control.py --mode aggressive --interval 1
```

1. `--mode quiet` -> `--mode aggressive` - sustained-load temperatures on this card.
2. `/usr/bin/python3` -> venv python - the Debian `python3-pynvml` package drags
   in `libnvidia-ml1` newer than the kernel module from the NVIDIA `.run`
   installer; installing it once broke `nvidia-smi` with a driver/library
   mismatch (2026-05-24). The venv keeps the installer-provided
   `libnvidia-ml.so.1` intact.

## Log-throttling patch (daemon, 2026-09-27)

Upstream logs one line per GPU per second (~86k lines/day) - it actively hurt
the 2026-09-26 crash post-mortem (every journal read needed
`grep -v nvidia-fan-control`). Patch: log only when the target speed moves by
>= 3 points, plus a 60 s heartbeat carrying the **measured** fan readback
(liveness trail; a seized fan would otherwise never surface because the target
never changes). Deadband is load-bearing: the curve interpolates ~1%/C, so
sensor jitter would log every poll without it. `log.error` paths stay
unthrottled on purpose - a failing NVML read is a real fault and should be loud.
Fan behaviour itself unchanged; speeds still set every poll.

```diff
+HEARTBEAT_SECONDS = 60
+CHANGE_DEADBAND_PERCENT = 3
+
         self.fan_counts = []
+        self.last_logged_speed = {}
+        self.last_heartbeat = 0.0

     def update_fans(self):
+        now = time.monotonic()
+        heartbeat_due = (now - self.last_heartbeat) >= HEARTBEAT_SECONDS
+        if heartbeat_due:
+            self.last_heartbeat = now
@@
-                log.info(f"GPU {{gpu_idx}}: {{temp}}C -> {{target_speed}}% (was: {{current_speeds}})")
+                last = self.last_logged_speed.get(gpu_idx)
+                if last is None or abs(target_speed - last) >= CHANGE_DEADBAND_PERCENT:
+                    log.info(f"GPU {{gpu_idx}}: {{temp}}C -> {{target_speed}}% (was: {{current_speeds}})")
+                    self.last_logged_speed[gpu_idx] = target_speed
+                elif heartbeat_due:
+                    log.info(f"GPU {{gpu_idx}}: {{temp}}C -> {{target_speed}}% (steady, actual: {{current_speeds}})")
```

Update rule: re-apply BOTH patches on any upstream sync - a naive overwrite
silently restores 86k lines/day.
