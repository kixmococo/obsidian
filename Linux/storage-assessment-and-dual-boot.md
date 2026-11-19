# Storage Assessment & Ubuntu + Kali Dual Boot

Goal: understand what is on the disk, confirm where each OS lives, check for any leftover Windows, then add **Kali** next to **Ubuntu** without breaking Ubuntu. Both should boot from the firmware (F12/F10/Esc) boot menu every time.

---

## 0. My machine (snapshot taken 2026-11-19)

```
nvme0n1     476.9G                         (single NVMe SSD)
├─nvme0n1p1     1G  vfat   /boot/efi       EFI System Partition (ESP)
└─nvme0n1p2 475.9G  ext4   /               Ubuntu root  (46G used, 398G free)
Firmware mode: UEFI
```

What this tells me:
- **Ubuntu lives on `nvme0n1p2`**, and its bootloader lives on the ESP at `nvme0n1p1` (`/boot/efi/EFI/ubuntu/`).
- **No Windows on this disk.** There's no NTFS partition, no "Microsoft reserved" partition and no Windows recovery partition. The original Windows install was wiped when Ubuntu took the whole disk. (Section 3 explains how to check this properly.)
- **There's no unallocated space yet.** Ubuntu's partition fills the disk, so it has to be **shrunk** before Kali can go in (Section 5).

> [!note] "Memory" vs "storage"
> RAM (memory) is temporary working space. Storage (disk/SSD) is where the OSes live. Section 1 has commands for both, but dual booting only involves storage.

---

## 1. Quick assessment commands

### Disks & partitions: the big picture
| Command | What it shows |
|---|---|
| `lsblk` | Tree of disks → partitions → mount points |
| `lsblk -e7 -o NAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINTS,PARTTYPENAME` | Same, but hides snap loop devices (`-e7`) and adds filesystem, UUID and partition type |
| `lsblk -f` | Filesystems, labels, UUIDs and free space per partition |
| `sudo fdisk -l` | Partition tables, sector sizes and disk model |
| `sudo parted -l` | Partition table type (gpt/msdos) and partition flags (`boot, esp`) |
| `sudo blkid` | UUIDs and PARTUUIDs (what fstab and GRUB use) |
| `findmnt` / `findmnt /` | What is mounted where, including mount options |
| `cat /etc/fstab` | What mounts at boot |
| `sudo lshw -short -class disk -class volume` | Hardware view of drives |
| `inxi -Dxx` (`sudo apt install inxi`) | Drive model, size, temperature and usage summary |
| `gnome-disks` / `gparted` | GUI versions of the above |

### Space usage: what's taking up room
| Command | What it shows |
|---|---|
| `df -hT -x squashfs -x tmpfs` | Used and free space per real filesystem |
| `sudo du -xh --max-depth=1 / \| sort -h` | Biggest top-level directories (`-x` stays on one filesystem) |
| `du -sh ~/* ~/.??* 2>/dev/null \| sort -h` | Biggest items in your home folder |
| `ncdu -x /` (`sudo apt install ncdu`) | Interactive drill-down. **Best tool for this job.** |
| `snap list --all` | Old snap revisions (the loop devices) |
| `journalctl --disk-usage` | Space used by system logs |

### Drive health
| Command | What it shows |
|---|---|
| `sudo smartctl -a /dev/nvme0n1` (`smartmontools`) | SMART health, wear %, error log |
| `sudo nvme smart-log /dev/nvme0n1` (`nvme-cli`) | NVMe health: `percentage_used`, `media_errors` |

### RAM / swap
| Command | What it shows |
|---|---|
| `free -h` | RAM and swap usage |
| `swapon --show` | Swap file or partition in use |
| `sudo dmidecode -t memory \| grep -E "Size\|Speed\|Type:"` | Installed RAM sticks, speed and type |
| `cat /proc/meminfo` | Detailed memory counters |

