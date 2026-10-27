# `cac-setup.sh` Breakdown

A line-by-line walkthrough of what `cac-setup.sh` does and why. The script
sets up a CAC (Common Access Card) smart card reader to work with Firefox
on Ubuntu, so DoD/PKI sites can authenticate with the card instead of a
username/password.

It touches: apt packages, udev rules, systemd services, (optionally) the
Firefox package itself, and Firefox's per-profile NSS certificate/module
database. It does **not** touch Secure Boot, the TPM, LUKS, or anything else
at the firmware/disk-encryption layer — see the "What it doesn't touch"
section at the bottom.

---

## Table of contents

1. [Header & safety settings](#1-header--safety-settings)
2. [Smart card middleware install](#2-smart-card-middleware-install)
3. [Card reader permissions (udev)](#3-card-reader-permissions-udev)
4. [Enabling pcscd](#4-enabling-pcscd)
5. [Firefox: snap vs .deb](#5-firefox-snap-vs-deb)
6. [Finding Firefox profiles](#6-finding-firefox-profiles)
7. [Downloading & splitting the DoD root certificates](#7-downloading--splitting-the-dod-root-certificates)
8. [Registering the CAC module + importing certs](#8-registering-the-cac-module--importing-certs)
9. [Checking the card](#9-checking-the-card)
10. [What it doesn't touch](#10-what-it-doesnt-touch)

---

## 1. Header & safety settings

```bash
set -euo pipefail
```

- `-e`: exit immediately if any command fails.
- `-u`: treat unset variables as an error instead of silently expanding to
  empty string.
- `-o pipefail`: if any command in a pipeline fails, the whole pipeline's
  exit status reflects that (not just the last command's).

Together these make the script "fail loud" rather than silently continuing
with a broken step.

The `log()` / `warn()` / `err()` helpers (lines 23-25) just wrap `echo` with
color codes (green/yellow/red) so output is easier to scan.

Three constants are set up front:

- `OPENSC_LIB` — path to the OpenSC PKCS#11 driver (`opensc-pkcs11.so`),
  the shared library that actually knows how to talk to a CAC.
- `MODULE_NAME="CAC"` — the name under which that driver gets registered
  inside Firefox.
- `DOD_CERTS_URL` — where to download the DoD's root/intermediate CA
  certificate bundle from (`dl.dod.cyber.mil`).

---

## 2. Smart card middleware install

```bash
sudo apt install -y pcscd libccid opensc pcsc-tools libnss3-tools openssl unzip wget
```

| Package | Role |
|---|---|
| `pcscd` | The PC/SC Smart Card Daemon — the background service that talks to card readers over USB and exposes a standard API to apps. |
| `libccid` | Generic USB CCID driver — supports the vast majority of smart card readers (including most CAC readers) without a vendor-specific driver. |
| `opensc` | Provides `opensc-pkcs11.so` (the PKCS#11 module Firefox will load) plus CLI tools like `opensc-tool` and `pkcs11-tool`. |
| `pcsc-tools` | Diagnostic tools (`pcsc_scan`, etc.) for testing raw reader/card communication. |
| `libnss3-tools` | Provides `certutil` and `modutil` — used later to edit Firefox's NSS cert/module databases. |
| `openssl` | Used later to convert/split the DoD certificate bundle. |
| `unzip`, `wget` | Used later to fetch and unpack the DoD cert zip. |

## 3. Card reader permissions (udev)

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=usb --action=add
```

`libccid` ships a udev rule that sets device permissions for CCID smart
card readers, but udev only *applies* rules to devices as they're plugged
in. If a reader was already plugged in before `libccid` was installed, its
permissions won't update on their own. This re-loads the rule set and
re-fires a synthetic "add" event for every USB device currently attached,
which re-applies the new permissions to a reader that's already connected
— without requiring a physical unplug/replug.

## 4. Enabling pcscd

```bash
sudo systemctl enable pcscd.socket
sudo systemctl restart pcscd.socket
sudo systemctl stop pcscd.service 2>/dev/null || true
```

`pcscd` ships as a socket-activated service: `pcscd.socket` listens, and
systemd starts `pcscd.service` on demand the first time something connects
to that socket. The script enables+restarts the *socket* (so it survives
reboots and picks up fresh permissions) and then stops any already-running
`pcscd.service`, with the comment explaining why: the socket will
transparently restart the service on the next connection, now running with
the updated reader permissions from step 3. The `|| true` means "it's fine
if the service wasn't running to begin with."

## 5. Firefox: snap vs .deb

This is the biggest and most invasive section.

**Why it exists:** Ubuntu's Firefox snap runs in a confined sandbox
(AppArmor + snap's own interfaces), and that sandbox frequently blocks
Firefox from loading the OpenSC PKCS#11 `.so` module or accessing
`pcscd`'s socket, so the CAC never gets recognized. Mozilla's official
`.deb` build runs unconfined and doesn't have this problem.

```bash
wait_for_firefox_closed() {
    while pgrep -x firefox >/dev/null || pgrep -f 'snap/firefox' >/dev/null; do
        warn "Firefox is running. Close it completely, then press Enter to continue."
        read -r
    done
}
```

A small helper that blocks until no Firefox process (snap or otherwise) is
running — it's called twice: once before any profile-touching work starts,
and again right before the NSS database edits, since Firefox locks its
cert/module databases while open and any change made while it's running
would be lost or rejected.

```bash
if snap list firefox &>/dev/null; then
    ...
    read -rp "Replace it with Mozilla's official .deb build? ... [y/N] " SWAP
    if [[ "${SWAP,,}" == "y" ]]; then
```

Only runs this whole section if Firefox is currently installed as a snap,
and only proceeds with the swap if the user explicitly opts in (default is
"no" — `${SWAP,,}` lowercases the answer so `Y`/`y` both match).

If the user says yes:

1. **Copy the snap profile out first** (lines 64-68) — snap confines
   Firefox's profile to `~/snap/firefox/common/.mozilla/firefox`; the
   `.deb` build looks in `~/.mozilla/firefox`. This copies bookmarks,
   saved passwords, history, etc. over to where the `.deb` build will look
   for them — but only if `~/.mozilla/firefox` doesn't already exist, so
   it won't clobber an existing profile on a re-run.
2. **Add Mozilla's apt repository** (lines 70-78) — downloads Mozilla's
   repo signing key into `/etc/apt/keyrings`, adds a `sources.list.d`
   entry pointing at `packages.mozilla.org` and marking it `signed-by`
   that key, and sets an apt pin priority of `1000` for anything from that
   origin. The pin is important: Ubuntu ships a *transitional* `firefox`
   package that's actually just a wrapper that installs the snap, and
   without a high-priority pin apt could prefer that package over
   Mozilla's real `.deb`.
3. **Swap the package** (lines 80-83) — `snap remove firefox`, then
   `apt install -y --allow-downgrades firefox` (downgrades allowed because
   the Mozilla build's version numbering can be lower than Ubuntu's snap
   transition package).
4. **Point the new profile at the copied one** (lines 85-93) — this is a
   workaround for a specific `.deb` Firefox quirk: on first launch, the
   `.deb` build creates a *brand new* blank profile unless there's already
   a `profiles.ini` entry keyed to a hash of the Firefox install path
   (`Install4F96D1932A9F858E` here, which corresponds to
   `/usr/lib/firefox`). The script reads which profile is currently marked
   default in `profiles.ini`, and if that install-hash section doesn't
   exist yet, appends one (plus a matching `installs.ini`) pointing that
   hash at the copied profile. This is what makes the `.deb` build actually
   open the bookmarks/passwords copied in step 1 instead of starting fresh.

If the user says no (keep the snap):

```bash
sudo snap connect firefox:pcscd
```

Explicitly grants the snap's `pcscd` plug/slot interface connection, which
is the one configuration change that *can* let a sandboxed snap Firefox
talk to the smart card daemon — not guaranteed to fully fix the module
loading issue, but the best available option while staying on the snap.

## 6. Finding Firefox profiles

```bash
for base in "$HOME/.mozilla/firefox" "$HOME/snap/firefox/common/.mozilla/firefox"; do
    for p in "$base"/*/; do
        [[ -f "$p/cert9.db" ]] && PROFILES+=("${p%/}")
    done
done
```

Walks both possible profile root locations (regular `.deb`-style path and
the snap's confined path — both are checked regardless of which branch ran
above, so this still works correctly on a re-run or if the snap-keep path
was taken) and collects every subdirectory that actually looks like an
initialized Firefox profile (i.e. has an NSS cert database, `cert9.db`).
If none are found, it warns the user to launch Firefox once (which creates
a profile) and exits cleanly rather than erroring out.

## 7. Downloading & splitting the DoD root certificates

```bash
TMP="$(mktemp -d)"
trap 'rm -rf "$TMP"' EXIT
wget -q -O "$TMP/dod.zip" "$DOD_CERTS_URL"
unzip -q "$TMP/dod.zip" -d "$TMP/dod"
```

Creates a temp directory (auto-cleaned on exit via `trap`), downloads the
official DoD PKI certificate bundle, and unzips it. This zip contains the
same certificate set in multiple formats (PEM, DER, P7B, etc.) for
different platforms.

```bash
P7B="$(find "$TMP/dod" -iname '*_DoD.der.p7b' | head -n1)"
[[ -n "$P7B" ]] || P7B="$(find "$TMP/dod" -iname '*.der.p7b' | head -n1)"
openssl pkcs7 -inform DER -in "$P7B" -print_certs -out "$TMP/all.pem" 2>/dev/null \
    || openssl pkcs7 -in "$P7B" -print_certs -out "$TMP/all.pem"
```

Picks out the `.der.p7b` file (a PKCS#7 bundle containing *all* the DoD
root and intermediate CAs in one file) — preferring the one specifically
named `*_DoD.der.p7b`, falling back to any `.der.p7b` if the exact name
isn't found (in case DoD renames the file in a future release). Converts
that single PKCS#7 bundle into one big PEM file containing every cert
concatenated together (first tries assuming DER encoding; falls back to
PEM-encoded PKCS#7 if that fails).

```bash
awk -v dir="$TMP" '/BEGIN CERT/{n++} n{print > dir "/cert" n ".pem"}' "$TMP/all.pem"
```

Splits that single multi-cert PEM file into individual `cert1.pem`,
`cert2.pem`, etc. files — one certificate per file — since the import step
next needs to import them one at a time with individual nicknames.

## 8. Registering the CAC module + importing certs

```bash
wait_for_firefox_closed
```

Called again here as a last check — in case the user launched Firefox
back up during the (potentially slow) download step above.

```bash
for profile in "${PROFILES[@]}"; do
    if modutil -dbdir "sql:$profile" -list 2>/dev/null | grep -q "$MODULE_NAME"; then
        log "  CAC module already registered"
    else
        modutil -dbdir "sql:$profile" -add "$MODULE_NAME" -libfile "$OPENSC_LIB" -force
    fi
    ...
```

For every discovered profile:

- `modutil ... -add` registers the OpenSC PKCS#11 driver
  (`opensc-pkcs11.so`) as a security device/module named `CAC` inside that
  profile's NSS database (`sql:$profile` tells NSS to use the modern
  SQLite-backed `cert9.db`/`key4.db` format rather than the legacy
  Berkeley DB format). This is what makes Firefox's "Security Devices"
  dialog aware of the smart card driver at all. It's idempotent — first
  checked with `modutil -list` so re-running the script doesn't try to
  double-register it.

```bash
    for cert in "$TMP"/cert*.pem; do
        nick="$(openssl x509 -in "$cert" -noout -subject -nameopt multiline | sed -n 's/^ *commonName *= //p')"
        certutil -A -d "sql:$profile" -n "${nick:-$(basename "$cert")}" -t "CT,C,C" -i "$cert"
    done
```

For each split-out DoD certificate: pulls its Common Name out with
`openssl x509` (used as a human-readable nickname in Firefox's cert
manager; falls back to the filename if the cert has no CN for some
reason), then imports it with `certutil -A` using trust flags `CT,C,C` —
meaning trusted (**T**) to issue SSL/TLS server certs plus valid CA
(**C**) for both the SSL and email-signing trust categories. This is what
lets Firefox actually validate a `.mil` site's certificate chain up to a
DoD root once you authenticate with the CAC.

## 9. Checking the card

```bash
if opensc-tool --list-readers 2>/dev/null | grep -q .; then
    opensc-tool --list-readers
    pkcs11-tool --module "$OPENSC_LIB" --list-slots 2>/dev/null | grep -Ei 'slot|token label' || warn "No card detected. Is the CAC inserted?"
else
    warn "No reader found by pcscd. Try unplugging and replugging the reader."
fi
```

A final sanity check, purely informational — doesn't change anything.
`opensc-tool --list-readers` confirms `pcscd` can see a physical reader at
all; if it can, `pkcs11-tool --list-slots` reports whether a card is
actually inserted and readable (showing its token label if so). This
gives the user immediate feedback on whether the hardware side is working,
separate from whether Firefox is configured correctly.

---

## 10. What it doesn't touch

For reference, since these come up as reasonable questions about any
script that installs drivers and touches system packages:

- **Secure Boot** — never referenced. Nothing here edits UEFI/NVRAM
  variables, enrolls a MOK key, or calls `mokutil`/`sbctl`. (DKMS kernel
  modules — e.g. Nvidia drivers — *can* trigger a Secure Boot MOK-signing
  prompt on apt install, but none of the packages this script installs are
  DKMS modules.)
- **TPM** — never referenced. No `tpm2_*` tools, no `systemd-cryptenroll`,
  no `clevis`, nothing that reads or clears the TPM.
- **Disk encryption** — no LUKS/cryptsetup commands anywhere.

Everything the script does is scoped to: apt packages, udev rules, the
`pcscd` systemd unit, the Firefox package choice (snap vs. `.deb`), and
per-profile files under `~/.mozilla/firefox/<profile>/` (or the snap
equivalent path).
