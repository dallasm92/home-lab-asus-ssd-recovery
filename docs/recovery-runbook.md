# Recovery Runbook

This runbook captures the safe repair process used for the ASUS external NTFS SSD.

## 1. Pre-flight Health Check

From MacMint:

```bash
lab-status
```

Expected before physical work:

```text
Backup state: OK
Weekly restore state: OK
Quarterly restore state: OK
Failed units (macmint/asus/pi-core): none / none / none
Endpoint failures: 0
```

## 2. Clean ASUS Shutdown

```bash
ssh dallas@192.168.1.221 'sudo shutdown -h now'
```

Then confirm ASUS is offline:

```bash
ping -c 2 192.168.1.221
ssh -o BatchMode=yes -o ConnectTimeout=5 dallas@192.168.1.221 'hostname'
```

Expected:

- ping receives no packets
- SSH times out or refuses connection

Only remove the SSD after ASUS is fully powered off.

## 3. Windows Disk Identification

After attaching the SSD to `MAIN-PC`, open PowerShell as Administrator:

```powershell
Get-Volume -DriveLetter E | Format-List DriveLetter,FileSystemLabel,FileSystem,HealthStatus,OperationalStatus,SizeRemaining,Size
```

Observed before repair:

```text
DriveLetter       : E
FileSystem        : NTFS
HealthStatus      : Warning
OperationalStatus : Scan Needed
Size              : 2000188076032
```

Important safety rules:

- Do not format the disk.
- Do not initialize the disk.
- Do not repartition the disk.
- Avoid GUI repair prompts so the repair output is visible and auditable.

## 4. Windows Repair

```powershell
chkdsk E: /f
```

Observed repair summary:

```text
Windows has made corrections to the file system.
No further action is required.
0 KB in bad sectors.
```

## 5. Safe Eject And Reconnect

Use Windows Safely Remove Hardware / Eject before unplugging the SSD.

Then reconnect the SSD to ASUS and power ASUS back on.

## 6. Initial ASUS Validation

```bash
ping -c 2 192.168.1.221
ssh -o BatchMode=yes -o ConnectTimeout=8 dallas@192.168.1.221 'hostname && uptime && findmnt /mnt/externalssd && findmnt /DATA/Gallery/immich'
```

The disk initially mounted with the old temporary `force` option because `/etc/fstab` still contained it.

## 7. Service Recovery Check

```bash
/home/dallas/scripts/lab-service-recovery-check.sh --wait 30
```

The first pass can warn while Docker and app containers warm up. Repeat or use the reboot wrapper polling loop until green.

Successful result:

```text
RECOVERY CHECK PASSED
```

## 8. Remove Temporary `force`

Create a backup and remove only the `force` option:

```bash
ssh dallas@192.168.1.221 "sudo cp /etc/fstab /etc/fstab.pre-chkdsk-2026-05-27 && sudo sed -i 's/,force,/,/' /etc/fstab && grep -nE 'externalssd|immich' /etc/fstab"
```

Final `/etc/fstab` lines:

```text
UUID=080B995869D05BCD  /mnt/externalssd  ntfs3  defaults,noatime,uid=1000,gid=1000,fmask=0133,dmask=0022,nofail,x-systemd.device-timeout=10  0  0
/mnt/externalssd/immich  /DATA/Gallery/immich  none  bind,nofail,x-systemd.device-timeout=10  0  0
```

The `nofail,x-systemd.device-timeout=10` settings were retained to keep boot resilient if the external disk is absent or slow.

## 9. Reboot Test

```bash
bash /home/dallas/scripts/lab-reboot-asus-and-recover.sh
```

The wrapper:

- sends the reboot
- waits for the host to go unreachable
- waits for SSH recovery
- runs post-boot service checks
- polls until recovery passes

## 10. Final Checks

Confirm mount options:

```bash
ssh dallas@192.168.1.221 "findmnt /mnt/externalssd; findmnt /DATA/Gallery/immich"
```

Confirm there are no NTFS dirty/MFT warnings:

```bash
ssh dallas@192.168.1.221 "journalctl -b -k --no-pager | grep -iE 'ntfs3|sdb2|dirty|mft|externalssd' | tail -n 80"
```

Confirm the overall baseline:

```bash
lab-status
```
