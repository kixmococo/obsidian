# Making a Bootable USB from an ISO on Linux

Full process: locate the thumbdrive, wipe it, and write an ISO to it with `dd`.

## 1. Locate the thumbdrive

```bash
lsblk
```

Look for your drive by size — thumbdrives usually show up as `/dev/sdb` or `/dev/sdc`
(not `/dev/sda`, which is almost always your main disk). Note the **whole-disk**
device name, not a partition (`/dev/sdb`, not `/dev/sdb1`).

If you're not sure which one it is, run `lsblk` before and after plugging in the
drive and see what appears.

## 2. Unmount any existing partitions on it

```bash
sudo umount /dev/sdb1
```

Repeat for any other partitions it shows (`sdb2`, etc.).

## 3. Wipe it clean

```bash
sudo wipefs -a /dev/sdb
```

## 4. Write the ISO directly with `dd`

For a bootable ISO, you don't format-then-copy-the-file. Skip formatting/partitioning
entirely and use `dd` to write the ISO's raw bytes directly onto the whole disk. The
ISO already contains its own filesystem and partition table, so `dd` overwrites the
drive with all of that in one shot:

```bash
sudo dd if=~/Downloads/yourfile.iso of=/dev/sdb bs=4M status=progress oflag=sync
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

You should see the ISO's filesystem/label show up on `/dev/sdb1`.

## 6. Eject safely

```bash
sudo eject /dev/sdb
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
sudo lsof /dev/sdb1
# or
sudo fuser -m /dev/sdb1
```
Shows processes with open handles on the drive (a file manager preview pane, a
terminal `cd`'d into it, a background indexer, etc.)

**2. Kill whatever's holding it**
```bash
sudo fuser -km /dev/sdb1
```
`-k` kills the processes using it, `-m` targets the mount point/device.

**3. Force unmount**
```bash
sudo umount -f /dev/sdb1
```
If that still fails, try lazy unmount — detaches it now and cleans up once it's no
longer busy:
```bash
sudo umount -l /dev/sdb1
```

**4. Check for auto-remount**
GNOME/KDE file managers often auto-remount a drive right after you unmount it,
especially if it's still showing in the sidebar. Close any file manager windows
showing the drive, and check:
```bash
mount | grep sdb
```
If it's back, kill the file manager process or unmount via `udisksctl` instead —
it goes through the same daemon the desktop uses, so it won't just get remounted
behind your back:
```bash
udisksctl unmount -b /dev/sdb1
```

**5. If `wipefs`/`mkfs` itself says busy**

Same root cause — something still has the device node open. Re-run the fuser/lsof
check against the whole disk, not just the partition:
```bash
sudo fuser -km /dev/sdb
sudo wipefs -a /dev/sdb
```

## Alternative: just copying a file (not making it bootable)

If you want the ISO file to just sit on a normal formatted drive rather than
making the drive bootable, use the format-then-copy process instead:

```bash
sudo fdisk /dev/sdb          # n (new partition) -> defaults -> w (write)
sudo mkfs.exfat /dev/sdb1    # or mkfs.vfat -F 32 for FAT32
sudo mount /dev/sdb1 ~/usb
cp ~/Downloads/yourfile.iso ~/usb/
sudo umount ~/usb
```
