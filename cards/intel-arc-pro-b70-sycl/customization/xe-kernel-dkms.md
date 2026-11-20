# xe driver, fan-patched, via DKMS

- The distro `xe` driver lacks the hwmon/pwm surface the fan curve needs; a fan-only patched module is built per kernel via DKMS (`xe-arc-fanonly`).
- 2026-08-27 incident + hardening: after a kernel upgrade (7.1.3 -> 7.1.8 backports) DKMS autoinstall failed because the build script resolved the source tarball **inside** `/usr/src/linux-source-7.1/`, a manually-created, non-dpkg-owned directory. Fix: source-version guard in the build script + stale-tarball detection; `pwm1_auto_point` symbol check verifies the patched symbols exist in the built module.
- Old kernels are kept installed as boot fallbacks; nothing purged.
