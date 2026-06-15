# Lathe 技能在QoderWork中的使用教程

Lathe 是一套面向 LLM 编码助手的**动手实践型教程生成技能体系**。通过 6 个协同工作的技能（Skill），你可以按需生成高质量的技术教程，并逐步续写、验证、提问和管理。

> 本教程适用于 QoderWork / Qoder Code 环境。技能以 SKILL.md 格式提供，兼容所有支持 Agent Skills 的 LLM 编码助手。

## 目录

- [快速开始](#快速开始)
- [技能一览](#技能一览)
- [存储路径配置](#存储路径配置)
- [/lathe — 生成教程](#lathe--生成教程)
- [/lathe-extend — 续写教程](#lathe-extend--续写教程)
- [/lathe-verify — 验证教程](#lathe-verify--验证教程)
- [/lathe-ask — 提问教程内容](#lathe-ask--提问教程内容)
- [/lathe-tag — 管理教程标签](#lathe-tag--管理教程标签)
- [/lathe-voice — 创建自定义语气](#lathe-voice--创建自定义语气)
- [完整工作流示例](#完整工作流示例)
- [教程文件结构参考](#教程文件结构参考)
- [常见问题](#常见问题)

---

## 快速开始

### 1. 安装技能

将 `qoderwork-skills-nocli/` 目录下的 6 个技能文件夹复制到 QoderWork 的技能目录：

```bash
# 复制到用户级技能目录（所有项目可用）
cp -r qoderwork-skills-nocli/* ~/.qoderwork/skills/

# 或复制到项目级技能目录（仅当前项目可用）
cp -r qoderwork-skills-nocli/* .qoderwork/skills/
```

### 2. 验证安装

在 QoderWork 中输入 `/lathe`，如果技能正确加载，AI 会询问你的经验水平和教程主题。

### 3. 生成第一个教程

```
/lathe 用 Go 构建一个 Raft 共识算法实现
```

AI 会引导你完成：确认经验水平 → 锁定工具版本 → 研究主题 → 生成第一部分 → 自动存储。

---

## 技能一览

| 技能 | 命令 | 功能 | 写入文件 |
|------|------|------|----------|
| **lathe** | `/lathe <主题>` | 生成教程的第一部分 | `part-01.md`、`metadata.json` |
| **lathe-extend** | `/lathe-extend <slug> [指导]` | 为已有教程添加下一部分 | `part-NN.md`、更新 `metadata.json` |
| **lathe-verify** | `/lathe-verify <slug>` | 端到端验证教程是否可用 | `verify-result.json` |
| **lathe-ask** | `/lathe-ask <slug> <part-NN.md>` | 回答关于教程某部分的问题 | 无（只读） |
| **lathe-tag** | `/lathe-tag <slug>` | 管理教程的搜索标签 | 更新 `metadata.json` |
| **lathe-voice** | `/lathe-voice [名称]` | 创建自定义写作语气预设 | `voices/<名称>.md` |

---

## 存储路径配置

所有教程存储在同一个基础目录下，路径记录在配置文件中：

**配置文件位置：** `~/.lathe/config.json`

```json
{
  "tutorials_base_path": "~/others/lathe_tutorials"
}
```

### 首次使用

首次调用 `/lathe` 时，技能会检测配置文件是否存在：

- **不存在：** 询问你是否使用默认路径 `~/others/lathe_tutorials/`，或指定新路径。确认后自动创建配置文件。
- **已存在：** 静默读取路径，直接使用。

### 修改存储路径

直接编辑配置文件即可：

```bash
# 查看当前配置
cat ~/.lathe/config.json

# 修改路径
echo '{"tutorials_base_path": "~/my-tutorials"}' > ~/.lathe/config.json
```

> 修改路径后，已有教程不会自动迁移。如需访问旧教程，请将旧目录内容复制到新位置。

### 默认目录结构

```
~/others/lathe_tutorials/          ← 教程基础目录
├── <slug>/                        ← 每个教程一个子目录
│   ├── metadata.json              ← 教程元数据
│   ├── part-01.md                 ← 第一部分
│   ├── part-02.md                 ← 第二部分（由 /lathe-extend 添加）
│   └── verify-result.json         ← 验证结果（由 /lathe-verify 生成）
└── voices/                        ← 自定义语气预设
    └── <名称>.md                  ← voice 规范文件
```

---

## /lathe — 生成教程

### 功能

为任意编程主题生成动手实践型教程的第一部分。这是整个技能体系的核心入口。

### 调用方式

```
/lathe <主题描述>
```

**示例：**

```
/lathe 用 Zig 构建数字合成器
/lathe 用 Rust 实现一个键值存储
/lathe 用 Go 构建 Raft 共识算法
```

### 工作流程

调用后，AI 会按以下步骤引导你：

**1. 解析存储路径**
自动从 `~/.lathe/config.json` 读取教程存储位置。首次使用会询问。

**2. 确认经验水平**

> "你在这个领域的经验水平如何——初学者、有一定了解、还是在相关领域有经验？"

回答后，AI 会据此调整教程的深度和解释密度。

**3. 澄清主题（如需要）**
如果主题描述比较宽泛（如"用 Go 构建网络服务"），AI 会问一个澄清问题来确定范围。如果已经足够具体，则跳过。

**4. 锁定仓库和工具链版本**
AI 会询问：
- 教程是否针对特定的代码仓库（提供 URL 和分支）
- 需要锁定哪些工具/语言的版本（如 `Go 1.22`、`Zig 0.13.0`）

你可以提议版本，AI 会确认：

> "我将基于 **Go 1.22** 来写——可以吗，还是你想用其他版本？"

**5. 研究主题**
AI 会通过网络搜索查阅 3-8 个权威来源（官方文档、论文、源代码等），确保教程内容基于真实资料而非模型记忆。

> 如果当前会话没有网络工具，AI 会提前告知，并在关键声明处标记 `[!UNVERIFIED]`。

**6. 生成第一部分**
AI 在内部完成预检（选择控制性示例、确定章节结构等）后，撰写 `part-01.md`。

**7. 存储教程**
将 `part-01.md` 和 `metadata.json` 写入教程目录，然后告知你：

> **教程已保存**到 `~/others/lathe_tutorials/raft-go/`。
> 这是第一部分。要添加更多部分，调用 `/lathe-extend raft-go`。
> 验证是可选的：调用 `/lathe-verify raft-go`。
> 要对某个部分提问，调用 `/lathe-ask raft-go part-01.md`。

### 教程质量标准

每篇教程遵循以下标准：

- **控制性示例：** 贯穿始终的具体产物（如"一个 4 声部减法合成器"），不使用 `foo`/`bar`
- **具体数字：** 采样率、缓冲区大小、页面大小等，避免模糊的"很快"/"很大"
- **内联引用：** 关键事实旁直接附上来源链接
- **检查点：** 每个部分末尾有可执行的验证命令和预期输出
- **标注系统：** 使用 `> [!PREDICT]`、`> [!RECALL]`、`> [!HEADS-UP]` 等标注增强学习效果

---

## /lathe-extend — 续写教程

### 功能

为已有教程添加下一部分。新部分会继承教程的控制性示例、数字、语气和工具版本。

### 调用方式

```
/lathe-extend <slug> [指导...]
```

**示例：**

```
/lathe-extend raft-go
/lathe-extend raft-go 这一部分实现日志复制
/lathe-extend digital-synth-zig 添加低通滤波器
```

如果不提供指导，AI 会根据上一部分末尾的"接下来"方向自动延续。

### 工作流程

1. **吸收现有教程** — 阅读所有已有部分和 `metadata.json`，理解控制性示例、固定数字、voice 等上下文
2. **研究新内容** — 与 `/lathe` 相同的研究纪律：查阅权威来源，记录 URL
3. **确定部分编号** — 自动递增（如已有 `part-01.md`，下一个是 `part-02.md`）
4. **写入新部分** — 遵循完整的教程结构，以 `> [!RECALL]` 间隔检索开头
5. **更新 metadata.json** — 追加新部分到 `parts` 数组，追加新来源
6. **报告完成** — 告知你如何查看和验证

### 注意事项

- 每次只写一个部分，不批量生成
- 不修改已有的部分文件
- 继承相同的工具版本，不静默升级
- 如果读者想迁移到新版本，建议创建新教程而非在原教程中漂移

---

## /lathe-verify — 验证教程

### 功能

在全新的临时目录中，像真实读者一样逐步执行教程，检查每步是否确实可用。

### 调用方式

```
/lathe-verify <slug>
```

**示例：**

```
/lathe-verify raft-go
```

### 工作流程

1. **读取元数据** — 获取部分列表和所需工具
2. **标记为进行中** — 写入 `verify-result.json`，状态设为 `verifying`
3. **创建临时目录** — 在 `mktemp -d` 生成的干净环境中操作
4. **逐步执行** — 从 `part-01` 开始，安装前置条件、创建文件、粘贴代码、运行检查点命令
5. **记录结果** — 更新 `verify-result.json`

### 验证结果

验证完成后会产生三种状态之一：

| 状态 | 含义 | 场景 |
|------|------|------|
| `verified` | 验证通过 | 所有检查点输出与教程声明一致 |
| `skipped` | 跳过 | 缺少必需工具（如未安装 Zig 编译器） |
| `failed` | 验证失败 | 代码无法编译、输出不匹配、步骤矛盾 |

**验证通过示例：**

```json
{
  "status": "verified",
  "checked_at": "2025-01-15T10:30:00Z"
}
```

**验证失败示例：**

```json
{
  "status": "failed",
  "part": "part-02.md",
  "failed_step": 2,
  "error": "编译错误：未定义的变量 `filter_cutoff`",
  "checked_at": "2025-01-15T10:30:00Z"
}
```

### 注意事项

- 验证是**只读操作**，不会修改教程的任何部分文件
- `skipped` 不等于 `failed`——缺少工具是环境问题，不是教程问题
- 验证前需要本地安装教程所需的工具链

---

## /lathe-ask — 提问教程内容

### 功能

针对教程的特定部分提出问题，AI 会基于该教程的具体内容回答，而非泛泛地重新讲解主题。

### 调用方式

```
/lathe-ask <slug> <part-NN.md>
<你的问题>
```

**示例：**

```
/lathe-ask raft-go part-01.md
为什么用环形缓冲区而不是 channel？

/lathe-ask digital-synth-zig part-02.md
如果滤波器截止频率超过奈奎斯特频率会怎样？
```

### 回答原则

- **扎根于本教程** — 使用教程中相同的示例、数字和术语回答
- **指向具体位置** — 优先"看 §3 中的 `process_buffer` 循环"而非从头推导
- **对缺口诚实** — 如果问题暴露了教程的错误或含糊之处，会直接说明
- **对话式** — 保持会话以回答后续追问

### 注意事项

- 纯粹的只读操作，不修改任何文件
- 不会触发验证、续写或打标签

---

## /lathe-tag — 管理教程标签

### 功能

为教程选择或更新搜索标签，使教程更容易被发现和分类。

### 调用方式

```
/lathe-tag <slug>
```

**示例：**

```
/lathe-tag raft-go
```

### 标签规范

AI 会选择 2-5 个小写、可复用的标签，涵盖以下维度：

| 维度 | 示例 |
|------|------|
| **语言/运行时** | `go`、`rust`、`zig`、`python` |
| **领域** | `compilers`、`databases`、`audio`、`networking` |
| **核心技术** | `consensus`、`dsp`、`parsing`、`concurrency` |

优先选择能与其他教程自然分组的通用标签，避免过于具体的一次性标签。

### 操作方式

- **设置标签：** 完全替换 `tags` 数组
- **添加标签：** 追加到现有数组并去重
- **移除标签：** 从现有数组中删除

标签保存在 `metadata.json` 的 `tags` 字段中。

---

## /lathe-voice — 创建自定义语气

### 功能

创建自定义的写作语气（Voice）预设，控制教程的文风、视角和表达方式。

### 内置 Voice

| Voice | 特点 | 适用场景 |
|-------|------|----------|
| **plainspoken**（默认） | 直接、精确、不虚构人格。平实散文，信任读者 | 大多数技术教程 |
| **companion** | 温暖、幽默、第一人称。键盘旁的老朋友 | 入门教程、需要亲和力的场景 |

### 调用方式

```
/lathe-voice [名称]
```

**示例：**

```
/lathe-voice terse
/lathe-voice field-notes
```

### 创建流程

1. **选择名称** — 提供或确认一个简短的 slug（如 `terse`、`socratic`）
2. **风格访谈** — AI 会询问：
   - 正式 ↔ 随意？密集 ↔ 疏朗？冷峻 ↔ 温暖？
   - 人称：第一人称、协作式还是非人称？
   - 幽默程度：无、克制还是活泼？
   - 这种 voice 绝不应该像什么？
3. **起草规范** — 生成包含三个章节的 markdown 文件：风格与调性、避免、校准对比
4. **展示并确认** — 展示完整草稿，迭代直到你批准
5. **持久化** — 写入 `<TUTORIALS_DIR>/voices/<名称>.md`

### 使用自定义 Voice

创建后，在生成教程时指定 voice：

```
/lathe 用 Rust 构建解析器，使用 terse voice
```

Voice 名称会记录在 `metadata.json` 中，`/lathe-extend` 会自动继承。

### 管理 Voice

- **查看已有 voice：** 检查 `<TUTORIALS_DIR>/voices/` 目录
- **删除 voice：** 直接删除对应的 `.md` 文件
- **内置 voice 不可覆盖：** `plainspoken` 和 `companion` 保留为内置

### 安全护栏

技能会拒绝创建以下 voice：
- 冒充真实人物的 voice
- 虚构资历或权威背书的 voice
- 试图将 AI 生成内容伪装为人类撰写的 voice

---

## 完整工作流示例

以下是一个从零开始、完整使用所有技能的示例流程：

### 第一步：配置存储路径（首次使用）

```
用户：/lathe 用 Go 构建 Raft 共识算法

AI：这是首次使用 Lathe。教程默认存储在 ~/others/lathe_tutorials/，要使用其他位置吗？

用户：默认就好

AI：已创建配置。教程将存储在 ~/others/lathe_tutorials/。
    你在这个领域的经验水平如何——初学者、有一定了解、还是在相关领域有经验？

用户：有一定了解
```

### 第二步：锁定版本并生成

```
AI：我将基于 Go 1.22 来写——可以吗？这个教程是独立教程还是针对特定仓库？

用户：Go 1.22 可以，独立教程

AI：（研究主题中...查阅了 Raft 论文、Go 标准库文档等 6 个来源）
    （生成 part-01.md 和 metadata.json）

    教程已保存到 ~/others/lathe_tutorials/raft-go/。
    要添加更多部分，调用 /lathe-extend raft-go。
```

### 第三步：续写更多部分

```
用户：/lathe-extend raft-go 这一部分实现 Leader 选举

AI：（阅读已有教程，研究 Leader 选举，生成 part-02.md）
    已添加 part-02.md。可以调用 /lathe-verify raft-go 验证。
```

### 第四步：验证教程

```
用户：/lathe-verify raft-go

AI：（在临时目录中逐步执行，运行检查点命令）
    验证结果：
    - part-01.md：✓ 通过
    - part-02.md：✓ 通过
    教程验证全部通过！
```

### 第五步：提问和打标签

```
用户：/lathe-ask raft-go part-02.md
     为什么选举超时要随机化？

AI：在 part-02.md 的 "选举超时" 章节中，随机化是为了防止
    两个候选者同时发起选举导致的活锁...

用户：/lathe-tag raft-go

AI：已为教程设置标签：["go", "distributed-systems", "consensus", "raft"]
```

### 第六步（可选）：创建自定义语气

```
用户：/lathe-voice socratic

AI：（进行风格访谈，起草 voice 规范）
    这是 socratic voice 的草稿...是否满意？

用户：很好

AI：已保存到 ~/others/lathe_tutorials/voices/socratic.md。
    下次生成教程时可以指定：/lathe <主题>，使用 socratic voice
```

---

## 教程文件结构参考

### metadata.json

每个教程的元数据文件，记录教程的完整信息：

```json
{
  "slug": "raft-go",
  "title": "用 Go 从零实现 Raft 共识算法",
  "topic": "用 Go 构建 Raft 共识算法实现",
  "created": "2025-01-15T10:00:00Z",
  "status": "unverified",
  "tags": ["go", "distributed-systems", "consensus", "raft"],
  "parts": ["part-01.md", "part-02.md"],
  "pending_part": "",
  "repo": "",
  "repo_branch": "",
  "tools": [
    {"name": "Go", "version": "1.22"}
  ],
  "sources": [
    "https://raft.github.io/raft.pdf",
    "https://pkg.go.dev/sync"
  ],
  "voice": "plainspoken",
  "model": "DeepSeek V4"
}
```

**字段说明：**

| 字段 | 说明 |
|------|------|
| `slug` | 教程的唯一标识符（URL 友好的短名称） |
| `title` | 教程标题（来自 part-01.md 的 H1） |
| `topic` | 用户的原始主题描述 |
| `created` | 创建时间（ISO 8601） |
| `status` | 状态：`unverified` / `verified` / `failed` |
| `tags` | 标签数组（由 `/lathe-tag` 管理） |
| `parts` | 已有的部分文件列表 |
| `repo` | 关联的代码仓库 URL（独立教程为空） |
| `tools` | 锁定的工具链及版本 |
| `sources` | 研究期间查阅的来源 URL（去重） |
| `voice` | 使用的写作语气 |
| `model` | 生成最后部分的 AI 模型 |

### 部分文件（part-NN.md）

每个部分遵循统一结构：

```markdown
# [标题]

[引子段落]

## 你将构建什么
[控制性示例的具体描述]

## 前置条件
[需要安装的工具和前置知识]

## [具体章节标题]
[教学内容，含代码块、旁白、设计笔记]

## 检查点
> [!PREDICT]
> 运行之前：你预期会看到什么输出？

**运行以下命令验证：**
```bash
<命令>
```
预期输出：...
**可能的错误：** ...

## 接下来
[未决问题和前进方向]

## 练习
1. [具体练习]
2. [具体练习]

## 来源
1. [标题](url) —— 说明
```

### verify-result.json

验证结果文件，记录验证状态：

```json
{
  "status": "verified",
  "checked_at": "2025-01-15T10:30:00Z"
}
```

### 标注类型速查

| 标注 | 用途 | 使用频率 |
|------|------|----------|
| `> [!PREDICT]` | 检查点前的预测提示 | 每个部分 1 次 |
| `> [!RECALL]` | 第 N≥2 部分顶部的间隔检索 | 每个部分 1 次 |
| `> [!HEADS-UP]` | 即将踩坑的警告 | 按需，每部分≤2 次 |
| `> [!ASIDE]` | 词源、故事等旁白 | 按需 |
| `> [!DESIGN-NOTE]` | 多段落的"为什么"讨论 | 放在章节末尾 |
| `> [!UNVERIFIED]` | 无法确认的关键声明 | 诚实标记 |
| `> [!NOTE]` | 中性补充信息 | 按需 |
| `> [!TIP]` | 便捷技巧 | 按需 |

---

## 常见问题

### Q：教程可以跨多个会话续写吗？

可以。每次调用 `/lathe-extend` 时，AI 会从 `metadata.json` 和已有部分文件中读取完整上下文，因此即使换了会话甚至换了 AI 模型，也能无缝续写。

### Q：可以在不同的 AI 工具之间切换使用吗？

可以。技能文件（SKILL.md）是标准的 Markdown 格式，兼容 Claude Code、QoderWork、Qwen Code 等支持 Agent Skills 的工具。只要将技能文件安装到对应工具的技能目录即可。

### Q：验证失败了怎么办？

查看 `verify-result.json` 中的 `error` 和 `failed_step` 字段定位问题。常见原因：
- 代码块中的代码不完整或有遗漏
- 检查点命令的预期输出与实际不符
- 前置条件遗漏了某个工具

修复教程后重新运行 `/lathe-verify`。

### Q：如何修改已生成教程的内容？

直接在编辑器中修改 `part-NN.md` 文件即可。修改后建议重新运行 `/lathe-verify` 确认可用性。

### Q：Voice 会影响教程的准确性吗？

不会。Voice 仅控制语气和风格（如是否使用第一人称、幽默程度等）。教程的准确性、研究纪律、引用规范和验证规则是不变量，不受 Voice 影响。

### Q：如何查看所有已生成的教程？

浏览教程基础目录（默认为 `~/others/lathe_tutorials/`），每个子目录就是一个教程。也可以通过标签筛选：

```bash
# 查找包含特定标签的教程
grep -rl '"compilers"' ~/others/lathe_tutorials/*/metadata.json
```
