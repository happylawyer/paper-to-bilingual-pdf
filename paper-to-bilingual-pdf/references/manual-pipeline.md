# Manual Pipeline

Use this when a user or another agent cannot automate one of the browser or desktop steps. Keep the wording concise and practical.

## End-to-End Flow

1. Prepare the source PDF.
   - If the PDF is scanned or image-based, open it in Adobe Acrobat.
   - Run OCR / Recognize Text.
   - Save a new searchable PDF.
   - Quickly test by selecting or copying text from several pages.

2. Extract structure with MinerU.
   - Prefer letting the agent install or run MinerU locally first.
   - If local MinerU is blocked by dependencies, downloads, runtime errors, or environment limits, use the online fallback.
   - Open https://mineru.net/OpenSourceTools/Extractor
   - Upload the OCR/searchable PDF.
   - Choose layout/document extraction.
   - Download the JSON result, usually `layout.json`, `*_middle.json`, or a named chapter JSON.
   - Keep the downloaded folder together if MinerU also produces images or intermediate files.

3. Give the JSON to the agent.
   - Provide the absolute path to the JSON file.
   - State the desired output folders if different from defaults.
   - Ask the agent to create:
     - a page-matched digital PDF;
     - a reflowed reading PDF;
     - a backup copy of the layout JSON;
     - QA for正文, page numbers, page count, and blank pages.

4. Agent conversion and QA.
   - The agent should derive a clean base name from the input path.
   - It should preserve blank pages in the page-matched digital PDF.
   - It should check page count against `len(pdf_info)`.
   - It should inspect preview images for first page, middle正文, last正文, table/list-heavy pages, and blank pages.
   - It should clean temporary preview images before reporting completion.

5. Make a bilingual textbook with Immersive Translate.
   - Use the page-matched digital PDF, not the reflowed reading PDF, when page correspondence matters.
   - Open https://app.immersivetranslate.com/file
   - Upload the digital PDF.
   - Choose document/file translation.
   - Select bilingual layout with original English on the left and Chinese on the right when available.
   - Export or print to PDF.

6. Printing checks.
   - Preview the exported PDF before printing.
   - Check first page, a dense正文 page, a table/list page, and the last正文 page.
   - Confirm original English is on the left and Chinese is on the right.
   - Confirm page order has not drifted.
   - For paper printing, use the browser/system print dialog and choose the side-by-side or `2 pages per sheet` option only if it preserves readability.

## User-Facing Handoff Text

If I cannot operate Adobe, MinerU, or Immersive Translate directly in this environment, tell the user:

```text
你可以按这个流程手动补齐外部步骤：

1. 先用 Adobe Acrobat 打开原 PDF。如果它是扫描版，运行 OCR / Recognize Text，然后另存为可搜索 PDF。
2. 先让我尝试安装或调用本地 MinerU 来提取结构并生成 JSON。
3. 如果本地 MinerU 因依赖、下载、运行环境或输出异常受阻，再打开 MinerU 在线提取器：https://mineru.net/OpenSourceTools/Extractor
4. 上传 OCR 后的 PDF，完成 layout/document extraction，下载生成的 JSON，例如 layout.json。
5. 把 JSON 的绝对路径发给我。我会生成两份 PDF：一份保留原书页序、页码和空白页的 digital PDF；一份适合连续阅读的 reading PDF，并做正文、页码、空白页 QA。
6. 如果需要中英双语对照，再打开沉浸式翻译：https://app.immersivetranslate.com/file
7. 上传我生成的 digital PDF，选择双语/文档翻译，尽量选择左侧英文原文、右侧中文译文的布局，然后导出或打印为 PDF。
8. 打印前检查首页、正文密集页、表格页和末页，确认页序没漂移、原文和中文左右对应。
```
