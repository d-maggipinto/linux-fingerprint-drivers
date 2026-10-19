# Synaptics Tudor match-in-sensor (USB 06cb:00eb)

**Status: Working via the synaTudorMiS native driver (not upstream).**

Part of the Synaptics "Tudor" match-in-sensor family, see the
[family entry](../06cb:00ff/). Enrolment and matching happen on the sensor over
a TLS-paired channel. `06cb:00eb` is the variant in the Dynabook Satellite Pro
L50-G; firmware 10.1 (build 3031663, 2019-08), product id 65, security flags
0xa10f. Endpoints: bulk 0x01/0x81, interrupt 0x83.

## What was broken

Upstream libfprint's in-tree `synaptics` (BMKT) driver does not list the id,
and adding it does not help: the raw firmware version query succeeds, but the
first BMKT command is answered with status 0x0401 ("unsupported firmware
version" in the log). No in-tree driver can drive it.

## The fix

Route 1 from the family entry, the open `synatlsmoc` driver, with one line added
to its id table (and the autosuspend hwdb). The README of synaTudorMiS already
listed 00EB among the "pre v110" ids expected to work; it does.

`patches/0001-*.patch` is that id addition against
[vojtapl/libfprint](https://github.com/vojtapl/libfprint) commit `e97e405`
(2026-10-18). `patches/0002-*.patch` carries security fixes to the driver found
while reviewing it for this sensor (TLS handshake completion gate, a
use-after-free in `write_dft`, pairing data saved only after certificate
verification, bounded loops). Both are submitted upstream as vojtapl/libfprint#6
and #7; apply 0002 too, it is not sensor-specific.

Sensor-specific notes:

- fprintd needs the persistent-data patch from synaTudorMiS
  (`libfprint/fprintd-load-store-persistent-data-from-device.patch`); it applies
  to fprintd 1.94.5 and master. Without it every fprintd start re-pairs the
  sensor and all enrolments are lost. The same applies to libfprint's example
  programs: test with fprintd, not with `examples/enroll`.
- The driver logs "Received status 0x0401 on get_image_metrics, disabling them"
  on this sensor. Harmless.
- Pairing invalidates any Windows Hello pairing on the same sensor.

Build/install/PAM: see [docs/BUILD.md](../../docs/BUILD.md). On Debian-based
systems a `/usr/local` install of both libfprint and fprintd plus a systemd
drop-in (`ExecStart=/usr/local/libexec/fprintd`) leaves the distro packages
untouched; scripts for that, a hardened fprintd unit drop-in and the review
notes are in the write-up linked from `CREDITS`.

## Tested on

- Kali rolling (kernel 7.1.5, fprintd 1.94.5, GNOME/GDM Wayland), **Dynabook
  Satellite Pro L50-G-13F**: pairing, enrol, verify, lock-screen and GDM
  fingerprint unlock, pairing reused across fprintd restarts and reboots.
