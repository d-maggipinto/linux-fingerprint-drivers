# Synaptics Tudor match-in-sensor (USB 06cb:00fd)

**Status: Working via the synaTudor relink (not upstream).**

Part of the Synaptics "Tudor" match-in-sensor family — see the
[family entry](../06cb:00ff/). Enrollment and matching happen on the sensor over
a TLS-paired channel, which is why libfprint's in-tree `synaptics` (BMKT) driver
cannot drive it. `06cb:00fd` is the variant common in Lenovo IdeaPad / ThinkBook
machines; it ships a **newer driver generation (DLLs v139)** than the
HP-oriented package covered in the `00ff` entry.

## What was broken

No upstream libfprint driver; the sensor is claimed by no in-tree driver.

## The fix

Use the synaTudor relink (Route 2 in the [family entry](../06cb:00ff/)): it
runtime-links Synaptics' own Windows driver and exposes it as a libfprint TOD
module. For `06cb:00fd` specifically:

- use the **IdeaPad consumer driver package**
  (`download.lenovo.com/consumer/mobiles/xalc0l0fqu9mwjc0.exe`, which contains
  the `00fd` id) and bump the expected DLLs to **v139**
  (`synaFpAdapter139.dll`, `synaWudfBioUsb139.dll`);
- add `06cb:00fd` to the TOD id table and the udev rule.

The relinker also required a few sensor-independent fixes: the WDF
interface-property store (index 50–52), claiming the USB interface up-front,
`GetModuleHandleW(NULL)` handling, USER32/power/WTS/registry stubs, and larger
IPC record/message size caps. All are in the PRs listed in `CREDITS`.

Build/install/PAM: see [docs/BUILD.md](../../docs/BUILD.md). One extra detail for
relink users: the driver needs ~4 s to load its DLLs and pair, so a boot-time
warm-up is needed for the login/lock screen to find it ready (see `contrib/` in
the fork below).

## Tested on

- Ubuntu 26.04 (fprintd 1.94.5, libfprint 1.95.1+tod1), Regolith/sway + GDM,
  **Lenovo IdeaPad 3 14ITL6** — enroll, boot/login, lock/suspend and sudo all
  work.
