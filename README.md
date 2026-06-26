# auto-conference-minutes-skill

基于音频录音自动生成会议纪要的技能集合。上传 `.wav` 会议录音，即可自动转写为文字，并基于内置的会议纪要模板生成格式化的 `.docx` 会议纪要文档。

本仓库包含两个相互配合的技能：

| 技能 | 作用 |
|------|------|
| **auto-conference-minutes** | 将 `.wav` 音频转写为文字，并基于转写内容填充会议纪要模板 |
| **docx** | Word 文档（`.docx`）的创建、读取、编辑与分析——会议纪要最终的文档化由它完成 |

> ⚠️ **重要**：`auto-conference-minutes` 需要配合 `docx` 技能一起使用。前者负责"听写"和"总结"，后者负责"把总结写进规范的 Word 文档"。两者缺一不可。

---

## 整体工作流程

```
.wav 录音
   │
   │  ① auto-conference-minutes：ASR 转写
   ▼
.qwen_long.txt （带时间戳的分段转写文本）
   │
   │  ② auto-conference-minutes：按提示词模板归纳摘要
   ▼
会议纪要内容（议程 / 讨论摘要 / 行动项 / 决议 / 后续安排）
   │
   │  ③ docx：读取并填充 会议纪要模板.docx
   ▼
最终交付的 .docx 会议纪要
```

1. **音频转写** — 调用 `qwen_long_audio.py`，将 `.wav` 切分后逐段送入阿里云 Qwen3-ASR 识别，合并为带时间戳的文本。
2. **内容归纳** — 依据 `references/prompt_template.md` 中的提示词，将转写文本整理为结构化摘要。
3. **文档填充** — 复制 `assets/会议纪要模板.docx`，用 `docx` 技能读取模板结构并填入摘要内容，保存为最终会议纪要。

---

## 仓库结构

```
.
├── auto-conference-minutes/        # 会议纪要自动生成技能
│   ├── SKILL.md                    # 技能定义与执行步骤
│   ├── _meta.json
│   ├── assets/
│   │   └── 会议纪要模板.docx        # 会议纪要 Word 模板
│   ├── references/
│   │   └── prompt_template.md      # 会议纪要总结提示词及内容指南
│   └── scripts/auto_minutes_python/   # 音频转写工具链
│       ├── qwen_long_audio.py      # 长音频 ASR 主脚本
│       ├── aliyun_asr.py           # 阿里云 QwenASR 调用模块
│       ├── config_qwen_split.ini   # 转写配置（API 凭证、切分、输出路径）
│       ├── requirements.txt        # Python 依赖
│       └── ini_config/             # INI 配置解析支持模块
│
├── docx/                           # Word 文档操作技能（auto-conference-minutes 依赖）
│   ├── SKILL.md                    # 技能定义（创建 / 编辑 / 分析 .docx）
│   ├── LICENSE.txt
│   └── scripts/                    # 打包、解包、校验、批注、变更接受等脚本
│       ├── office/                 # unpack / pack / validate / soffice 等
│       ├── comment.py
│       └── accept_changes.py
│
├── LICENSE
└── README.md
```

---

## 环境要求

### 运行环境

- **Linux / WSL**（技能路径使用正斜杠，未在原生 Windows 下验证）
- **Python 3.10+**
- **uv**（用于创建并管理虚拟环境）

### auto-conference-minutes 依赖

进入 `auto-conference-minutes/scripts/auto_minutes_python/` 创建 uv 虚拟环境并安装依赖：

