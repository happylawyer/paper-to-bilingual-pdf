---
name: paper-to-bilingual-pdf
description: Convert MinerU layout JSON files into study PDFs for textbook workflows, and guide local MinerU installation/extraction when the user starts from a PDF. Use when Codex is asked to process MinerU `layout.json` or `*_middle.json` files, run or install MinerU locally, create page-matched digital PDFs, create reflowed reading PDFs, preserve page order and blank pages, back up layout JSON files, quality-check text/page numbers, explain the fallback manual pipeline from Adobe OCR to MinerU extraction to agent conversion, or guide the user through Immersive Translate bilingual textbook output and printing.
---

# 纸质书转双语对照PDF

## Project Purpose

This project turns a textbook PDF extraction workflow into a repeatable agent process. It helps users move from a scanned or digital textbook PDF to study-ready PDFs while preserving the structure that matters for coursework:正文, page order, page numbers, tables/lists when possible, and intentional blank pages.

It can produce:

- a page-matched `digital` PDF that keeps the original book-like page sequence, including blank pages and page numbers;
- a reflowed `reading` PDF for easier continuous reading on screen;
- a backed-up MinerU JSON file using a clean chapter name;
- QA notes that confirm page counts,正文 extraction, page headers/numbers, blank pages, and sampled visual previews;
- local MinerU setup/extraction guidance when the user provides a PDF instead of JSON;
- user-facing instructions for blocked outside steps: Adobe OCR, MinerU extraction, Immersive Translate bilingual output, and printing.

## Effect Examples

Use the bundled images when the user wants to see or explain the result:

- `assets/scanned-vs-replica.png`: compares the original scanned paper-book page with the page-matched electronic replica.
- `assets/bilingual-immersive-translate.png`: shows the Immersive Translate bilingual output, with original text on the left and Chinese translation on the right using a matching format.

## Operating Flow

The full workflow is:

1. Make the source PDF searchable. If it is scanned/image-based, use Adobe Acrobat OCR / Recognize Text first.
2. Install or call local MinerU first, using the upstream MinerU skill/tooling when available, to extract PDF structure into `layout.json`, `*_middle.json`, or a named chapter JSON.
3. If local MinerU installation or extraction is blocked, tell the user exactly what failed and offer the manual/online path: MinerU Extractor https://mineru.net/OpenSourceTools/Extractor, upload the searchable PDF, and download the JSON.
4. Give the JSON path to the agent.
5. The agent derives a clean base name, backs up the JSON, creates the page-matched `digital` PDF, and creates the reflowed `reading` PDF.
6. The agent QA-checks page count,正文, page numbers, blank pages, and visual previews before moving to the next chapter.
7. For bilingual output, upload the page-matched `digital` PDF to Immersive Translate: https://app.immersivetranslate.com/file
8. Export or print a bilingual PDF, preferably with original English on the left and Chinese on the right.

Use this skill to recreate paper textbook structure from MinerU layout JSON into two study PDFs:

- a page-matched `digital` PDF whose page count and blank pages match the source JSON;
- a reflowed `reading` PDF for easier continuous reading.

Preserve the user's priority:正文不缺失, page order/page numbers align, blank pages remain in the page-matched version, and temporary QA artifacts are cleaned up.

If a step cannot be automated in the current environment, state the blocker clearly, give the user the manual workflow in `references/manual-pipeline.md`, and continue with any local steps that are possible.

## Upstream MinerU Skill and Tools

At the start of a new environment, check whether the upstream MinerU skill is installed:

```sh
ls "${CODEX_HOME:-$HOME/.codex}/skills/mineru"
```

If missing and the user wants MinerU installed, install the official skill from:

```sh
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo opendatalab/MinerU-Ecosystem \
  --path skills \
  --name mineru
```

Tell the user to restart Codex to pick up newly installed skills.

Default to local MinerU when the user provides a PDF and the environment can support installation/running. If local setup fails because of missing system dependencies, model downloads, network limits, GPU/CPU constraints, path issues, or an unrecognized MinerU output shape, do not stall: explain the blocker and provide the online/manual fallback.

For users who want to inspect MinerU, compare results, or use a no-install path, provide the online Extractor:

- MinerU Extractor: https://mineru.net/OpenSourceTools/Extractor

If the user still only has a scanned or image-based PDF, ask them to OCR it first with Adobe Acrobat before running local MinerU or uploading it to MinerU online. See `references/manual-pipeline.md` for the handoff text.

## Default Directories

Unless the user specifies otherwise:

- digital output: `$HOME/Desktop/Json转PDF-digital`
- reading output: `$HOME/Desktop/reading`
- converter scripts: use the user's existing converter scripts when present, commonly under `$HOME/Documents/Codex/pdf-converters`, or ask the user for the converter path if not found.

Use these local scripts when present:

```sh
python3 "$HOME/Documents/Codex/pdf-converters/layout_json_to_page_matched_pdf.py" <input.json> <digital.pdf>
python3 "$HOME/Documents/Codex/pdf-converters/layout_json_to_clean_pdf.py" <input.json> <reading.pdf>
```

## Naming

Derive the base name from the input path.

