---
name: auto-conference-minutes
description: 根据音频录音自动生成会议纪要。当用户上传 .wav 格式的录音文件并要求基于该录音生成会议纪要文档时使用此技能。技能首先使用本地 ASR 脚本将音频转写为文字，然后填充 .docx 会议纪要模板，生成格式化会议纪要。
---

# 自动会议纪要生成

## 概述

本技能将 `.wav` 音频录音转写为文字，然后基于转写文本生成格式化的会议纪要 `.docx` 文档。

音频转写工具链位于 `scripts/auto_minutes_python/`，包含 `qwen_long_audio.py` ASR 脚本及阿里云 ASR 调用模块。

## 工作流程

按以下步骤依次执行：

1. **步骤 1 — 激活 uv 虚拟环境**（位于 skills 文件夹下）
2. **步骤 2 — 运行 ASR 转写脚本**，将音频转换为文字
3. **步骤 3 — 读取转写文本输出**
4. **步骤 4 — 复制会议纪要模板**到目标输出目录
5. **步骤 5 — 读取模板并填充内容**，基于转写文本进行总结
6. **步骤 6 — 保存文档并交付给用户**

---

## 步骤 1 — 激活 uv 环境

激活 `scripts/auto_minutes_python/` 目录下已配置好 ASR 依赖的 uv 虚拟环境。

```bash
source /root/.qwenpaw/workspaces/default/skills/auto-conference-minutes/scripts/auto_minutes_python/.venv/bin/activate
```

---

## 步骤 2 — 运行 ASR 转写

激活环境后，进入 `scripts/auto_minutes_python/` 目录，运行转写脚本：

```bash
cd auto-conference-minutes/scripts/auto_minutes_python
python qwen_long_audio.py "<音频文件路径.wav>"
```

脚本从同目录下的 `config_qwen_split.ini` 读取配置。转写文本将保存到配置中指定的目录（默认为 `output/`），文件名格式为 `{原文件名}.qwen_long.txt`。
注意这个脚本执行的时间可以设置长一些，一些长音频可能需要更多的时间来完成转换。
---

## 步骤 3 — 读取转写文本

打开生成的 `.qwen_long.txt` 文件。生成的文件位于auto-conference-minutes/scripts/auto_minutes_python/output 这个目录下文件包含带时间戳的分段内容：

```
1	[0.000,15.320]	第一段文字内容...
2	[15.320,32.100]	第二段文字内容...
```

将所有文本分段拼接起来（忽略时间戳和分段编号），得到完整的会议转写文本，用于后续总结。

---

## 步骤 4 — 复制会议纪要模板

将模板文件从技能 assets 目录复制到用户工作区：

```
auto-conference-minutes/assets/会议纪要模板.docx
```

→ 复制到便于操作的输出位置（例如音频文件旁边或指定的输出文件夹）。

---

## 步骤 5 — 读取模板并填充内容

读取复制后的模板 `.docx` 文件，了解其结构（占位符、标题、表格等）。然后基于转写文本内容填充会议纪要，摘要内容包括：

- **会议标题 / 日期 / 与会人员** — 从音频中推断，或保留为占位符供用户填写
- **议程** — 讨论的主要议题
- **讨论摘要** — 每个议题的主要观点
- **行动项** — 分配的任务、负责人、截止时间（从提及的承诺中推断）
- **决议** — 会议期间达成的结论
- **后续安排** — 待办事项和下次会议计划

内容要简洁、结构清晰，沿用模板原有格式和标题。

---

## 步骤 6 — 保存并交付

保存填充后的文档（覆盖复制出的那份）。将最终的 `.docx` 文件路径告知用户，并简要说明生成的内容。

---

## 资源说明

本技能包含以下打包资源：

### assets/

- `会议纪要模板.docx` — 会议纪要模板文件，复制后填充

### scripts/

- `auto_minutes_python/` — 完整的音频转写工具链目录，包含：
  - `qwen_long_audio.py` — 音频转写主脚本
  - `aliyun_asr.py` — 阿里云 ASR 调用模块
  - `config_qwen_split.ini` — 转写配置（API 凭证、分段参数、输出路径等）
  - `ini_config/` — INI 配置文件解析支持模块
  - `.venv/` — uv 虚拟环境（**TODO：需在目标机器上运行 `uv venv` 创建并安装依赖**）

### references/

- `prompt_template.md` — 会议纪要总结提示词及内容填充指南

---

## 注意事项

- **uv 环境**（步骤 1 中的路径）需在目标机器上创建。进入 `scripts/auto_minutes_python/` 目录，运行 `uv venv .venv` 并安装依赖。
- `qwen_long_audio.py` 脚本依赖 `config_qwen_split.ini` 配置文件，确保包含有效的 ASR 服务 API 凭证。
- 如果音频文件超过配置的 `max_size_mb`，脚本会自动将其切分为多个片段后再进行转写。
- 转写文本的输出路径由 `.ini` 文件中的 `[output] result_dir` 配置，默认为 `output/`。
- 本技能设计在 **Linux / WSL** 环境下运行，路径使用正斜杠（`/`）。