```bash
cd auto-conference-minutes/scripts/auto_minutes_python
uv venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

依赖（见 `requirements.txt`）：`openai`、`dashscope`、`soundfile`。

### docx 技能依赖

`docx` 技能在生成最终 Word 文档时需要以下外部工具可用（位于 `PATH` 中）：

- **docx**（`npm install -g docx`）—— 新建文档
- **LibreOffice**（`soffice`）—— `.doc`→`.docx` 转换、接受修订、导出 PDF
- **pandoc** —— 文本提取
- **pdftoppm**（poppler-utils）—— 文档转图片

> Windows 上若依赖缺失，`docx` 技能会报告依赖问题并停止，不会反复重试。

### API 凭证

转写脚本通过 `config_qwen_split.ini` 读取阿里云百炼（DashScope）API Key，也可用环境变量 `DASHSCOPE_API_KEY` 覆盖。请将 `[api] api_key` 替换为有效凭证后再运行。

---

## 使用方法

### 使用前提

确保以下条件已满足：

1. `auto-conference-minutes` 与 `docx` 两个技能已安装在同一个 skills 目录下。
2. Python 虚拟环境已创建并安装依赖（见「环境要求」一节）。
3. `config_qwen_split.ini` 中已填写有效的阿里云百炼 API Key（或已设置 `DASHSCOPE_API_KEY` 环境变量）。
4. 待转写的 `.wav` 会议录音文件已准备好。

### 适用平台

本技能可在以下支持技能（Skill）机制的 AI 助手平台中使用：

| 平台 | 说明 |
|------|------|
| **Claude Code** | Anthropic 官方 CLI 工具，支持 Skill 加载，可在终端或 VS Code 扩展中使用 |
| **QwenPaw** | 阿里云百炼的 AI 助手平台，原生支持技能机制，本技能最初在此平台开发与测试 |
| **OpenClaude** | 开源 Claude 助手项目，支持自定义技能加载 |

只要平台支持读取 `SKILL.md` 并按其中定义的步骤自动执行，即可使用本技能。将 `auto-conference-minutes` 和 `docx` 两个技能目录放置在该平台的 skills 目录下，平台即可自动识别与调用。

### 如何使用

在上述平台中，向 AI 助手上传 `.wav` 会议录音文件，并表达需要生成会议纪要的意图（例如："请基于这段会议录音生成会议纪要"）。技能将自动执行以下流程：

- **输入**：`.wav` 格式的会议录音文件
- **处理**：音频转写 → 内容归纳 → 模板填充（全程自动，无需手动操作）
- **输出**：一份格式化的 `.docx` 会议纪要文档，包含会议基本信息、议程、讨论摘要、行动项、决议、后续安排等章节

### 技能内部工作流程

技能在收到请求后按以下步骤依次执行（详见 `auto-conference-minutes/SKILL.md`）：

1. **激活 uv 虚拟环境** — 激活 `scripts/auto_minutes_python/.venv`
   > ⚠️ `SKILL.md` 步骤 1 中给出的示例路径（`/root/.qwenpaw/...`）需替换为你本机 `.venv` 的**真实绝对路径**，不要照抄。
2. **ASR 转写** — 运行 `qwen_long_audio.py`，将音频切分后逐段送入阿里云 Qwen3-ASR 识别，输出带时间戳的 `.qwen_long.txt` 文件。
3. **文本归纳** — 读取转写文本，按 `references/prompt_template.md` 的结构归纳为会议纪要摘要。
4. **模板填充** — 复制 `assets/会议纪要模板.docx`，使用 `docx` 技能读取模板结构并填入归纳内容，保存为最终的 `.docx` 文件。
5. **交付** — 将最终文档路径返回给用户。

---

## 配置说明（config_qwen_split.ini）

| 段 / 键 | 说明 |
|---------|------|
| `[api] api_key` | 阿里云百炼 API Key（可被 `DASHSCOPE_API_KEY` 覆盖） |
| `[api] region` | 地域：`cn`（北京）/ `intl`（新加坡） |
| `[qwen] model` | ASR 模型名称（如 `qwen3-asr-flash-2026-02-10`） |
| `[qwen] language` | 语言代码，留空自动识别（`zh` / `en` / `ja` / `yue` 等） |
| `[qwen] enable_itn` | 是否启用逆文本标准化（ITN） |
| `[split] max_size_mb` | 切分大小（MB），QwenASR 单段限制 ≤10MB，建议 3–6MB |
| `[split] keep_chunks` | 是否保留切分中间文件 |
| `[split] use_threads` / `threads` | 是否并发及并发线程数（I/O 密集，建议 3–5） |
| `[output] result_dir` | 结果保存目录，默认 `output` |

---

## 注意事项

- `auto-conference-minutes` 与 `docx` 技能需放在同一 skills 目录下协同使用。
- 转写脚本的切分上限受 QwenASR 单段 ≤10MB 限制，请勿将 `max_size_mb` 设置过大。
- 会议纪要内容遵循"实事求是、简洁精炼、可操作、保留上下文、使用中文"的质量规则，详见 `auto-conference-minutes/references/prompt_template.md`。

## 致谢

本项目的 `auto-conference-minutes` 技能为作者原创，但其中依赖的 `docx` 技能来自 **Anthropic, PBC**（© 2025），特此致谢。

`docx` 技能是 Anthropic 提供的专有技能（Proprietary Skill），**不是开源项目**，其使用受 [Anthropic 服务条款](https://www.anthropic.com/legal/consumer-terms) 约束。完整许可证内容见 `docx/LICENSE.txt`。

感谢 Anthropic 提供这一高质量的 Word 文档操作技能，使得 `auto-conference-minutes` 能够基于它完成会议纪要的文档化输出。本项目在符合 Anthropic 许可条款的前提下使用该技能，并遵守开源社区的归属与致谢规范。

## 许可证

- 本仓库根目录的 `LICENSE` 适用于 `auto-conference-minutes` 技能及本仓库中作者原创部分。
- `docx` 技能的许可证见 `docx/LICENSE.txt`，版权归 Anthropic, PBC 所有。

---

---

# English

A skill collection that automatically generates meeting minutes from audio recordings. Upload a `.wav` recording of your meeting, and it will be transcribed to text and turned into a formatted `.docx` meeting-minutes document based on the built-in template.

This repository contains two skills that work together:

| Skill | Purpose |
|-------|---------|
| **auto-conference-minutes** | Transcribes `.wav` audio to text and fills the meeting-minutes template from the transcript |
| **docx** | Creates, reads, edits, and analyzes Word (`.docx`) documents — it produces the final document for the minutes |

> ⚠️ **Important**: `auto-conference-minutes` must be used together with the `docx` skill. The former handles "dictation" and "summarization"; the latter handles "writing the summary into a proper Word document". Neither can do the job alone.

---

## Overall Workflow

```
.wav recording
   │
   │  ① auto-conference-minutes: ASR transcription
   ▼
