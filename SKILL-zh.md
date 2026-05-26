---
name: cyberpunk-ppt-maker
description: 创建暗黑霓虹赛博朋克风格 PPT 演示文稿、封面幻灯片和匹配的海报风格图像，具有一致的黑色网格、高光视觉语言。当用户要求"赛博朋克风 PPT"、"霓虹科技风封面"、匹配的幻灯片图像、暗黑霓虹科技视觉效果，或希望从新内容生成相同风格的可编辑 PPT 时使用此技能。
---

# 赛博朋克 PPT 制作器

生成一致的暗黑霓虹赛博朋克风格封面、单页海报、可编辑的 16:9 PPT 演示文稿和 `1080x1920` 竖版讲座幻灯片。保持风格稳定：黑色或近黑色背景、高对比度橙色/青色/粉色强调色、简短有力的标题。仅对以封面为主的海报请求使用小红书风格竖版输出；对完整竖版讲解使用讲座竖版路径。

## 工作流程

1. 分类请求：
- 完整演示文稿或多页 PPT：使用 `scripts/generate_cyberpunk_ppt.py`。
- 单个封面或单张海报图像：使用 [references/prompt-templates.md](references/prompt-templates.md) 中的提示模板。
- 混合请求：先构建 PPT，然后从相同的内容结构派生图像提示。

2. 构建内容结构：
- 封面、状态、标题密集型海报使用模板 A。
- 内页、列表页、解释页使用模板 B。
- 将标题压缩为简短、高影响力的短语。
- 每页保持一个情感焦点。

3. 保持固定风格：
- 应用 [references/style-guide.md](references/style-guide.md) 中的规则。
- 不要切换到商务、扁平、简约或柔和风格。
- 除非用户明确要求不可编辑的输出，否则不要将 PPT 文本展平为整页光栅图像。

4. 对于可编辑的 PPT 输出：
- 使用 [references/spec-format.md](references/spec-format.md) 编写 JSON 规范，或使用 [references/markdown-outline-format.md](references/markdown-outline-format.md) 转换 Markdown 大纲。
- 当需要快速模板时，从 [assets/examples/cyberpunk-demo-spec.json](assets/examples/cyberpunk-demo-spec.json) 开始。
- 当用户更习惯 Markdown 标题而非 JSON 时，从 [assets/examples/cyberpunk-demo-outline.md](assets/examples/cyberpunk-demo-outline.md) 开始。

**自动输出模式（推荐）：** 省略 `--output` 以自动将所有文件组织到 `~/ai-gen-ppt/<title>_<timestamp>/` 下：

```bash
python3 <SKILL_DIR>/scripts/generate_cyberpunk_ppt.py \
  --spec ./cyberpunk-spec.json
```

这将创建一个类似 `~/ai-gen-ppt/赛博风PPT_20260503_2315/` 的目录，包含 PPTX、`spec.json` 和 `assets/`。

**显式输出路径（向后兼容）：**

```bash
python3 <SKILL_DIR>/scripts/generate_cyberpunk_ppt.py \
  --spec ./cyberpunk-spec.json \
  --output ./output.pptx \
  --assets-dir ./generated_cyberpunk_assets \
  --pdf-output ./output.pdf
```
（将 `<SKILL_DIR>` 替换为实际技能路径，例如 `~/.workbuddy/skills/cyberpunk-ppt-maker`）

5. 声称成功前进行验证：
- 确认 `.pptx` 可以打开。
- 使用 `python-pptx` 验证幻灯片数量。
- 如果需要 PDF，导出并使用 `pdfinfo` 验证页数。

6. 对于直接本地封面或幻灯片 PNG 输出：

```bash
python3 <SKILL_DIR>/scripts/export_cyberpunk_images.py \
  --spec ./cyberpunk-spec.json
```

省略 `--output-dir` 以自动将 PNG 保存到 `~/ai-gen-ppt/<title>_<timestamp>/png/` 下。或使用 `--output-dir ./slide_pngs` 显式指定。

此脚本在内部构建 PPT，导出 PDF，并写入 `slide_01.png`、`slide_02.png` 等。

7. 用于 Markdown 大纲到 JSON 规范：

```bash
python3 <SKILL_DIR>/scripts/markdown_to_cyberpunk_spec.py \
  --input ./cyberpunk-outline.md
```

省略 `--output` 以自动组织到 `~/ai-gen-ppt/` 下。所有交付物（规范、PPTX、PDF、PNG）放入一个带时间戳的目录。

8. 用于一键 Markdown 到所有交付物：

```bash
python3 <SKILL_DIR>/scripts/markdown_to_cyberpunk_spec.py \
  --input ./cyberpunk-outline.md \
  --output ./cyberpunk-spec.json \
  --pptx-output ./cyberpunk.pptx \
  --pdf-output ./cyberpunk.pdf \
  --png-dir ./cyberpunk_pngs
```

