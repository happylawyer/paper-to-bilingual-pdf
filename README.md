# 纸质书转双语对照 PDF

中文版 | [English Version](https://github.com/happylawyer/paper-to-bilingual-pdf/blob/main/README.en.md)

这是一个 Agent skill，可以把扫描的纸质书电子化为标准字体排版的PDF，同时可以制作保持原版格式的双语对照PDF。

从 Adobe OCR、MinerU 结构提取、JSON 转 PDF，到沉浸式翻译生成左右双语对照 PDF。

## 能实现什么

- 从 MinerU 的 `layout.json` 或 `*_middle.json` 生成一份尽量贴近原书页序的 `digital` PDF。
- 保留正文、页码、页眉、章节顺序和原本的空白页。
- 生成另一份更适合连续阅读的 `reading` PDF。
- 自动备份 JSON，并按章节名规范命名。
- 对页数、正文、页码、空白页和预览页做 QA。
- 优先让 agent 安装并在本地运行 MinerU；如果本地安装、依赖或外部网站步骤受阻，再给用户清楚的人工流程。

## 效果展示

**纸质/扫描版与复刻电子书对比：**

<a href="https://raw.githubusercontent.com/happylawyer/paper-to-bilingual-pdf/main/paper-to-bilingual-pdf/assets/scanned-vs-replica.png">
  <img src="https://raw.githubusercontent.com/happylawyer/paper-to-bilingual-pdf/main/paper-to-bilingual-pdf/assets/scanned-vs-replica.png" alt="纸质/扫描版与复刻版对比" width="100%">
</a>

<br>

**沉浸式翻译后的双语对照效果，左侧原文，右侧中文译文：**

<a href="https://raw.githubusercontent.com/happylawyer/paper-to-bilingual-pdf/main/paper-to-bilingual-pdf/assets/bilingual-immersive-translate.png">
  <img src="https://raw.githubusercontent.com/happylawyer/paper-to-bilingual-pdf/main/paper-to-bilingual-pdf/assets/bilingual-immersive-translate.png" alt="双语对照效果" width="100%">
</a>

## 运作流程

1. 用 Adobe Acrobat 对扫描 PDF 做 OCR，保存为可搜索 PDF。
2. 让 agent 安装或调用本地 MinerU，对 PDF 做结构提取并生成 JSON。
3. 如果本地 MinerU 受阻，再打开 MinerU Extractor：https://mineru.net/OpenSourceTools/Extractor，上传 PDF 并下载 JSON。
4. 把生成的 JSON 绝对路径交给 Codex，或让 Codex 继续处理本地 MinerU 生成的 JSON。
5. Skill 生成 page-matched `digital` PDF 和 reflowed `reading` PDF。
6. Skill 检查页数、正文、页码、空白页和抽样预览。
7. 打开沉浸式翻译：https://app.immersivetranslate.com/file
8. 上传 `digital` PDF，生成左原文、右中文的双语对照 PDF。
9. 打印前检查首页、正文密集页、表格页和末页，确认页序没有漂移。

## 安装

在 Codex 中安装：

```sh
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo happylawyer/paper-to-bilingual-pdf \
  --path paper-to-bilingual-pdf \
  --name paper-to-bilingual-pdf
```

安装后重启 Codex，让 skill 被新会话识别。

## 使用

在 Codex 中可以这样说：

```text
Use $paper-to-bilingual-pdf to convert /path/to/layout.json into page-matched and reading PDFs.
```

也可以直接给 PDF，让 agent 先尝试安装或运行本地 MinerU：

```text
Use $paper-to-bilingual-pdf to run local MinerU on /path/to/book.pdf, then create page-matched and reading PDFs.
```

或者直接给出 JSON/PDF 路径并说明输出目录。默认输出目录是：

- `digital` PDF：`$HOME/Desktop/Json转PDF-digital`
- `reading` PDF：`$HOME/Desktop/reading`

## 目录

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

## 注意

这个 skill 负责 agent 流程、本地 MinerU 结构提取和本地 PDF 生成。Adobe、MinerU 在线提取和沉浸式翻译可能需要用户登录或手动上传；如果 agent 遇到安装、依赖、浏览器或登录阻碍，它会给出可照着执行的人工步骤。
