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
