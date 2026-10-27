# Converting PowerPoint, Excel & Word to PDF on Linux

Covers `.pptx`, `.ppt`, `.xlsx`, `.xls`, `.csv`, `.docx`, `.doc`, `.odt`, and `.rtf`. The main tool is **LibreOffice** in headless mode (no window opens), with OnlyOffice as a fallback for files that don't render well.

---

## 1. Install

```bash
# Impress = slides, Calc = spreadsheets, Writer = documents
sudo apt install libreoffice-impress libreoffice-calc libreoffice-writer

# Microsoft fonts — fixes most "the layout looks wrong" problems
sudo apt install ttf-mscorefonts-installer
fc-cache -f
```

Check the version (some features below need 7.4+):

```bash
libreoffice --version
```

> **Note:** On some systems the command is `soffice` instead of `libreoffice`. They're the same program.

---

## 2. PowerPoint → PDF

### Single file
```bash
libreoffice --headless --convert-to pdf "My Slides.pptx"
```
The PDF lands in your **current directory**, not next to the source file.

### Choose the output folder
```bash
libreoffice --headless --convert-to pdf --outdir ~/Documents/pdfs "My Slides.pptx"
```

### Whole folder
```bash
libreoffice --headless --convert-to pdf --outdir ./pdf *.pptx *.ppt
```

### Recursive (all subfolders)
```bash
find . -type f \( -iname "*.pptx" -o -iname "*.ppt" \) -print0 |
  while IFS= read -r -d '' f; do
    libreoffice --headless --convert-to pdf --outdir "$(dirname "$f")" "$f"
  done
```
This puts each PDF next to its source file and handles spaces in names.

