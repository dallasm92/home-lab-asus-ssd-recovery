# Validation Evidence

## Pre-maintenance Baseline

Before shutting ASUS down:

```text
Backup state: OK (OK)
Weekly restore state: OK (OK)
Quarterly restore state: OK (OK)
Failed units (macmint/asus/pi-core): none / none / none
Endpoint failures: 0
```

## Windows Volume State Before Repair

```text
DriveLetter       : E
FileSystem        : NTFS
HealthStatus      : Warning
OperationalStatus : Scan Needed
SizeRemaining     : 1971270840320
Size              : 2000188076032
```

## CHKDSK Repair Result

Important `chkdsk E: /f` lines:

```text
Deleted corrupt attribute list entry
Deleting corrupt attribute record
2 unindexed files recovered to lost and found
Correcting errors in the master file table's (MFT) BITMAP attribute.
CHKDSK discovered free space marked as allocated in the volume bitmap.
Windows has made corrections to the file system.
No further action is required.
0 KB in bad sectors.
```

## Final Mount State

After removing `force` and rebooting ASUS:

```text
TARGET           SOURCE    FSTYPE OPTIONS
/mnt/externalssd /dev/sdb2 ntfs3  rw,noatime,uid=1000,gid=1000,dmask=0022,fmask=0133,iocharset=utf8

TARGET               SOURCE             FSTYPE OPTIONS
/DATA/Gallery/immich /dev/sdb2[/immich] ntfs3  rw,noatime,uid=1000,gid=1000,dmask=0022,fmask=0133,iocharset=utf8
```

The absence of `force` in both mount option sets is the key Linux-side success condition.

## Final ASUS Recovery Check

```text
NO_REBOOT_REQUIRED
OK: reboot-required cleared
OK: no failed systemd units
OK: docker active
```

Container health returned green for:

- `homepage`
- `uptimekuma`
- `big-bear-syncthing`
- `duplicati`
- `portainer`
- `immich-server`
- `immich-machine-learning`
- `immich-redis`
- `immich-postgres`

Endpoint checks returned expected HTTP status codes for:

- `homepage.lan`
- `kuma.lan`
- `casaos.lan`
- `npm.lan`
- `portainer.lan`
- `duplicati.lan`
- `immich.lan`
- `syncthing.lan`

Final result:

```text
RECOVERY CHECK PASSED
```

## Final Lab Health

```text
Backup state: OK (OK)
Weekly restore state: OK (OK)
Quarterly restore state: OK (OK)
Failed units (macmint/asus/pi-core): none / none / none
Endpoint failures: 0
```
