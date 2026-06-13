# QA Checklist

Use this checklist for every chapter.

1. Confirm the digital PDF page count equals `len(pdf_info)` from the JSON.
2. Confirm the reading PDF has extractable text and no unexpected blank pages.
3. Confirm digital blank pages correspond to JSON pages with no `preproc_blocks` or `para_blocks`.
4. Inspect preview images for:
   - first page;
   - one middle正文 page;
   - last正文 page;
   - every unexpected blank page;
   - table/list-heavy pages when present.
5. Confirm page headers and page numbers are visible on sampled正文 pages.
6. Delete preview PNGs after inspection.
7. In the final answer, list paths and blank pages. Do not over-explain.
