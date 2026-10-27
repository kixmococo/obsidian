# Images & Screenshots → Searchable PDF on Linux

Two stages:
1. **Combine** a folder of photos/screenshots into one PDF (`img2pdf`).
2. **OCR** that PDF so text inside the images becomes searchable and selectable (`ocrmypdf`).

OCR adds an invisible text layer on top of each image. The page looks exactly the same, but you can now `Ctrl+F`, highlight, and copy text, and Obsidian can search it.

---

## 1. Install

```bash
sudo apt install img2pdf ocrmypdf tesseract-ocr tesseract-ocr-eng
```

Other languages (handy for Spanish/Korean material):

```bash
sudo apt install tesseract-ocr-spa tesseract-ocr-kor
tesseract --list-langs     # confirm what's installed
```

---

# Part 1: Folder of Images → PDF

## Why img2pdf
`img2pdf` embeds JPEGs **without re-compressing** them, so there's no quality loss and it's fast. It's a better default than ImageMagick, which re-encodes images and on Ubuntu often blocks PDF output by security policy.

## Basic usage
```bash
cd ~/Pictures/screenshots
img2pdf *.png -o combined.pdf
```

Mixed formats:
```bash
img2pdf *.png *.jpg *.jpeg -o combined.pdf
```

## ⚠️ Get the page order right
Shell globs sort alphabetically, so `img10.png` comes **before** `img2.png`. Use natural (version) sort:

```bash
ls -v *.png | tr '\n' '\0' | xargs -0 img2pdf -o combined.pdf
```

Screenshot tools usually name files by timestamp (`Screenshot from 2026-10-27 14-03-11.png`), which already sorts correctly. To sort by when the files were **modified** instead:

```bash
ls -tr *.png | tr '\n' '\0' | xargs -0 img2pdf -o combined.pdf
```

The most reliable approach for anything important is to rename the files first: `001.png`, `002.png`, and so on.

## Page size
By default each page is exactly the size of its image. To put everything on standard paper:

```bash
img2pdf *.jpg --pagesize Letter -o combined.pdf      # US Letter
img2pdf *.jpg --pagesize A4 -o combined.pdf          # A4
img2pdf *.jpg --pagesize Letter --border 0.5in -o combined.pdf   # with margins
```

## Common error: alpha channel
PNG screenshots often have transparency, and img2pdf may refuse them:

```
Refusing to work on images with alpha channel
```

Fix it by flattening onto white, **on copies**, so your originals stay untouched:

```bash
sudo apt install imagemagick
mkdir flat
for f in *.png; do
  convert "$f" -background white -alpha remove -alpha off "flat/$f"
done
cd flat
img2pdf $(ls -v *.png) -o ../combined.pdf
```

(On ImageMagick 7, use `magick` instead of `convert`.)

## Phone photos coming out sideways
Phones store rotation as metadata. Bake it into the pixels first:

```bash
mogrify -auto-orient *.jpg      # modifies files in place — use copies
```

`ocrmypdf --rotate-pages` (Part 2) also fixes sideways pages based on the text direction.

---

# Part 2: OCR — Making Text Searchable

## Basic usage
```bash
ocrmypdf combined.pdf combined-ocr.pdf
```

## Recommended command
```bash
ocrmypdf --rotate-pages --deskew -l eng combined.pdf combined-ocr.pdf
```

| Flag | What it does |
|---|---|
| `--rotate-pages` | Detects pages that are sideways or upside down and fixes them |
| `--deskew` | Straightens slightly crooked scans and photos |
| `-l eng` | OCR language. Combine with `+`: `-l eng+spa` |
| `--clean` | Removes speckles before OCR (helps photos of paper; needs `unpaper`) |
| `--optimize 1` | Light file-size reduction (default). `2` or `3` shrink more but need extra tools (`pngquant`, `jbig2`) |
| `--jobs 4` | Use 4 CPU cores |

## PDFs that already contain some text
OCRmyPDF stops with an error if any page already has real text (common for a PowerPoint-exported PDF with a few screenshot pages). Choose a mode:

