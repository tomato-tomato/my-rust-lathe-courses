---
name: lathe
description: 按需生成、续写、验证、提问动手实践型技术教程。当用户说 /lathe 并附带主题时使用，如 "/lathe 用 Zig 构建数字合成器"；也处理续写(extend)、验证(verify)、提问(ask)、打标签(tag)、创建语气(voice)和列出教程(list)等所有教程相关操作。支持中文和英文关键词。
---

# Lathe — 教程体系

Lathe 是一套完整的动手实践型教程管理工具，涵盖生成、续写、验证、提问、标签、语气和列表七大功能。标杆是 Robert Nystrom（Crafting Interpreters）、Sam Who、Julia Evans、Bartosz Ciechanowski、Amit Patel（Red Blob Games）的写作水平。

## 使用方式

所有操作通过 `/lathe` 调用，后跟**意图关键词**（中文或英文均可）和参数：

```
/lathe <主题>                          → 生成新教程
/lathe 续写|extend <slug> [指导]       → 续写教程下一部分
/lathe 验证|verify <slug>              → 端到端验证教程
/lathe 提问|ask <slug> <part> <问题>   → 提问教程内容
/lathe 标签|tag <slug>                 → 管理教程标签
/lathe 语气|voice [名称]               → 创建自定义语气
/lathe 列表|list [slug]                → 列出所有教程或查看某教程状态
```

也支持纯自然语言触发（无需 `/lathe`），AI 会自动识别意图。中文和英文关键词均可识别。

## 教程存储路径配置

所有教程统一存储在可配置的基础目录下，路径记录在 `~/.lathe/config.json`：

```json
{
  "tutorials_base_path": "~/others/lathe_tutorials"
}
```

**每次调用本技能时，首先解析存储路径：**

1. 尝试读取 `~/.lathe/config.json`，获取 `tutorials_base_path`。
2. 如果文件不存在或字段缺失：告诉用户 *"这是首次使用 Lathe。教程默认存储在 `~/others/lathe_tutorials/`，要使用其他位置吗？"*，用户确认后创建 `~/.lathe/` 目录并写入 `config.json`。
3. 后续操作中用 `<TUTORIALS_DIR>` 代指解析到的路径。

> 默认值 `~/others/lathe_tutorials/` 是之前一直使用的位置，已存在的教程都在那里。

## 技能目录

本技能的入口文件是 `SKILL.md`，各操作模块文件位于其同级的 `references/` 子目录下。后文中用 `<SKILL_DIR>` 代指 **SKILL.md 所在目录**（如 `~/.qoder/skills/lathe` 或 `<项目>/.reasonix/skills/lathe`）——需要读取模块文件时，路径以该目录为准。

## 操作路由

根据用户消息中的意图关键词，读取对应模块文件执行完整协议：

| 用户意图 | 识别关键词（中文 / 英文） | 使用示例 | 读取模块 |
|----------|-------------------------|----------|----------|
| **生成新教程** | 主题描述（无下述关键词时） / topic description | `/lathe 用 Go 构建 Raft` | [references/generate.md](references/generate.md) |
| **续写教程** | 续写、继续、下一部分 / extend、continue、next、add、part | `/lathe extend raft-go` 或 `/lathe 续写 raft-go` | [references/extend.md](references/extend.md) |
| **验证教程** | 验证、检查、测试 / verify、check、test、validate | `/lathe verify raft-go` 或 `/lathe 验证 raft-go` | [references/verify.md](references/verify.md) |
| **提问内容** | 提问、问、问题 / ask、question | `/lathe ask raft-go part-01 why use ring buffer` | [references/ask.md](references/ask.md) |
| **管理标签** | 标签、打标签 / tag、tags、label | `/lathe tag raft-go` 或 `/lathe 标签 raft-go` | [references/tag.md](references/tag.md) |
| **创建语气** | 语气、风格 / voice、tone、style | `/lathe voice terse` 或 `/lathe 语气 terse` | [references/voice.md](references/voice.md) |
| **列出教程** | 列表、列出、查看状态 / list、ls、status | `/lathe list` 或 `/lathe 列表 raft-go` | [references/list.md](references/list.md) |

