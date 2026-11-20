# Hardware watchdog (RTX PRO 5000 box)

**`/etc/systemd/system.conf.d/99-watchdog.conf`**

```ini
# /etc/systemd/system.conf.d/99-watchdog.conf
[Manager]
RuntimeWatchdogSec=60
RebootWatchdogSec=10min

```

## Deploy script (idempotent, kernel-verified)

```bash
#!/usr/bin/env bash
# Phase 12: arm the iTCO_wdt hardware watchdog so a wedged kernel self-recovers.
#
# Why: on 2026-09-26 10:07:57 the RTX PRO 5000 box hard-hung while completely idle and stayed
# dead ~25h until someone physically power-cycled it. The hang left NO kernel
# log, no panic and an empty /sys/fs/pstore, so there was nothing to act on after
# the fact either. /dev/watchdog0 existed the whole time but systemd's
# RuntimeWatchdogSec defaults to off, so nothing was petting it and nothing reset
# the board. With it armed, that outage becomes a ~1 minute reboot.
# See logs/the RTX PRO 5000 box/2026-09-27-crash-postmortem-ath12k-pci-dropout.md
#
# `daemon-reload` is sufficient and is what we use. (An earlier draft of this
# script claimed a reload does not pick up watchdog changes and used the more
# invasive `daemon-reexec`; that was WRONG. Verified empirically on this host
# 2026-09-27: editing the drop-in to 33s + `systemctl daemon-reload` made systemd
# report `33s` AND log `Watchdog running with a hardware timeout of 33s`, i.e. the
# device really is re-armed. systemd's MANAGER_RELOAD path calls
# set_manager_settings() -> manager_set_watchdog().)
#
# Idempotent: safe to re-run.
set -euo pipefail
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "${SCRIPT_DIR}/../env.sh"

REPO_DIR="${SCRIPT_DIR}/.."
DROPIN=/etc/systemd/system.conf.d/99-watchdog.conf
# Must match RuntimeWatchdogSec in systemd/99-watchdog.conf. Two forms: systemd
# normalises 60 to "1min" for `show`, the kernel reports plain seconds in sysfs.
WANT_RUNTIME="1min"
WANT_SECONDS="60"

# Hard failure, not a warning. A phase that "succeeds" without a watchdog device
# is worse than no phase at all: it makes the repo assert protection that is not
# there, and the failure mode is another silent multi-hour outage.
if [ ! -e /dev/watchdog0 ] && [ ! -e /dev/watchdog ]; then
    echo "[12] ERROR: no /dev/watchdog device." >&2
    echo "      iTCO_wdt is missing/blacklisted, or the board has no watchdog." >&2
    echo "      Check: lsmod | grep iTCO_wdt ; modprobe iTCO_wdt" >&2
    exit 1
fi

echo "[12] Installing ${DROPIN}"
sudo mkdir -p /etc/systemd/system.conf.d
sudo install -m 0644 "${REPO_DIR}/systemd/99-watchdog.conf" "${DROPIN}"

# A later-sorting drop-in would silently win over ours and the value check below
# would still pass if it happened to be nonzero -- so say so explicitly.
OTHERS="$(find /etc/systemd/system.conf.d -name '*.conf' ! -name '99-watchdog.conf' 2>/dev/null | sort || true)"
if [ -n "${OTHERS}" ]; then
    echo "[12] NOTE: other system.conf.d drop-ins present; a later-sorting one overrides us:"
    printf '        %s\n' ${OTHERS}
fi

echo "[12] systemctl daemon-reload (re-reads system.conf, re-arms the watchdog)"
sudo systemctl daemon-reload
sleep 2

# Check 1: systemd's configured value is EXACTLY what we asked for. Merely
# "nonzero" is not enough -- it would pass on a stale or overriding drop-in.
ARMED="$(systemctl show -p RuntimeWatchdogUSec --value)"
echo "[12] RuntimeWatchdogUSec=${ARMED} (want ${WANT_RUNTIME})"
if [ "${ARMED}" != "${WANT_RUNTIME}" ]; then
    echo "[12] ERROR: expected ${WANT_RUNTIME}, got '${ARMED}'." >&2
    echo "      Another drop-in is overriding ${DROPIN}, or the file did not apply." >&2
    exit 1
fi

# Check 2: the value above is systemd's CONFIG ONLY. systemd assigns it before it
# opens the device (watchdog_setup() sets watchdog_timeout then calls
# watchdog_open()), so it reads back fine even when the open failed -- missing
# module, or EBUSY because another watchdog daemon holds /dev/watchdog0. So verify
# the KERNEL side.
#
# Deliberately sysfs and not a journal grep: systemd only logs "Watchdog running
# with a hardware timeout of X" when the timeout actually CHANGES, so on a re-run
# with the value already correct there is no line to find. A journal check makes
# the phase fail on its second run -- which it did, and which broke the
# idempotency this phase promises. sysfs reports current state unconditionally.
WD_STATE_F=/sys/class/watchdog/watchdog0/state
WD_TIMEOUT_F=/sys/class/watchdog/watchdog0/timeout
if [ -r "${WD_STATE_F}" ]; then
    WD_STATE="$(cat "${WD_STATE_F}")"
    WD_TIMEOUT="$(cat "${WD_TIMEOUT_F}" 2>/dev/null || echo '?')"
    echo "[12] kernel: watchdog0 state=${WD_STATE} timeout=${WD_TIMEOUT}s identity=$(cat /sys/class/watchdog/watchdog0/identity 2>/dev/null || echo '?')"
    if [ "${WD_STATE}" != "active" ]; then
        echo "[12] ERROR: systemd reports ${ARMED} but the kernel says watchdog0 is" >&2
        echo "      '${WD_STATE}', not active. Nothing is petting the device, so a hang" >&2
        echo "      will NOT self-recover. Check: fuser -v /dev/watchdog0 ;" >&2
        echo "      systemctl status watchdog wd_keepalive" >&2
        exit 1
    fi
    if [ "${WD_TIMEOUT}" != "${WANT_SECONDS}" ]; then
        echo "[12] ERROR: kernel timeout is ${WD_TIMEOUT}s, expected ${WANT_SECONDS}s." >&2
        exit 1
    fi
else
    # No sysfs view (older kernel): fall back to proving PID 1 holds the device.
    echo "[12] NOTE: ${WD_STATE_F} unreadable; falling back to PID 1 fd check"
    if ! sudo ls -l /proc/1/fd/ 2>/dev/null | grep -q '/dev/watchdog'; then
        echo "[12] ERROR: PID 1 holds no fd on /dev/watchdog* — watchdog NOT live." >&2
        exit 1
    fi
fi

echo "[12] Armed: ${ARMED}, kernel-confirmed active. Done."

```

## Trigger incident

Box hard-hung while completely idle 2026-09-26 10:07:57, dead ~25 h. No kernel
log, no panic, empty pstore - nothing to act on after the fact. `/dev/watchdog0`
existed the whole time but nothing was petting it. With the dog armed that outage
becomes a ~1 minute reboot. Companion sysctl (softlockup panic + auto-reboot)
mirrors the Arc box setup.
