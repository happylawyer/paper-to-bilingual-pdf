# Paper Book to Bilingual PDF

[中文](README.md) | English

This is an Agent Skill that turns scanned paper books into electronic PDFs with standard-font typesetting, and can also create bilingual comparison PDFs that preserve the original layout.

From Adobe OCR, MinerU structure extraction, and JSON-to-PDF conversion to Immersive Translate side-by-side bilingual PDF output.

## What It Does

- Converts MinerU `layout.json` or `*_middle.json` into a page-matched `digital` PDF.
- Preserves body text, page order, page numbers, headers, chapter sequence, and intentional blank pages.
- Creates a reflowed `reading` PDF for continuous screen reading.
- Backs up the JSON with a clean chapter-based name.
- QA-checks page count, body text, page numbers, blank pages, and preview pages.
- Prioritizes installing and running MinerU locally through the agent; if local setup, dependencies, or external website steps are blocked, gives a clear manual fallback workflow.

## Result Examples

**Scanned/paper page compared with the replicated electronic version:**

<a href="https://raw.githubusercontent.com/happylawyer/paper-to-bilingual-pdf/main/paper-to-bilingual-pdf/assets/scanned-vs-replica.png">
  <img src="https://raw.githubusercontent.com/happylawyer/paper-to-bilingual-pdf/main/paper-to-bilingual-pdf/assets/scanned-vs-replica.png" alt="Scanned page and electronic replica" width="100%">
</a>

<br>

**Immersive Translate bilingual output, with the original text on the left and Chinese translation on the right:**

<a href="https://raw.githubusercontent.com/happylawyer/paper-to-bilingual-pdf/main/paper-to-bilingual-pdf/assets/bilingual-immersive-translate.png">
  <img src="https://raw.githubusercontent.com/happylawyer/paper-to-bilingual-pdf/main/paper-to-bilingual-pdf/assets/bilingual-immersive-translate.png" alt="Bilingual output" width="100%">
</a>

## Workflow

1. Use Adobe Acrobat OCR / Recognize Text if the PDF is scanned.
2. Have the agent install or call local MinerU to extract PDF structure and generate JSON.
3. If local MinerU is blocked, open MinerU Extractor: https://mineru.net/OpenSourceTools/Extractor, upload the PDF, and download the JSON output.
4. Give the generated JSON absolute path to Codex, or let Codex continue with the JSON produced by local MinerU.
5. The skill creates a page-matched `digital` PDF and a reflowed `reading` PDF.
6. The skill QA-checks pages, body text, page numbers, blank pages, and visual samples.
7. Open Immersive Translate: https://app.immersivetranslate.com/file
8. Upload the `digital` PDF and create a bilingual PDF with original text on the left and Chinese translation on the right.
9. Before printing, check the first page, a dense body-text page, a table page, and the last body-text page.

## Installation

Install in Codex:

```sh
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo happylawyer/paper-to-bilingual-pdf \
  --path paper-to-bilingual-pdf \
  --name paper-to-bilingual-pdf
```

Restart Codex after installation so new sessions can discover the skill.

## Usage

In Codex, say:

```text
Use $paper-to-bilingual-pdf to convert /path/to/layout.json into page-matched and reading PDFs.
```

You can also give it a PDF and let the agent try local MinerU first:

```text
Use $paper-to-bilingual-pdf to run local MinerU on /path/to/book.pdf, then create page-matched and reading PDFs.
```

Or provide the JSON/PDF path and output directories. The default output directories are:

- `digital` PDF: `$HOME/Desktop/Json转PDF-digital`
- `reading` PDF: `$HOME/Desktop/reading`

## Other Agents

If you are not using Codex, give this repository to an agent that can read files and run commands, and ask it to read `paper-to-bilingual-pdf/SKILL.md` first. It should prefer local MinerU, then fall back to `references/manual-pipeline.md` if setup or extraction is blocked.

## Structure

```text
paper-to-bilingual-pdf/
├── SKILL.md
├── agents/openai.yaml
├── assets/
│   ├── scanned-vs-replica.png
│   └── bilingual-immersive-translate.png
└── references/
    ├── manual-pipeline.md
    └── qa-checklist.md
```

## Note

This skill covers the agent workflow, local MinerU structure extraction, and local PDF generation. Adobe, MinerU online extraction, and Immersive Translate may require login or manual upload. If the agent hits installation, dependency, browser, or login blockers, it will provide exact manual instructions instead.