### Boot setup
| Command | What it shows |
|---|---|
| `[ -d /sys/firmware/efi ] && echo UEFI \|\| echo Legacy` | Boot mode (should be UEFI) |
| `sudo efibootmgr -v` | **Firmware boot entries.** This list is what the F12 boot menu shows. |
| `sudo ls -R /boot/efi/EFI` | Bootloaders stored on the ESP (one folder per OS) |
| `mokutil --sb-state` | Whether Secure Boot is on |
| `ls /boot` | Installed kernels and initrds |

---

## 2. Finding where the current OS lives

```bash
findmnt -no SOURCE /          # partition holding root (/)  → /dev/nvme0n1p2
findmnt -no SOURCE /boot/efi  # the ESP                     → /dev/nvme0n1p1
cat /etc/os-release           # which distro is running
sudo efibootmgr -v            # firmware entry "ubuntu" → \EFI\ubuntu\shimx64.efi
```

If `/boot` or `/home` were separate partitions they would appear in `findmnt` and `/etc/fstab`. On this machine they are not.

---

## 3. Is Windows (or its leftovers) still on the machine?

Run these. If none of them shows a match, Windows is gone.

```bash
lsblk -f | grep -i ntfs                       # Windows partitions are NTFS
sudo parted -l | grep -iE "msftres|msftdata"   # Microsoft Reserved / Basic Data partitions
sudo fdisk -l | grep -iE "Microsoft|Windows"   # "Microsoft reserved", "Windows recovery environment"
sudo ls /boot/efi/EFI                          # a "Microsoft" folder = Windows bootloader is/was there
sudo efibootmgr -v | grep -i windows           # stale "Windows Boot Manager" firmware entry
sudo os-prober                                 # scans every disk for other OSes
```

Where Windows normally sits on a factory laptop:
| Partition | Size | Type |
|---|---|---|
| ESP | 100–300 MB | FAT32, holds `EFI/Microsoft/Boot/bootmgfw.efi` |
| MSR | 16 MB | Microsoft reserved (no filesystem) |
| C: | most of the disk | NTFS, "Microsoft basic data" |
| Recovery | 0.5–1 GB | NTFS, "Windows recovery environment" |

**The Windows license key usually survives a wipe.** OEM machines store it in firmware, not on disk. To check:
```bash
sudo strings /sys/firmware/acpi/tables/MSDM | tail -1   # prints the embedded product key, if one exists
```
So Windows can be reinstalled later from Microsoft's ISO and will activate itself.

> [!tip] Also check for a second drive
> `lsblk -d -o NAME,SIZE,MODEL,TRAN` lists every physical drive. Some laptops have an empty second M.2 slot, and a second SSD is the safest place for another OS (see Option A below).

---

## 4. How UEFI multi-boot works (the idea that prevents most problems)

- Each OS keeps **its own folder** on the shared ESP: `EFI/ubuntu/`, `EFI/kali/`, `EFI/Microsoft/`.
- Each OS gets **its own firmware boot entry** (`efibootmgr` shows them). The F12 menu lists those entries.
- Each distro's GRUB reads **its own** `/boot/grub/grub.cfg` on **its own** root partition, found by UUID. When Ubuntu updates its kernel, Kali's boot path is untouched, and the reverse is also true.

So the target end state looks like this:
```
nvme0n1p1  ESP (shared, 1G)   EFI/ubuntu/   EFI/kali/
nvme0n1p2  Ubuntu  ext4  /    (shrunk, data intact)
nvme0n1p3  Kali    ext4  /    (new)
```
F12 menu shows **ubuntu** and **kali**, and each boots straight into its own GRUB.

The two things that break this setup:
1. **Formatting the ESP** during the Kali install. That wipes Ubuntu's bootloader.
2. **Touching the wrong partition** in the installer's partitioner.

The procedure below guards against both.

---

## 5. Procedure: add Kali, keep Ubuntu intact

### Option A (safest): install Kali on a second SSD
If the machine has a spare M.2 or SATA slot, install Kali onto the new drive and **unplug or ignore the Ubuntu drive** in the installer. Each drive then has its own ESP, and the F12 menu picks between them. Nothing on the Ubuntu disk is touched. If that's possible, skip to Step 6.

