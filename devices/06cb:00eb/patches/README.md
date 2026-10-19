Patches against https://github.com/vojtapl/libfprint, branch `main`,
base commit e97e405 (2026-10-18, "Merge pull request #5 ... fix-get-version-stall").

- 0001-synatlsmoc-add-Synaptics-06cb-00eb.patch — id table + autosuspend hwdb
  (upstream PR vojtapl/libfprint#6)
- 0002-synatlsmoc-security-fixes.patch — TLS handshake completion gate,
  write_dft use-after-free, pairing data saved after verification, bounded
  loops, minor hardening (upstream PR vojtapl/libfprint#7). Not sensor-specific.

Apply with `git am` in that order. fprintd additionally needs
`libfprint/fprintd-load-store-persistent-data-from-device.patch` from
https://github.com/vojtapl/synaTudorMiS (applies to fprintd 1.94.5 and master).
