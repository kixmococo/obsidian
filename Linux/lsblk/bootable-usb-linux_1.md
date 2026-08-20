# Making a Bootable USB from an ISO on Linux

Full process: locate the thumbdrive, wipe it, and write an ISO to it with `dd`.

> **Note on device names:** examples below use `/dev/sda` with partitions like
> `/dev/sda1`. On many systems `/dev/sda` is the main internal disk — always
> confirm with `lsblk` that `sda` really is your thumbdrive before running
> anything destructive.

## 1. Locate the thumbdrive

```bash
lsblk
```

Identify your drive by size. Note the **whole-disk** device name, not a
partition (`/dev/sda`, not `/dev/sda1`).

If you're not sure which one it is, run `lsblk` before and after plugging in the
drive and see what appears.

## 2. Unmount any existing partitions on it

```bash
sudo umount /dev/sda1
```

Repeat for any other partitions it shows (`sda2`, etc.).

## 3. Wipe it clean

```bash
sudo wipefs -a /dev/sda
```

## 4. Write the ISO directly with `dd`

For a bootable ISO, you don't format-then-copy-the-file. Skip formatting/partitioning
entirely and use `dd` to write the ISO's raw bytes directly onto the whole disk. The
ISO already contains its own filesystem and partition table, so `dd` overwrites the
drive with all of that in one shot:

```bash
sudo dd if=~/Downloads/yourfile.iso of=/dev/sda bs=4M status=progress oflag=sync
```

- `if=` — the ISO in your Downloads
- `of=` — the whole disk device, **not** a partition (no trailing number)
- `status=progress` — shows it's actually working (can take a few minutes)
- `oflag=sync` — makes sure it doesn't report "done" before data is actually flushed

## 5. Verify

```bash
sync
lsblk -f
```

You should see the ISO's filesystem/label show up on `/dev/sda1`.

## 6. Eject safely

```bash
sudo eject /dev/sda
```

---

## ⚠️ Warning

Step 4 **erases everything currently on the drive**, with no confirmation prompt.
If you typo the device letter, you can wipe the wrong disk. Double-check `lsblk`
immediately before running `dd`.

---

## Troubleshooting: "device is busy"

Usually means something still has it mounted or open.

**1. Find what's using it**
```bash
sudo lsof /dev/sda1
# or
sudo fuser -m /dev/sda1
```
Shows processes with open handles on the drive (a file manager preview pane, a
terminal `cd`'d into it, a background indexer, etc.)

**2. Kill whatever's holding it**
```bash
sudo fuser -km /dev/sda1
```
`-k` kills the processes using it, `-m` targets the mount point/device.

**3. Force unmount**
```bash
sudo umount -f /dev/sda1
```
If that still fails, try lazy unmount — detaches it now and cleans up once it's no
longer busy:
```bash
sudo umount -l /dev/sda1
```

**4. Check for auto-remount**
GNOME/KDE file managers often auto-remount a drive right after you unmount it,
especially if it's still showing in the sidebar. Close any file manager windows
showing the drive, and check:
```bash
mount | grep sda
```
If it's back, kill the file manager process or unmount via `udisksctl` instead —
it goes through the same daemon the desktop uses, so it won't just get remounted
behind your back:
```bash
udisksctl unmount -b /dev/sda1
```

**5. If `wipefs`/`mkfs` itself says busy**

Same root cause — something still has the device node open. Re-run the fuser/lsof
check against the whole disk, not just the partition:
```bash
sudo fuser -km /dev/sda
sudo wipefs -a /dev/sda
```

## Troubleshooting: "nothing mounted" but also "resource busy"

If `umount` says nothing is mounted, but then trying to mount/format says the
resource is busy, one of these is going on:

**1. It's mounted at a different path than you're checking**
```bash
mount | grep sda
findmnt /dev/sda1
cat /proc/mounts | grep sda
```
Whatever path shows up there is what you need to unmount — not the device name.

**2. You're checking the wrong device/partition**

Confirm exactly what's mounted where (whole disk vs. partition):
```bash
lsblk -f
```

**3. Stale/leftover mount reference (kernel vs. mtab mismatch)**
```bash
sudo umount -l /dev/sda1   # lazy unmount, even if 'nothing mounted' was reported
sudo umount -f /dev/sda1   # force
```
Both are safe to run even if it's already unmounted — they just no-op with an
error you can ignore.

**4. Something else has the raw device node open (not a filesystem mount)**

This is the more likely culprit for this exact symptom. A program can hold
`/dev/sda` open without it being "mounted" — e.g. GNOME Disks, GParted sitting
open, `udisksd`, or an antivirus/backup tool scanning it.
```bash
sudo lsof /dev/sda /dev/sda1
sudo fuser -v /dev/sda /dev/sda1
```
Whatever shows up, kill it:
```bash
sudo fuser -k /dev/sda /dev/sda1
```

**5. udisks auto-mounting it back instantly**

If unmount succeeds but something (like a file manager window still pointed at
the drive) triggers an auto-remount within milliseconds, umount reports success
but the next command sees it as busy again — the flip-flopping symptom. Close
all file managers, then check immediately:
```bash
udisksctl unmount -b /dev/sda1; mount | grep sda
```

## Alternative: just copying a file (not making it bootable)

If you want the ISO file to just sit on a normal formatted drive rather than
making the drive bootable, use the format-then-copy process instead:

```bash
sudo fdisk /dev/sda          # n (new partition) -> defaults -> w (write)
sudo mkfs.exfat /dev/sda1    # or mkfs.vfat -F 32 for FAT32
sudo mount /dev/sda1 ~/usb
cp ~/Downloads/yourfile.iso ~/usb/
sudo umount ~/usb
```