### Option B: same disk (this machine's current layout)

#### Step 1: Back up first (not optional)
```bash
# Personal data → external drive
rsync -aAXh --info=progress2 ~/ /media/$USER/EXTERNAL/home-backup/

# Partition table: lets you restore the layout exactly
sudo sfdisk -d /dev/nvme0n1 > ~/nvme0n1-partition-table.sfdisk

# The ESP itself (only 1G)
sudo tar -czf ~/esp-backup.tar.gz -C /boot/efi .

# Copy those two files to the external drive too
```
Optional but recommended: take a full system snapshot with **Timeshift** (`sudo apt install timeshift`) saved to the external drive.

#### Step 2: Record the current boot state
```bash
sudo efibootmgr -v > ~/efibootmgr-before.txt
lsblk -f > ~/lsblk-before.txt
```

#### Step 3: Make the USB sticks
- **Kali installer** ISO (kali.org → "Installer Images", 64-bit) → write it with `sudo dd if=kali.iso of=/dev/sdX bs=4M status=progress oflag=sync` or with Balena Etcher. Run `lsblk` first to **triple-check `sdX`**.
- An **Ubuntu live USB** (needed to shrink the partition with GParted). A second stick is the easiest way. Ventoy can hold both ISOs on one stick.

#### Step 4: Shrink Ubuntu's partition (from the live USB)
A mounted ext4 root **cannot be shrunk while it's running**, so this step is done from the live USB.

