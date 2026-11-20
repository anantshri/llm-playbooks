# Hardware watchdog + panic-on-softlockup (Arc Pro B70 box)

Two files, deployed from git, md5-verified. Both are required - see "why both" below.

## Deployable code

**`/etc/systemd/system.conf.d/99-watchdog.conf`**

```ini
# /etc/systemd/system.conf.d/99-watchdog.conf
[Manager]
RuntimeWatchdogSec=60
RebootWatchdogSec=10min

```

**`/etc/sysctl.d/99-crash-recovery.conf`**

```ini
# /etc/sysctl.d/99-crash-recovery.conf
kernel.softlockup_panic = 1
kernel.panic = 10

```

## Deploy

```sh
sudo install -m 0644 99-watchdog.conf /etc/systemd/system.conf.d/99-watchdog.conf
sudo systemctl daemon-reload          # re-reads system.conf AND re-arms the dog
sudo sysctl --system                # or reboot
```

## Verify

```sh
# verify systemd picked it up (want: RuntimeWatchdogUSec=1min)
systemctl show -p RuntimeWatchdogUSec --value

# verify the KERNEL side is being petted (systemd config alone can read back
# fine even when opening the device failed):
cat /sys/class/watchdog/watchdog0/state     # want: active
cat /sys/class/watchdog/watchdog0/timeout   # want: 60

```

## Why both files

- The watchdog alone cannot catch the 2026-11-18 wedge mode: one CPU spins in
  soft lockup while PID 1 stays schedulable, so systemd keeps petting the dog.
  `kernel.softlockup_panic=1` turns the "CPU#0 stuck for Ns" detector into a
  panic; `kernel.panic=10` reboots 10 s later.
- `kernel.hung_task_panic` deliberately NOT set: large model loads sit in
  legitimate >120 s D-state - false-reboot risk on a remote-only box.
- Trigger incident: `BUG: Bad page state` -> CPU0 soft lockup -> box dead ~9.5 h
  until manual power cycle (2026-11-18). 16th bad-page event in 10 weeks on
  non-ECC DDR5; memtest86+ owed at the next planned reboot window.
- Notes: `systemctl daemon-reload` is sufficient and verified (a `daemon-reexec`
  also works but is unnecessary). An earlier claim that reload does not re-arm
  the watchdog was wrong - checked empirically: changing the drop-in and
  reloading made systemd report the new value and log
  "Watchdog running with a hardware timeout of X".