**路由规则：**
1. 解析 `/lathe` 后的第一个关键词来确定意图。中文和英文关键词均可识别（如 "续写" 等价于 "extend"，"验证" 等价于 "verify"，"提问" 等价于 "ask"，"标签" 等价于 "tag"，"语气" 等价于 "voice"，"列表" 等价于 "list"）。疑问词（为什么、怎么 / why、how、what）**仅在同时给出已知 slug 时**视为提问意图，否则按歧义处理——避免把 `/lathe 怎么用` 这类话误判为 ask。
2. **意图不明确时，先询问再执行（见下方"意图歧义处理"）。** 不要擅自假设用户想要生成教程。
3. 识别到意图后，**立即读取对应模块文件**（位于 `<SKILL_DIR>/references/`），按其中的完整协议执行。
4. 也支持纯自然语言触发（不含 `/lathe`），通过上下文语义匹配意图，中英文均可。
5. 模块文件可能引用下方的共享配置。

## 意图歧义处理

当用户输入无法明确判断意图时（例如：只输入了 `/lathe`、只给了一个 slug、或者文本含糊），**不要默认生成教程**，而是向用户展示选项让其选择：

```
你想对 Lathe 教程执行什么操作？

1. 🟢 **生成新教程** — 为新主题创建教程第一部分
2. 📝 **续写教程 (extend)** — 为已有教程添加下一部分
3. ✅ **验证教程 (verify)** — 端到端验证教程是否可用
4. ❓ **提问内容 (ask)** — 针对教程某个部分提问
5. 🏷️ **管理标签 (tag)** — 为教程选择或更新标签
6. 🎙️ **创建语气 (voice)** — 创建自定义写作语气预设
7. 📋 **列出教程 (list)** — 查看所有教程及其状态

请回复编号或关键词（中英文均可）。
```

**歧义判定标准：**
- `/lathe`（无后续内容） → 歧义，展示选项
- `/lathe raft-go`（仅 slug，无关键词） → 歧义，展示选项（可能是续写、验证、提问等）
- `/lathe 这个教程怎么样`（含糊表达） → 歧义，展示选项
- `/lathe how is this tutorial`（含糊英文表达） → 歧义，展示选项
- `/lathe 用 Go 构建 Raft`（明确的主题描述） → 明确为**生成新教程**
- `/lathe 续写 raft-go` 或 `/lathe extend raft-go`（含关键词） → 明确为**续写**
- `/lathe 验证 raft-go` 或 `/lathe verify raft-go`（含关键词） → 明确为**验证**

用户选择后，读取对应模块文件执行。如果用户选择的意图需要额外参数（如续写需要 slug），在读取模块前补充询问。

## 共享配置：metadata.json 结构

每个教程的 `<TUTORIALS_DIR>/<slug>/metadata.json` 遵循此结构：

```json
{
  "slug": "<kebab-case 短名称>",
  "title": "<教程标题>",
  "topic": "<用户的原始主题描述>",
  "created": "<ISO 8601 时间戳>",
  "status": "unverified",
  "tags": ["<标签1>", "<标签2>"],
  "parts": ["part-01.md"],
  "language": "<教程正文语言，如 zh-CN、en>",
  "repo": "<远程 URL 或空>",
  "repo_branch": "<分支或空>",
  "local_project_path": "<本地项目路径或空>",
  "tools": [{"name": "<工具>", "version": "<版本>"}],
  "sources": ["<url1>", "<url2>"],
  "voice": "plainspoken",
  "model": "<AI 模型名称，如 QoderWork>"
}
```

> 旧教程的 metadata 可能含有已废弃的 `pending_part` 字段——读取时忽略，写回时不必保留。

## 共享配置：教程语言

教程**正文语言**与触发语言解耦：

1. **生成时确定**：默认跟随用户当前对话语言；用户显式指定（如 "用英文写"）时以指定为准。将结果记录到 `metadata.json` 的 `language` 字段。
2. **续写/提问时沿用**：读取 `language` 字段并保持一致，不因用户切换对话语言而漂移。
3. **旧教程缺失该字段时**：按已有部分的实际语言推断，并在下次写 metadata 时补上。

## 共享配置：教程结构

每篇教程（或系列的每个部分）遵循此结构，但章节*标题*必须特定于领域：