1. Boot the Ubuntu live USB → "Try Ubuntu".
2. Open **GParted** (`sudo apt install gparted` in the live session if it's missing).
3. Right-click `nvme0n1p2` → **Resize/Move**.
   - **Only shrink from the right side.** Leave "Free space preceding" at **0**. Moving the start of a partition is slow and risky. Shrinking the end is quick and safe.
   - Example split for this 476 GB disk: Ubuntu **~300 GB**, leaving **~175 GB** unallocated for Kali. (Ubuntu currently uses 46 GB, so there's plenty of headroom.)
4. Click **Apply** and wait. GParted runs `e2fsck` and then `resize2fs` for you.
5. Reboot **into Ubuntu** (remove the USB) and confirm it still works before you continue:
   ```bash
   df -h /          # should show the new, smaller size
   lsblk            # unallocated space won't show here, but sudo parted /dev/nvme0n1 print free will
   ```

> [!warning]
> Don't create the Kali partition in GParted. Leave the space **unallocated** and let the Kali installer create the partition. That makes the empty space easy to spot in the installer.

#### Step 5: Firmware settings check
Enter the BIOS/UEFI setup (usually F2 or Del):
- Boot mode: **UEFI only** (no Legacy/CSM). Mixing modes is the #1 cause of "OS won't show up".
- **Secure Boot:** Ubuntu works with it on. If the Kali USB or the installed Kali refuses to boot with a security violation, turn Secure Boot **off**. Ubuntu boots fine with it off too.
- SATA mode: leave it as it is now (usually AHCI). Don't change it.

#### Step 6: Install Kali
1. Boot the Kali USB from the F12 menu. Pick the entry that starts with **"UEFI:"**.
2. Choose **Graphical Install**.
3. At **Partition disks**, choose **Manual**. Never pick "Guided – use entire disk".
4. You'll see:
   ```
   #1  1.1 GB  F  ESP           ← existing EFI partition
   #2  322 GB     ext4          ← UBUNTU – DO NOT TOUCH
       175 GB     FREE SPACE    ← this is for Kali
   ```
5. Select **FREE SPACE** → *Create a new partition* → use all of it → **Primary** → Use as: **Ext4** → Mount point: **/** → Done.
6. Select the **existing ESP (#1)** and check that it says **Use as: EFI System Partition** and **Format: no**. If it shows "K" (keep) and no "F" (format), it's correct.
   - Swap: the installer may complain that there's no swap partition. Answer **No/continue**. You can add a swap file in Kali later (Step 8).
7. Select *Finish partitioning*. **Read the summary screen carefully.** The only changes listed should be:
   - creating/formatting the new partition #3 as ext4.
   - Nothing listed against partition #1 (format) or #2.
   
   If anything else appears, go back.
8. Continue the install. When it asks about GRUB, accept installing it. It goes into `EFI/kali/` on the shared ESP and adds a "kali" firmware entry. It does **not** remove `EFI/ubuntu/`.

#### Step 7: Verify both boot from F12
1. Reboot → F12 → you should see both **ubuntu** and **kali**. Boot each one.
2. In either OS:
   ```bash
   sudo efibootmgr -v      # both entries present
   sudo ls /boot/efi/EFI   # → BOOT  kali  ubuntu
   ```
3. Set the default OS (the one that boots with no key pressed):
   ```bash
   sudo efibootmgr                  # note the numbers, e.g. Boot0000* ubuntu, Boot0003* kali
   sudo efibootmgr -o 0000,0003     # Ubuntu first, Kali second
   ```
   You can also set the order in the BIOS boot-order screen.

#### Step 8: Optional polish
**Show the other OS in the GRUB menu as well** (handy, but the F12 menu works without it). On whichever OS should own the default menu:
```bash
sudo apt install os-prober
echo 'GRUB_DISABLE_OS_PROBER=false' | sudo tee -a /etc/default/grub
sudo update-grub
```
Do this on **only one** distro to avoid two GRUBs fighting over the menu. Whatever you choose, the F12 entries keep working.

**Swap file in Kali:**
```bash
sudo fallocate -l 8G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Access Ubuntu's files from Kali (read-only is safest):**
```bash
sudo mkdir -p /mnt/ubuntu && sudo mount -o ro /dev/nvme0n1p2 /mnt/ubuntu
```

---

## 6. Keeping it healthy long-term

- **Never `grub-install` by hand** onto the ESP unless you're fixing something. Normal `apt upgrade` and `update-grub` are safe in both distros.
- **Check after major upgrades:** `sudo efibootmgr` should still show both entries.
- **Firmware (BIOS) updates sometimes reset the boot order** or drop entries. Fix with `efibootmgr -o ...` or by re-adding the entry (see below).
- **Keep a live USB around.** It's the repair kit for everything below.

---

## 7. Recovery cheat sheet

**A firmware entry disappeared, but the files are still on the ESP:**
```bash
# Example: recreate the Ubuntu entry (ESP is partition 1 of nvme0n1)
sudo efibootmgr -c -d /dev/nvme0n1 -p 1 -L ubuntu -l '\EFI\ubuntu\shimx64.efi'
# Kali (check the filename with: ls /boot/efi/EFI/kali)
sudo efibootmgr -c -d /dev/nvme0n1 -p 1 -L kali -l '\EFI\kali\grubx64.efi'
```

**Ubuntu's bootloader got wiped. Reinstall it from a live USB (chroot):**
```bash
sudo mount /dev/nvme0n1p2 /mnt
sudo mount /dev/nvme0n1p1 /mnt/boot/efi
for d in dev proc sys run; do sudo mount --rbind /$d /mnt/$d; done
sudo chroot /mnt
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu
update-grub
exit
```
(Same steps for Kali with `nvme0n1p3` and `--bootloader-id=kali`.)

**ESP formatted by accident:** restore it from the backup made in Step 1:
```bash
sudo tar -xzf esp-backup.tar.gz -C /boot/efi
```
Then re-add the firmware entries as above.

**Partition table damaged:** restore it from the sfdisk dump (from a live USB):
```bash
sudo sfdisk /dev/nvme0n1 < nvme0n1-partition-table.sfdisk
```
This only restores the layout, so use it only if you understand what changed since the dump was taken.

---

## Related
- [[bashrc-and-aliases]]
- [[WIP Resetting Drivers]]
