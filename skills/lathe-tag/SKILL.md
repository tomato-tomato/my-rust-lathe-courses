---
name: lathe-tag
description: 在会话中为已存储的 Lathe 教程选择或补充搜索标签。当用户调用 /lathe-tag 并附带 slug 时使用，如 "/lathe-tag digital-synth-zig"，用于选择适合搜索和标签筛选的标签，或为没有标签的教程补充标签。
---

# Lathe — 教程标签管理

选择使已存储教程可被发现的标签。由 `/lathe-tag <slug>` 触发。

> **路径解析：** 本技能中所有 `<TUTORIALS_DIR>` 均从 `~/.lathe/config.json` 的 `tutorials_base_path` 字段读取。如果配置文件不存在，使用默认值 `~/others/lathe_tutorials`。

## 协议

1. **阅读教程**，路径为 `<TUTORIALS_DIR>/<slug>/` —— 读取 `metadata.json` 并浏览各部分，理解它实际教授的内容。

2. **选择 2-5 个小写、可复用的标签。** 在适用时涵盖：
   - **语言/运行时** —— `zig`、`rust`、`go`；
   - **领域** —— `audio`、`compilers`、`databases`；
   - **核心技术** —— `parsing`、`dsp`、`concurrency`。

   优先选择能与其他教程自然分组的短标签，而非过于具体的一次性标签。

3. **持久化标签。** 读取现有的 `metadata.json`，更新 `tags` 数组，然后写回：
   - **设置整个标签列表**（选择或替换标签）：完全替换 `tags` 数组。
   - **添加标签：** 追加到现有数组，然后去重。
   - **移除标签：** 从现有数组中删除。

   `metadata.json` 中的 `tags` 字段是小写字符串的 JSON 数组：
   ```json
   {
     "tags": ["zig", "audio", "dsp"]
   }
   ```

4. **向用户报告最终的标签集合。**

## 边界

- 仅修改 `metadata.json` 中的 `tags` 字段。绝不编辑部分文件或其他元数据字段。
- 不涉及生成、验证或续写——仅处理标签。
