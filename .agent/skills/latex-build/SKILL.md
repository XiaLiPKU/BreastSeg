# PMB LaTeX Build Skill

## Goal
你是 "PMB26-BreastSeg" 项目的 LaTeX 编译与排版助手。你的任务是帮助用户编译、调试和管理 `draft/` 目录下的 LaTeX 源码，确保输出符合 IOP/PMB 期刊的格式要求。

## Environment
- **TeX Distribution**: MacTeX 2025 (`/Library/TeX/texbin/`)
- **Available Engines**: `pdflatex`, `xelatex`, `lualatex`, `latexmk`
- **Bib Tools**: `bibtex`, `biber`
- **Working Directory**: `draft/`
- **Document Class**: `iopjournal.cls` (IOP Publishing 2024, 基于 `article` class)
- **Main File**: `main.tex` (当前项目主文件；若用户指定其他 `.tex`，以用户指定为准)

## Project Files (draft/)
| 文件 | 用途 |
|---|---|
| `main.tex` | 主 LaTeX 源文件 |
| `iopjournal.cls` | IOP 期刊 class 文件 (不要修改) |
| `iopjournal-guidelines.pdf` | 期刊排版指南 (仅供参考) |
| `figure1.pdf` | 示例图片 |
| `orcid.pdf` | ORCID 图标 (cls 依赖) |

## Compilation Commands

### Standard Build (推荐)
单次快速编译，适合日常写作中预览：
```bash
# // turbo
cd /Users/xiali/Projects/PMB26-BreastSeg/draft && pdflatex -interaction=nonstopmode main.tex
```

### Full Build (含参考文献)
完整编译流程，解析所有交叉引用和参考文献：
```bash
# // turbo
cd /Users/xiali/Projects/PMB26-BreastSeg/draft && latexmk -pdf -interaction=nonstopmode main.tex
```

### Full Build (手动流程)
如果 `latexmk` 出现问题，使用手动四步编译：
```bash
cd /Users/xiali/Projects/PMB26-BreastSeg/draft && \
  pdflatex -interaction=nonstopmode main.tex && \
  bibtex main && \
  pdflatex -interaction=nonstopmode main.tex && \
  pdflatex -interaction=nonstopmode main.tex
```

### Clean Build Artifacts
清除所有中间文件，重新开始：
```bash
# // turbo
cd /Users/xiali/Projects/PMB26-BreastSeg/draft && latexmk -C
```
或手动清除：
```bash
# // turbo
cd /Users/xiali/Projects/PMB26-BreastSeg/draft && rm -f *.aux *.log *.out *.toc *.lof *.lot *.bbl *.blg *.fls *.fdb_latexmk *.synctex.gz
```

## Capabilities

### 1. Compile & Preview (编译预览)
**Trigger:** 用户要求编译、build、或预览 PDF。
**Action:**
1. 确认主 `.tex` 文件名（默认 `main.tex`）。
2. 如果用户只是预览草稿 → 使用 **Standard Build**。
3. 如果需要完整输出（含引用和交叉引用） → 使用 **Full Build**。
4. 检查编译输出，报告 errors / warnings。

### 2. Debug Compilation Errors (调试编译错误)
**Trigger:** 编译失败，用户提供 log 或要求调试。
**Action:**
1. 读取 `.log` 文件，查找 `!` 开头的错误行。
2. 定位错误源（行号 + 命令）。
3. 按以下优先级排查：
   - **Missing package**: 检查 `\usepackage{}` 是否遗漏。
   - **Undefined control sequence**: 检查命令拼写或 `iopjournal.cls` 是否定义了该命令。
   - **Missing file**: 检查图片/bib 文件路径是否正确（相对于 `draft/` 目录）。
   - **BibTeX errors**: 检查 `.bib` 文件格式和 key 是否匹配 `\cite{}`。
4. 提供修复建议并重新编译验证。

### 3. Add Packages (添加宏包)
**Trigger:** 用户需要新功能（数学公式、算法伪代码、表格增强等）。
**Action:**
按 `iopjournal.cls` 已加载的包来判断兼容性。cls 已加载：
- `fancyhdr` (页眉页脚)
- `xcolor` (颜色)
- `graphicx` (图片)
- `hyperref` (超链接，`colorlinks=true, allcolors=blue`)

常用的安全追加包：
```latex
\usepackage{amsmath,amssymb}    % 数学公式
\usepackage{algorithm2e}         % 算法伪代码
\usepackage{booktabs}            % 专业表格 (\toprule, \midrule, \bottomrule)
\usepackage{multirow}            % 合并行
\usepackage{subcaption}          % 子图
\usepackage{siunitx}             % SI 单位
\usepackage[numbers]{natbib}     % 参考文献 (注意：PMB 用 Harvard style，确认格式)
```

> [!CAUTION]
> 不要加载与 `hyperref` 冲突的包（如 `url`），`hyperref` 必须在最后加载（cls 已处理）。

### 4. Manage Figures (管理图片)
**Trigger:** 用户要插入或修改图片。
**Action:**
1. 图片文件放在 `draft/` 目录下（或创建 `draft/figures/` 子目录）。
2. 使用 `\includegraphics` 时注意路径相对于 `draft/`。
3. 推荐格式：
   - **线条图/示意图**: PDF 或 EPS (矢量格式)
   - **照片/CT 图像**: 高分辨率 TIFF 或 PNG (≥300 DPI)
4. 输出 `\begin{figure}...\end{figure}` 代码片段。

### 5. IOP-Specific Commands (IOP 专用命令)
`iopjournal.cls` 定义了以下专用命令，写作时必须使用：

```latex
\articletype{Paper}                           % 文章类型
\title{...}                                    % 标题
\author{Name$^1$\orcid{xxxx-xxxx-xxxx-xxxx}}  % 作者 + ORCID
\affil{$^1$Department, Institution, ...}       % 单位
\email{name@institution.org}                   % 通讯邮箱
\keywords{keyword1, keyword2, ...}             % 关键词
\ack{...}                                      % 致谢
\funding{...}                                  % 资助信息
\roles{...}                                    % CRediT 作者贡献
\data{...}                                     % 数据可用性声明
\suppdata{...}                                 % 补充材料
```

> [!IMPORTANT]
> 使用 `[anonymous]` 选项可隐藏作者信息，用于双盲审稿：
> ```latex
> \documentclass[anonymous]{iopjournal}
> ```

## Rules (Critical)

1. **Never modify `iopjournal.cls`**: 这是期刊官方模板，任何修改都可能导致投稿被拒。
2. **Always use `-interaction=nonstopmode`**: 防止编译中途卡住等待输入。
3. **Check log, not just output**: 即使 PDF 生成成功，也要检查 warnings（特别是 `Overfull \hbox`、`Citation undefined`、`Label multiply defined`）。
4. **File paths relative to `draft/`**: 所有 `\includegraphics`、`\input`、`\bibliography` 路径都相对于 `draft/` 目录。
5. **Keep `.bib` in `draft/`**: BibTeX 文件也应放在 `draft/` 目录下。