.qwen_long.txt (timestamped, segmented transcript)
   │
   │  ② auto-conference-minutes: summarize per the prompt template
   ▼
Minutes content (agenda / discussion summary / action items / decisions / follow-ups)
   │
   │  ③ docx: read and fill 会议纪要模板.docx
   ▼
Final delivered .docx meeting minutes
```

1. **Audio transcription** — Run `qwen_long_audio.py` to split the `.wav` and send each chunk to Alibaba Cloud Qwen3-ASR, then merge the results into timestamped text.
2. **Summarization** — Following the prompt in `references/prompt_template.md`, organize the transcript into a structured summary.
3. **Document fill-in** — Copy `assets/会议纪要模板.docx`, use the `docx` skill to read the template structure and inject the summary, then save the final minutes.

---

## Repository Structure

```
.
├── auto-conference-minutes/        # Meeting-minutes auto-generation skill
│   ├── SKILL.md                    # Skill definition and execution steps
│   ├── _meta.json
│   ├── assets/
│   │   └── 会议纪要模板.docx        # Meeting-minutes Word template
│   ├── references/
│   │   └── prompt_template.md      # Summarization prompt and content guidelines
│   └── scripts/auto_minutes_python/   # Audio transcription toolchain
│       ├── qwen_long_audio.py      # Long-audio ASR main script
│       ├── aliyun_asr.py           # Alibaba Cloud QwenASR client module
│       ├── config_qwen_split.ini   # Transcription config (API creds, splitting, output)
│       ├── requirements.txt        # Python dependencies
│       └── ini_config/             # INI config parsing support module
│
├── docx/                           # Word document skill (required by auto-conference-minutes)
│   ├── SKILL.md                    # Skill definition (create / edit / analyze .docx)
│   ├── LICENSE.txt
│   └── scripts/                    # pack, unpack, validate, comment, accept-changes, etc.
│       ├── office/                 # unpack / pack / validate / soffice, ...
│       ├── comment.py
│       └── accept_changes.py
│
├── LICENSE
└── README.md
```

---

## Requirements

### Runtime

- **Linux / WSL** (skill paths use forward slashes; not verified on native Windows)
- **Python 3.10+**
- **uv** (to create and manage the virtual environment)

### auto-conference-minutes dependencies

Create a uv venv inside `auto-conference-minutes/scripts/auto_minutes_python/` and install dependencies:

```bash
cd auto-conference-minutes/scripts/auto_minutes_python
uv venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Dependencies (see `requirements.txt`): `openai`, `dashscope`, `soundfile`.

### docx skill dependencies

When producing the final Word document, the `docx` skill requires these external tools on `PATH`:

- **docx** (`npm install -g docx`) — new document creation
- **LibreOffice** (`soffice`) — `.doc`→`.docx` conversion, accepting tracked changes, PDF export
- **pandoc** — text extraction
- **pdftoppm** (poppler-utils) — document-to-image workflows

> On Windows, if a dependency is missing the `docx` skill reports the issue and stops; it does not retry indefinitely.

### API credentials

The transcription script reads the Alibaba Cloud Bailian (DashScope) API key from `config_qwen_split.ini`, overridable by the `DASHSCOPE_API_KEY` environment variable. Replace `[api] api_key` with a valid credential before running.

---

## Usage

### Prerequisites

Make sure the following conditions are met:

1. Both `auto-conference-minutes` and `docx` skills are installed in the same skills directory.
2. The Python virtual environment has been created and dependencies installed (see the "Requirements" section).
3. A valid Alibaba Cloud Bailian API key is set in `config_qwen_split.ini` (or via the `DASHSCOPE_API_KEY` environment variable).
4. The `.wav` meeting recording file to be transcribed is ready.