这将一次性写入规范、PPT、PDF 和 PNG 幻灯片。

9. 用于参考 PPT 风格克隆入口点：

```bash
python3 <SKILL_DIR>/scripts/clone_reference_cyberpunk_style.py \
  --reference-pptx ./reference.pptx \
  --content-markdown ./new-content.md
```

省略 `--output-spec` 以自动组织到 `~/ai-gen-ppt/` 下。或为向后兼容显式指定所有路径。

## 自动输出目录

当省略显式输出路径参数时，所有生成的文件自动组织到：

    ~/ai-gen-ppt/<title>_<timestamp>/

目录结构：

    赛博风PPT_20260503_2315/
    ├── 赛博风PPT.pptx
    ├── 赛博风PPT.pdf          （使用 --pdf-output 时）
    ├── spec.json
    ├── assets/
    │   ├── poster_bg_01.jpg
    │   └── ...
    └── png/
        ├── slide_01.png
        └── ...

- 适用于 Linux、macOS 和 Windows（使用 `Path.home()`）。
- 标题从规范的 `deck_title` 字段、第一张幻灯片的标题文本或 `ghost` 字段中提取。
- 标题经过清理，删除任何平台无效的字符（`< > : " / \ | ? *`）。
- 要覆盖，传递显式 `--output` / `--output-dir` / `--output-spec` 参数。

## 封面和图像的提示工作流程

- 阅读 [references/prompt-templates.md](references/prompt-templates.md)。
- 仅用用户内容填写占位符。
- 保留 [references/style-guide.md](references/style-guide.md) 中的风格标记。
- 如果会话中有图像生成技能可用，使用填充的提示调用它。
- 如果没有图像生成技能可用，返回最终润色的提示，而不是虚构一个渲染步骤。

## PPT 工作流程

- 默认使用可编辑文本框加上光栅化背景。
- 使用大标题、简短副标题和每页 2 到 4 个简洁的内容块。
- 当大纲包含长普通标题时，让 Markdown 脚本自动将其压缩为更短的赛博朋克标题行。
- 当幻灯片包含 `Body:` 项但没有显式布局块时，让 Markdown 脚本自动推断最佳布局。
- 当用户提供普通 Markdown 文档并想要完整演示文稿时，设置 `Batch Deck: on` 以自动创建封面 + 内页 + 结束页。
- 优先使用这些布局：
  - `cover`（封面）
  - `poster_cards`（海报卡片）
  - `flow`（流程）
  - `grid_four`（四宫格）
  - `split`（分割）
  - `code_mix`（代码混合）
  - `timeline`（时间线）
  - `wide_stack`（宽幅堆叠）
  - `statement`（声明）
  - `ending`（结束）
- 不要过度填充幻灯片。如果内容很长，将其分成更多幻灯片，而不是缩小所有内容。
- 对于小红书封面或其他竖版海报请求，在 Markdown 大纲或 JSON 规范中设置 `Canvas: xhs-vertical`。
- 对于像 OMLX 讲座幻灯片这样的 `1080x1920` 竖版讲解，设置 `Canvas: lecture-vertical`。此路径使用居中的标题堆叠、讲座页面布局和更锐利的背景，没有重度模糊的小红书海报处理。

## 资源

- [references/style-guide.md](references/style-guide.md)：固定的视觉 DNA。
- [references/prompt-templates.md](references/prompt-templates.md)：可复用的封面和内页提示。
- [references/spec-format.md](references/spec-format.md)：PPT 生成的 JSON 规范架构和示例。
- [references/markdown-outline-format.md](references/markdown-outline-format.md)：用于自动构建规范的 Markdown 标题格式。
- [assets/examples/cyberpunk-demo-spec.json](assets/examples/cyberpunk-demo-spec.json)：入门 JSON 规范。
- [assets/examples/cyberpunk-demo-outline.md](assets/examples/cyberpunk-demo-outline.md)：入门 Markdown 大纲。
- [assets/examples/xhs-vertical-cover-outline.md](assets/examples/xhs-vertical-cover-outline.md)：入门竖版小红书封面大纲。
- [assets/examples/lecture-vertical-outline.md](assets/examples/lecture-vertical-outline.md)：入门 `1080x1920` 竖版讲座大纲。
- `scripts/generate_cyberpunk_ppt.py`：带有赛博朋克布局和背景的可编辑 PPT 生成器。
- `scripts/export_cyberpunk_images.py`：封面和幻灯片图像的本地 PNG 导出助手。
- `scripts/markdown_to_cyberpunk_spec.py`：Markdown 大纲到 JSON 规范转换器和一键输出编排器。
- `scripts/clone_reference_cyberpunk_style.py`：参考 PPT 入口点，用于在同一赛博朋克系统中重建新演示文稿。