```
# [标题]

[引子——2 到 4 段]

## 你将构建什么
一段话。用控制性示例命名的具体最终状态。

## 前置条件
列表：需要安装的工具、读者应大致了解的知识。

## [具体的章节标题——命名本章节创造的东西]
为什么需要它。小块代码，每块标注插入位置。旁白或设计笔记。

## 检查点
> [!PREDICT]
> 运行之前：你预期会看到什么输出？
**运行以下命令验证：** + 预期输出 + 可能的错误

## 接下来
寄语或前向钩子：最后部分邀请读者走出铺设的道路；非最后部分用一段话命名未来部分将解答的未决问题，倾向悬念感。随后用平实散文（不是标注）提出**结尾反思**：挑本部分最重要的一个设计决策，要求读者解释*为什么*而非*是什么*，不替他们回答。

## 练习（编号，3-5 个，每个具体到 30 秒内可开始）

## 来源（编号，格式：[标题](url) —— 一句话说明；超过约 5 条则分组）
```

每个部分以 *"在本部分结束时，你将拥有 [具体的东西]"* 开头，以检查点结尾。每个部分独立成篇。本节是教程结构的**单一事实源**；references/generate.md 只补充具体写法细节，不重新定义结构。

## 共享配置：标注类型

| 标注 | 用途 | 频率 |
|------|------|------|
| `> [!PREDICT]` | 检查点前的预测提示 | 每部分 1 次 |
| `> [!RECALL]` | 第 N≥2 部分顶部的间隔检索 | 每部分 1 次 |
| `> [!HEADS-UP]` | 即将踩坑的警告 | 按需，≤2 次/部分 |
| `> [!ASIDE]` | 词源、故事等旁白（1-2 句） | 按需 |
| `> [!DESIGN-NOTE]` | 多段落的"为什么"讨论 | 章节末尾 |
| `> [!UNVERIFIED]` | 无法确认的关键声明 | 诚实标记 |
| `> [!NOTE]` | 中性补充信息 | 按需 |
| `> [!TIP]` | 便捷技巧 | 按需 |

以上表中的频率列是硬上限。在上限内仍须宁缺毋滥：非教学性标注（HEADS-UP、ASIDE、NOTE、TIP）每部分**合计不超过 3 个**。

## 共享配置：风格标杆

标杆作者体系定义 Lathe 教程的质量基准线（五位核心标杆 + 五位扩展参考），并按主题场景给出主锚点选择。**完整作者表与场景侧重表见 [references/style-benchmarks.md](references/style-benchmarks.md)——仅在生成/续写教程、需要确定写作节奏与结构时读取；提问、标签、列表等轻量操作无需加载。**

## 共享配置：内置 voice

- **plainspoken**（默认）—— 直接、精确、不虚构人格。平实散文，信任读者。
- **companion** —— 键盘旁温暖而幽默的朋友。第一人称，有主见，破折号式自我纠正。
- **craftsman** —— 从容的匠人师傅。散文式构建，长短句交替，精心打磨的类比贯穿段落，克制的机智。

以上仅为一行索引；**完整规范（风格细节与反模式清单）的单一事实源在 [references/voice.md](references/voice.md) "内置 voice"一节**。应用 voice 时（生成/续写），仅当用户指定了非默认 voice 才读取 references/voice.md；默认 plainspoken 直接使用上方索引即可。

voice 仅控制语气；绝不改变准确性、研究、引用、验证规则。

## 不变量（始终生效，与 voice 无关）

这些规则在**每种** voice 和**每个**操作中都成立：

- **有据可依或标记——绝不虚张声势。** 每个关键声明有来源或 `[!UNVERIFIED]` 标注。
- **在陷阱发生前命名它们。** 使用 `[!HEADS-UP]` 警告即将踩坑的内容。
- **先展示错误版本，再展示修复。** 演示诱人但有问题的方法，然后展示修复。
- **定义术语，然后给出内行名称。** 粗体标记规范术语，同义词紧随。
- **使用领域真实名称。绝不用 `foo`/`bar`。**
- **每次都用具体数字。** "48000"比"很快"更有记忆点。
- **指定异常输入。** 引入处理器后，立即回答"几乎匹配但不完全匹配的输入会怎样？"
- **在关键事实首次出现时内联引用。** 深度链接到具体章节，而非首页。

## 保持会话

无论执行哪个操作，完成后不要结束会话。保持可用以回答后续问题。你是他们在该主题上的专家向导。