### Supported Platforms

This skill can be used on AI assistant platforms that support the Skill mechanism:

| Platform | Description |
|----------|-------------|
| **Claude Code** | Anthropic's official CLI tool with Skill support; usable in terminal or VS Code extension |
| **QwenPaw** | Alibaba Cloud Bailian's AI assistant platform with native Skill support; this skill was originally developed and tested here |
| **OpenClaude** | Open-source Claude assistant project that supports custom Skill loading |

As long as the platform can read `SKILL.md` and automatically execute the steps defined within, this skill is usable. Place both the `auto-conference-minutes` and `docx` skill directories in the platform's skills directory, and the platform will auto-detect and invoke them.

### How to Use

On any of the above platforms, upload a `.wav` meeting recording file to the AI assistant and express the intent to generate meeting minutes (e.g., "Please generate meeting minutes based on this recording"). The skill will automatically execute the following flow:

- **Input**: a `.wav` meeting recording file
- **Processing**: audio transcription → content summarization → template fill-in (fully automatic, no manual steps required)
- **Output**: a formatted `.docx` meeting-minutes document containing sections for basic info, agenda, discussion summary, action items, decisions, and follow-ups

### Skill Internal Workflow

Upon receiving a request, the skill executes the following steps in order (see `auto-conference-minutes/SKILL.md` for details):

1. **Activate the uv virtual environment** — activate `scripts/auto_minutes_python/.venv`
   > ⚠️ The example path in step 1 of `SKILL.md` (`/root/.qwenpaw/...`) must be replaced with the **real absolute path** of `.venv` on your machine; do not copy it verbatim.
2. **ASR transcription** — run `qwen_long_audio.py` to split the audio and send each chunk to Alibaba Cloud Qwen3-ASR, producing a timestamped `.qwen_long.txt` file.
3. **Summarization** — read the transcript and organize it into a structured minutes summary per `references/prompt_template.md`.
4. **Template fill-in** — copy `assets/会议纪要模板.docx`, use the `docx` skill to read the template structure and inject the summarized content, then save the final `.docx` file.
5. **Delivery** — return the final document path to the user.

---

## Configuration (config_qwen_split.ini)

| Section / Key | Description |
|---------------|-------------|
| `[api] api_key` | Alibaba Cloud Bailian API key (overridable by `DASHSCOPE_API_KEY`) |
| `[api] region` | Region: `cn` (Beijing) / `intl` (Singapore) |
| `[qwen] model` | ASR model name (e.g. `qwen3-asr-flash-2026-02-10`) |
| `[qwen] language` | Language code; empty = auto-detect (`zh` / `en` / `ja` / `yue`, etc.) |
| `[qwen] enable_itn` | Enable inverse text normalization (ITN) |
| `[split] max_size_mb` | Chunk size (MB); QwenASR limit ≤10MB per segment, recommended 3–6MB |
| `[split] keep_chunks` | Whether to keep intermediate chunk files |
| `[split] use_threads` / `threads` | Enable concurrency and thread count (I/O-bound; 3–5 recommended) |
| `[output] result_dir` | Output directory, default `output` |

---

## Notes

- `auto-conference-minutes` and `docx` must live together under the same skills directory.
- `.venv/`, `*.ini` (configs containing real credentials), `__pycache__/`, `*.pyc`, `*.wav`, and `output/` are excluded by `.gitignore` and won't enter the repository; create the venv and fill in API credentials on the target machine.
- The chunk size is bounded by the QwenASR per-segment limit of ≤10MB — do not set `max_size_mb` too large.
- Minutes content follows the quality rules "factual, concise, actionable, context-preserving, in Chinese" — see `auto-conference-minutes/references/prompt_template.md`.

## Acknowledgments

The `auto-conference-minutes` skill in this repository is original work by the author, but it depends on the `docx` skill, which is provided by **Anthropic, PBC** (© 2025). We would like to express our gratitude.

The `docx` skill is a Proprietary Skill provided by Anthropic and is **not open source**. Its use is governed by [Anthropic's Terms of Service](https://www.anthropic.com/legal/consumer-terms). The full license text is available in `docx/LICENSE.txt`.

We thank Anthropic for providing this high-quality Word document manipulation skill, which enables `auto-conference-minutes` to produce properly formatted meeting-minutes documents. This project uses the `docx` skill within the scope of Anthropic's license terms and follows open-source community conventions for attribution and acknowledgment.

## License

- The `LICENSE` at the root of this repository applies to the `auto-conference-minutes` skill and all original parts authored by the repository owner.
- The `docx` skill is licensed under `docx/LICENSE.txt`, copyright Anthropic, PBC.
