# HP ProBook 650 G5 — BIOS Update via USB (from Linux)

## 1. Identify the exact machine
- Flip the laptop over and find the **product number** (starts with something like `8MG...` or similar) on the bottom sticker — not just "ProBook 650 G5," since there are multiple sub-SKUs with different BIOS families.
- Optionally confirm current BIOS version/date by booting into BIOS setup (F10) or checking `sudo dmidecode -s bios-version` if Linux is currently running on it.

## 2. Download the correct BIOS SoftPaq
- Go to `support.hp.com`, search your exact product number.
- Under **Software & Drivers > BIOS**, download the SoftPaq `.exe` (e.g. `SPxxxxxx.exe`).
- Double-check the version is newer than what's installed, and matches your product number exactly.

## 3. Extract the SoftPaq on Linux (no Wine needed)
Most HP SoftPaqs are self-extracting 7-Zip/InstallShield archives — you can pull the contents out without running Windows code.

```bash
sudo apt install p7zip-full   # if not already installed
mkdir ~/hp-bios-extract
cd ~/hp-bios-extract
7z x /path/to/SPxxxxxx.exe
```

- If `7z` errors out or the archive looks encrypted/won't unpack, fall back to Wine:
  ```bash
  wine SPxxxxxx.exe
  ```
  and let it run — it may self-extract to a `C:\SWSetup\` path inside the Wine prefix (`~/.wine/drive_c/SWSetup/...`).

## 4. Find the actual firmware files
After extraction, look inside the output folder for:
- A `.bin`, `.wcp`, or `.fd` file — the actual firmware image (often named an 8-character hex-like string).
- Possibly a `Hewlett-Packard` or `HP` folder structure containing a `BIOS` subfolder.
- A `.sig` or `.cat` file may accompany it — keep it alongside the image, some HP recovery methods check for it.

If it's unclear which files matter, list everything first:
```bash
find ~/hp-bios-extract -type f | sort
```
and look for the smallest set of firmware-looking files (large `.bin`/`.wcp`, a few KB `.sig`/`.cat`, maybe a `.txt` readme referencing filenames).

## 5. Format the USB drive as FAT32
Identify the drive first (be careful with the device name):
```bash
lsblk
```
Then format (replace `sdX` with your actual USB device, **not** a partition like `sdX1`):
```bash
sudo mkfs.vfat -F 32 -n HPBIOS /dev/sdX1
```
(If the drive has no partition table yet, create one with `fdisk`/`parted` first, then format the partition.)

## 6. Copy the firmware files to the USB
Mount it:
```bash
udisksctl mount -b /dev/sdX1
```
Copy the firmware image (and any `.sig`/`.cat` files found) to the **root** of the USB drive — do not put it in a subfolder unless HP's readme explicitly says to use a specific folder path (some models expect `\Hewlett-Packard\BIOS\Current\`).

```bash
cp ~/hp-bios-extract/<firmware-file>.bin /run/media/$USER/HPBIOS/
```

Unmount cleanly when done:
```bash
udisksctl unmount -b /dev/sdX1
```

## 7. Flash it on the ProBook
- Insert the USB into the ProBook 650 G5.
- Power on and immediately press **F10** to enter BIOS setup → look for an "Update System BIOS" or "Update from USB" option under Main/Advanced.
- Point it at the file on the USB drive.
- Alternative: try the **Win + B** (or Win + Up Arrow + B on some models) key combo while powering on — this triggers HP Sure Start BIOS recovery, which auto-scans USB drives for a correctly named firmware file without needing to enter setup manually.
- Do not power off or remove the USB during the flash.

## Notes / gotchas
- The single most common failure is filename/folder mismatch — the BIOS recovery routine looks for a specific filename pattern, and if the extracted file doesn't match it, the drive won't even show up as a valid update source.
- If extraction produces an installer `.exe` instead of raw firmware (i.e., a second-layer Windows installer), you may need Wine to run that inner installer far enough to unpack, since some SoftPaqs are double-wrapped.
- If you get stuck identifying which extracted file is the real firmware image, paste the `find` output and I can help narrow it down.
