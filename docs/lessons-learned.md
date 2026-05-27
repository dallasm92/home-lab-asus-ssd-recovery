# Lessons Learned

## What Worked

- The server was shut down cleanly before disconnecting the external SSD.
- The disk was repaired with the native Windows NTFS tool instead of relying on Linux `ntfsfix`.
- `chkdsk /f` was enough; `chkdsk /r` was not needed because the repair reported `0 KB in bad sectors`.
- The temporary `force` option was removed only after the Windows repair completed and ASUS services recovered.
- A reboot test proved that the system can mount the SSD without `force`.

## Why `force` Was Temporary

The Linux `ntfs3` `force` option can be useful as a short-term emergency recovery tool, but it should not be a steady-state mount option for a dirty NTFS volume. The correct durable fix is to repair the filesystem with Windows, then remove `force` and verify a clean Linux mount.

## Power-Loss Context

Apartment power outages were the likely trigger for the dirty NTFS state. For hosts that depend on external storage, clean shutdown behavior, backup freshness, and boot-safe mount options matter as much as the filesystem repair itself.

## What Stayed In Place

The following `/etc/fstab` hardening remains intentional:

- `nofail`
- `x-systemd.device-timeout=10`

Those options reduce the chance that a missing, slow, or failed external disk drops the server into emergency mode during boot.

## Operational Pattern

The repair followed a general safe-storage pattern:

1. Verify backup and restore state first.
2. Stop services and shut down the host cleanly.
3. Use the filesystem-native repair tool.
4. Eject media cleanly.
5. Restore the disk to the server.
6. Validate mounts and service health.
7. Remove temporary recovery settings.
8. Reboot and prove the final configuration works.

## Follow-up

The storage-risk item is closed. The next infrastructure project is the staged OPNsense rollout.
