# Wiping and Formatting a USB Drive to FAT32 on Linux

Full process: locate the drive, force it free if it's "busy," wipe it, partition it, and format to FAT32.

> **Note on device names:** examples use `/dev/sda` with partitions like `/dev/sda1`.
> On many systems `/dev/sda` is the main internal disk — always confirm with `lsblk`
> that `sda` really is your thumbdrive before running anything destructive.

## 1. Locate the drive

```bash
lsblk
```

Identify your drive by size. Note the **whole-disk** device name, not a partition
(`/dev/sda`, not `/dev/sda1`). If unsure, run `lsblk` before and after plugging in
the drive and compare.

## 2. Force it free if it says "busy"

If `umount` says nothing is mounted but formatting still says the resource is
busy, something still has the raw device node open (GNOME Disks, GParted,
`udisksd`, a backup/AV scan, etc.) — not necessarily a filesystem mount.

**Kill everything touching it:**
```bash
sudo fuser -k /dev/sda /dev/sda1 /dev/sda2 2>/dev/null
sudo fuser -k -m /dev/sda1 2>/dev/null
sudo killall gnome-disks udisksd gvfsd-trash gvfs-udisks2-volume-monitor 2>/dev/null
```

**Force + lazy unmount everything on the disk:**
```bash
for p in /dev/sda*; do sudo umount -f -l "$p" 2>/dev/null; done
```

**Release it at the block-device level (bypasses desktop mount tracking):**
```bash
sudo udisksctl unmount -b /dev/sda1 --force 2>/dev/null
sudo udisksctl power-off -b /dev/sda 2>/dev/null
```

**Replug the drive** — unplug, wait a few seconds, plug back in. Confirm with
`lsblk` that it's back as `/dev/sda`.

If it goes busy again the instant you plug it in, something is auto-mounting it
immediately. Either work fast (within a couple seconds of plugging in), or
temporarily disable auto-mount:
```bash
gsettings set org.gnome.desktop.media-handling automount false
# ...do the wipe/format...
gsettings set org.gnome.desktop.media-handling automount true   # re-enable after
```

To see exactly what's holding the device at any point:
```bash
sudo fuser -v /dev/sda /dev/sda1
sudo lsof /dev/sda /dev/sda1
```

## 3. Wipe existing filesystem signatures

```bash
sudo wipefs -a /dev/sda
```

## 4. Create a new partition table and partition

```bash
sudo fdisk /dev/sda
```

Inside `fdisk`, in order:
| Key | Action |
|---|---|
| `o` | new empty DOS (MBR) partition table |
| `n` | new partition |
| *(Enter)* | accept default: primary |
| *(Enter)* | accept default: partition number 1 |
| *(Enter)* | accept default: first sector |
| *(Enter)* | accept default: last sector (uses full disk) |
| `t` | change partition type |
| `b` | set type to `W95 FAT32` |
| `w` | write changes and exit |

## 5. Format the partition as FAT32

```bash
sudo mkfs.vfat -F 32 /dev/sda1
```

⚠️ Target the **partition** (`/dev/sda1`), not the whole disk and not `/dev/null` —
double-check the device name before hitting enter.

## 6. Verify

```bash
lsblk -f
```

You should see `/dev/sda1` listed with filesystem type `vfat`.

## 7. Mount it (if it doesn't auto-mount)

```bash
mkdir -p ~/usb
sudo mount /dev/sda1 ~/usb
```

Unmount when done:
```bash
sudo umount ~/usb
```

---

## ⚠️ Warning

Wiping and partitioning **erases everything currently on the drive**, with no
confirmation prompt. If you get the device letter wrong, you can wipe the wrong
disk. Always confirm with `lsblk` immediately before running any destructive
command.

---

## Appendix: writing a bootable ISO instead of FAT32

If the goal is a bootable USB from a `.iso` (not a general-purpose FAT32 drive),
skip formatting/partitioning — `dd` writes the ISO's own filesystem and partition
table directly onto the whole disk:

```bash
sudo dd if=~/Downloads/yourfile.iso of=/dev/sda bs=4M status=progress oflag=sync
sync
lsblk -f          # confirm the ISO's label shows up on /dev/sda1
sudo eject /dev/sda
```