### Recursive → one output folder
Run from inside the folder you want to convert. Finds every deck in the current directory and all its subfolders, and puts the PDFs in a new `conversions` folder right there. Originals stay where they are.
```bash
mkdir -p conversions
find . -path ./conversions -prune -o -type f \( -iname "*.pptx" -o -iname "*.ppt" \) ! -name '~$*' \
  -exec libreoffice --headless --convert-to pdf --outdir conversions {} +
```
> ⚠️ Files with the same name in different subfolders overwrite each other. See [Duplicate filenames](#duplicate-filenames-when-flattening).

### Notes on slides
- **Speaker notes are not exported** by default. If you need them, open the file in Impress → *File → Export as PDF* → check **Export notes pages**.
- **Animations and transitions** are dropped; each slide becomes one page in its final state.
- **Embedded video/audio** won't carry over (PDF limitation).

---

## 3. Excel → PDF

### Basic conversion
```bash
libreoffice --headless --convert-to pdf "Budget.xlsx"
```

**By default this exports every sheet**, using each sheet's print settings (print area, page breaks, orientation). This is the #1 source of surprises:
- Wide sheets get chopped across many pages.
- Sheets with no print area set export the entire used range.
- Hidden sheets are skipped.

### One page per sheet (best for wide spreadsheets)
LibreOffice 7.4+ supports a filter option that scales each sheet to fit a single page:

```bash
libreoffice --headless --convert-to \
  'pdf:calc_pdf_Export:{"SinglePageSheets":{"type":"boolean","value":"true"}}' \
  "Budget.xlsx"
```

The page becomes as large as needed to fit the sheet, which is great for on-screen reading and Obsidian. It's less good for printing on paper.

### Batch convert
```bash
libreoffice --headless --convert-to pdf --outdir ./pdf *.xlsx *.xls
```

### Recursive → one output folder
Run from inside the folder you want to convert. Finds every spreadsheet in the current directory and all its subfolders, and puts the PDFs in a new `conversions` folder right there. Originals stay where they are.
```bash
mkdir -p conversions
find . -path ./conversions -prune -o -type f \( -iname "*.xlsx" -o -iname "*.xls" \) ! -name '~$*' \
  -exec libreoffice --headless --convert-to pdf --outdir conversions {} +
```

Same thing, with each sheet fit to one page:
```bash
find . -path ./conversions -prune -o -type f \( -iname "*.xlsx" -o -iname "*.xls" \) ! -name '~$*' \
  -exec libreoffice --headless --convert-to \
  'pdf:calc_pdf_Export:{"SinglePageSheets":{"type":"boolean","value":"true"}}' \
  --outdir conversions {} +
```
> ⚠️ Files with the same name in different subfolders overwrite each other. See [Duplicate filenames](#duplicate-filenames-when-flattening).

### CSV files
```bash
libreoffice --headless --convert-to pdf data.csv
```
CSVs have no formatting, so the output is plain. If it looks bad, open in Calc, adjust column widths, then export.

### Getting clean Excel output (manual method)
For anything important, it's worth 30 seconds in the GUI:
1. Open in LibreOffice Calc.
2. *Format → Page Style → Sheet tab* → **Scaling mode: Fit print range(s) to width/height** → 1 page wide.
3. *Format → Page Style → Page tab* → set **Landscape** for wide sheets.
4. *Format → Print Ranges → Define* to export only what matters.
5. *File → Export as PDF*.

Save the file afterward and future headless conversions will reuse those settings.

---

## 4. Word → PDF

### Single file / batch
```bash
libreoffice --headless --convert-to pdf "Report.docx"
libreoffice --headless --convert-to pdf --outdir ./pdf *.docx *.doc *.odt *.rtf
```

Word documents usually convert the most cleanly of the three, since they're flowing text rather than precise layouts.

### Recursive → one output folder
Run from inside the folder you want to convert. Finds every document in the current directory and all its subfolders (e.g. `./1/1.docx` through `./5/5.docx`), and puts the PDFs in a new `conversions` folder right there. Originals stay where they are.
```bash
mkdir -p conversions
find . -path ./conversions -prune -o -type f \( -iname "*.docx" -o -iname "*.doc" \) ! -name '~$*' \
  -exec libreoffice --headless --convert-to pdf --outdir conversions {} +
```
> ⚠️ Files with the same name in different subfolders overwrite each other. See [Duplicate filenames](#duplicate-filenames-when-flattening).

### Duplicate filenames when flattening
If `./1/notes.docx` and `./2/notes.docx` both exist, the commands above keep only one `notes.pdf`. This version prefixes each PDF with its folder name (`1_notes.pdf`, `2_notes.pdf`). It works for any file type; just change the extensions in the `find` line.
```bash
mkdir -p conversions /tmp/lo_out
find . -path ./conversions -prune -o -type f \( -iname "*.docx" -o -iname "*.pptx" -o -iname "*.xlsx" \) ! -name '~$*' -print0 |
  while IFS= read -r -d '' f; do
    folder=$(basename "$(dirname "$f")")
    name=$(basename "${f%.*}")
    [ "$folder" = "." ] && prefix="" || prefix="${folder}_"   # no prefix for top-level files
    libreoffice --headless --convert-to pdf --outdir /tmp/lo_out "$f" >/dev/null
    mv "/tmp/lo_out/$name.pdf" conversions/"${prefix}${name}.pdf"
  done
```
It's slower because LibreOffice starts once per file, but nothing gets overwritten.

### Notes on documents
- **Track changes:** the PDF shows the document as it currently displays. If changes are still pending, you may get the markup. Accept or reject them in Writer (*Edit → Track Changes → Accept All*) before exporting.
- **Comments** are not exported by default. To include them, open in Writer → *File → Export as PDF* → check **Comments as PDF annotations** (or *in margin*).
- **Table of contents / page numbers:** fields usually update on export, but if a TOC looks stale, open in Writer → *Tools → Update → Update All*, then export.
- **Fonts:** Word docs lean heavily on Calibri and Cambria. Install `fonts-crosextra-carlito` and `fonts-crosextra-caladea` (see Troubleshooting) or line breaks and page counts can shift.
- **Fillable forms (.docx content controls)** become flat text. If you need a fillable PDF, export from Writer with **Create PDF Form** checked.

---

## 5. All Office Files at Once → `conversions/`

The most common real-world case: a downloaded folder (like a course annex) full of subfolders with a mix of Word, PowerPoint, and Excel files. `cd` into it and run:

```bash
mkdir -p conversions
find . -path ./conversions -prune -o -type f \( -iname "*.docx" -o -iname "*.pptx" -o -iname "*.xlsx" \) ! -name '~$*' \
  -exec libreoffice --headless --convert-to pdf --outdir conversions {} +
```

Result:
```
Current Folder/
├── (your original folders and files, untouched)
└── conversions/
    ├── file1.pdf
    ├── file2.pdf
    └── ...
```

| Part | What it does |
|---|---|
| `mkdir -p conversions` | Creates the output folder inside the current directory |
| `find . ...` | Searches the current directory **and every subfolder**, at any depth |
| `-path ./conversions -prune -o` | Skips the `conversions` folder so `find` never looks inside its own output |
| `\( -iname ... -o -iname ... \)` | Matches any of the listed extensions, ignoring case |
| `! -name '~$*'` | Skips the `~$` lock files Word/Excel leave behind |
| `--outdir conversions` | Sends every PDF to `conversions/` regardless of where its source lived |
| `{} +` | Passes all files to **one** LibreOffice run (much faster than one run per file) |

**Variations:**
- **Current folder only, no subfolders:** replace `-path ./conversions -prune -o` with `-maxdepth 1`.
- **Include older formats:** add `-o -iname "*.doc" -o -iname "*.ppt" -o -iname "*.xls"` inside the parentheses.
- **Run from anywhere:** replace `.` with the folder path (quote it if it has spaces, e.g. `~/Downloads/"Annex A"`), and give `--outdir` a full path.
- **Same-named files in different subfolders?** Use the [duplicate-safe script](#duplicate-filenames-when-flattening).

> If the command finishes but `conversions/` is empty, close any open LibreOffice windows and run it again.

---

## 6. Fallback: OnlyOffice Desktop Editors

OnlyOffice has the closest Microsoft Office rendering on Linux. Use it when LibreOffice's output has broken SmartArt, shifted text boxes, or wrong charts.

```bash
# Flatpak
flatpak install flathub org.onlyoffice.desktopeditors
```

Open the file → *File → Download as → PDF*. It's GUI-only for practical purposes, so use it for problem files rather than batches.

---

## 7. Troubleshooting

| Problem | Fix |
|---|---|
| Command runs but no PDF appears | A LibreOffice window is already open. Close it (or `pkill soffice`) and run again. Headless mode silently fails if another instance is running. |
| Text overflows boxes / wrong fonts | Install `ttf-mscorefonts-installer` and run `fc-cache -f`. Some decks use Calibri/Cambria; install `fonts-crosextra-carlito` and `fonts-crosextra-caladea` for metric-compatible replacements. |
| Excel sheet split across 20 pages | Use the `SinglePageSheets` option above, or set fit-to-width scaling in Calc. |
| Only want one sheet | Set print ranges in Calc, or hide the other sheets before exporting. |
| `Error: source file could not be loaded` | Check the path and quote filenames with spaces. Password-protected files must be unlocked first. |
| Conversion is slow for many files | Expected: LibreOffice starts fresh for each command. Passing many files in **one** command (like `*.pptx`) is much faster than looping. |

---

## 8. Quick reference

```bash
# Slides → PDF
libreoffice --headless --convert-to pdf --outdir OUT file.pptx

# Spreadsheet → PDF (all sheets, normal pages)
libreoffice --headless --convert-to pdf --outdir OUT file.xlsx

# Spreadsheet → PDF (each sheet on one page)
libreoffice --headless --convert-to 'pdf:calc_pdf_Export:{"SinglePageSheets":{"type":"boolean","value":"true"}}' file.xlsx

# Document → PDF
libreoffice --headless --convert-to pdf --outdir OUT file.docx

# Everything in this folder
libreoffice --headless --convert-to pdf --outdir ./pdf *.pptx *.ppt *.xlsx *.xls *.docx *.doc

# Everything in this folder + subfolders → ./conversions
mkdir -p conversions
find . -path ./conversions -prune -o -type f \( -iname "*.pptx" -o -iname "*.ppt" -o -iname "*.xlsx" -o -iname "*.xls" -o -iname "*.docx" -o -iname "*.doc" \) ! -name '~$*' \
  -exec libreoffice --headless --convert-to pdf --outdir conversions {} +
```