| Flag | Behavior | Use when |
|---|---|---|
| `--skip-text` | OCRs only pages with no text; leaves the others alone | **Best default for mixed PDFs** |
| `--redo-ocr` | Replaces old/bad OCR layers but keeps real text | A PDF has poor existing OCR |
| `--force-ocr` | Turns every page into an image and OCRs it all | Last resort. Destroys real text and increases file size |

```bash
ocrmypdf --skip-text --rotate-pages -l eng handout.pdf handout-ocr.pdf
```

> `--deskew` and `--clean` can't be combined with `--redo-ocr`.

## OCR an image directly (skip Part 1)
For a single image:
```bash
ocrmypdf --image-dpi 300 screenshot.png screenshot.pdf
```
`--image-dpi` is needed when the image has no DPI info, which is common for screenshots.

## Verify it worked
```bash
pdftotext combined-ocr.pdf - | less
```
If you see the text, it's searchable. You can also open the PDF and try `Ctrl+F`.

---

# Part 3: One-Shot Script

Save as `~/bin/imgs2searchable` (create `~/bin` if needed and make sure it's on your `PATH`):

```bash
#!/usr/bin/env bash
# Usage: imgs2searchable [folder] [output.pdf] [lang]
# Combines all images in a folder (natural sort order) into one OCR'd PDF.
set -euo pipefail

dir="${1:-.}"
out="${2:-combined-ocr.pdf}"
lang="${3:-eng}"

tmp="$(mktemp -d)"
trap 'rm -rf "$tmp"' EXIT

# Flatten transparency and fix rotation on copies, preserving order
i=0
while IFS= read -r f; do
  i=$((i+1))
  convert "$dir/$f" -auto-orient -background white -alpha remove -alpha off \
    "$tmp/$(printf '%04d' "$i").png"
done < <(ls -v "$dir" | grep -iE '\.(png|jpe?g|webp|bmp|tiff?)$')

if [ "$i" -eq 0 ]; then
  echo "No images found in $dir" >&2
  exit 1
fi

img2pdf "$tmp"/*.png -o "$tmp/combined.pdf"
ocrmypdf --rotate-pages --deskew -l "$lang" --jobs "$(nproc)" \
  "$tmp/combined.pdf" "$out"

echo "Done: $out ($i pages)"
```

```bash
chmod +x ~/bin/imgs2searchable

imgs2searchable ~/Pictures/lecture-screenshots notes.pdf
imgs2searchable ./photos spanish-handout.pdf eng+spa
```

This script converts everything to PNG for reliability, which makes files larger than a pure `img2pdf` JPEG run. OCRmyPDF's optimizer recovers some of that.

---

## Tips for better OCR accuracy

- **Resolution matters most.** Tesseract works best around 300 DPI. Crisp screenshots are usually fine; tiny or zoomed-out text is not. Take screenshots at full zoom when you can.
- **Photos of paper:** shoot straight on, with even lighting and no shadows, and fill the frame. Use `--clean --deskew`.
- **Dark mode screenshots** (light text on dark backgrounds) OCR worse. Light mode gives better results.
- **Handwriting** will mostly fail. Tesseract is built for printed text.
- **Tables and code** get recognized as text, but the layout won't be perfect when copied.
- OCR is never 100% accurate. It's great for **searching**, but double-check anything you copy out for exact values.

---

## Quick reference

```bash
# Folder of images → PDF (natural order)
ls -v *.png | tr '\n' '\0' | xargs -0 img2pdf -o combined.pdf

# PDF → searchable PDF
ocrmypdf --rotate-pages --deskew -l eng combined.pdf combined-ocr.pdf

# Mixed PDF (some pages already have text)
ocrmypdf --skip-text -l eng input.pdf output.pdf

# Single screenshot → searchable PDF
ocrmypdf --image-dpi 300 shot.png shot.pdf

# Check the text layer
pdftotext output.pdf - | less
```