- Strip wrapper prefixes such as `MinerU_`.
- Strip UUID/download suffixes such as `.pdf-<uuid>/layout.json`.
- Strip timestamps such as `__20260613201508`.
- Keep chapter prefixes such as `20_CCL_Ch20_Freedom_of_Expression`.
- Name outputs:
  - `<base>.pdf` in both output directories.
  - `<base>-layout.json` in the digital directory.

Examples:

- `$HOME/Downloads/09_Textbook_Ch09_Sample_Topic.json` -> `09_Textbook_Ch09_Sample_Topic.pdf`
- `$HOME/Downloads/MinerU_14_Textbook_Ch14_Sample_Topic__20260613200835.json` -> `14_Textbook_Ch14_Sample_Topic.pdf`
- `$HOME/Downloads/20_Textbook_Ch20_Sample_Topic.pdf-uuid/layout.json` -> `20_Textbook_Ch20_Sample_Topic.pdf`

## One-Chapter Workflow

Complete one chapter fully before starting the next chapter.

1. Inspect the JSON:

```sh
python3 - <<'PY'
import json, os, sys
p = sys.argv[1]
data = json.load(open(p))
infos = data.get("pdf_info", [])
print("exists", os.path.exists(p), "size", os.path.getsize(p))
print("pages", len(infos))
for i, page in enumerate(infos[:3]):
    print(i + 1, "page_idx", page.get("page_idx"), "page_size", page.get("page_size"),
          "preproc", len(page.get("preproc_blocks", [])),
          "para", len(page.get("para_blocks", [])),
          "discarded", len(page.get("discarded_blocks", [])))
if infos:
    page = infos[-1]
    print("last", len(infos), "page_idx", page.get("page_idx"), "page_size", page.get("page_size"),
          "preproc", len(page.get("preproc_blocks", [])),
          "para", len(page.get("para_blocks", [])),
          "discarded", len(page.get("discarded_blocks", [])))
PY
```

2. Create the output directories and copy the layout JSON:

```sh
mkdir -p "$HOME/Desktop/Json转PDF-digital" "$HOME/Desktop/reading"
cp <input.json> "$HOME/Desktop/Json转PDF-digital/<base>-layout.json"
```

3. Generate both PDFs:

```sh
python3 "$HOME/Documents/Codex/pdf-converters/layout_json_to_page_matched_pdf.py" \
  <input.json> "$HOME/Desktop/Json转PDF-digital/<base>.pdf"

python3 "$HOME/Documents/Codex/pdf-converters/layout_json_to_clean_pdf.py" \
  <input.json> "$HOME/Desktop/reading/<base>.pdf"
```

4. QA the files before moving on:

```sh
python3 - <<'PY'
import fitz, os
for f in ["<digital.pdf>", "<reading.pdf>"]:
    doc = fitz.open(f)
    counts = [len(p.get_text()) for p in doc]
    print(os.path.basename(os.path.dirname(f)), os.path.basename(f),
          "pages=", len(doc), "chars=", sum(counts),
          "min=", min(counts), "max=", max(counts),
          "blank_pages=", [i+1 for i,c in enumerate(counts) if c == 0])
    print("first:", doc[0].get_text()[:220].replace("\\n", " | "))
    print("last:", doc[-1].get_text()[-220:].replace("\\n", " | "))
PY
```

5. Render preview PNGs from the digital PDF for visual QA:

Render first, middle, last正文 page, and any blank pages. Inspect them with `view_image`. Verify:

- text is present and not visibly collapsed;
- page numbers and headers are present on正文 pages;
- blank pages remain blank when the JSON has no正文 blocks;
- separator pages remain in place;
- table pages are not missing.

6. If a digital page has no text, inspect that JSON page before accepting it as blank. It is acceptable when `preproc_blocks` and `para_blocks` are empty and only copyright/footer or ISBN/emond blocks exist in `discarded_blocks`.

7. Delete temporary preview PNGs after QA:

```sh
rm -f "$HOME/Desktop/Json转PDF-digital/<base>_p*_preview.png"
```

8. Report concise results:

- digital PDF path;
- reading PDF path;
- layout JSON backup path;
- page counts;
- blank pages preserved in the digital version;
- whether sampled正文, page headers/numbers, and末页 passed.

## Batch Workflow

When the user provides multiple chapters, process them strictly in order. Do not start the next chapter until the current chapter has generated both PDFs, passed QA, and had preview PNGs cleaned up.

Parallelize only inside a single chapter when operations cannot affect each other, such as generating digital and reading PDFs at the same time, or listing files while checking metadata.

## Immersive Translate Bilingual Textbook Step

After the digital PDF is ready, the user can create a bilingual textbook using Immersive Translate:

- Upload the page-matched digital PDF to https://app.immersivetranslate.com/file
- Choose bilingual/document translation mode.
- Use layout with original English on the left and Chinese on the right when available.
- Export or print the translated result to PDF.

Printing tips:

- Use the page-matched digital PDF, not the reflowed reading PDF, when page correspondence matters.
- In the browser/system print dialog, use `2 pages per sheet` or the bilingual side-by-side layout if Immersive Translate provides it.
- Prefer saving as PDF after previewing several pages: first page, a dense正文 page, a table/list page, and the last正文 page.
- Check that the left side is original English and the right side is Chinese, with no page-order drift.

If the browser step cannot be automated reliably, tell the user exactly which PDF to upload and give these instructions rather than pretending the bilingual PDF has been produced.
