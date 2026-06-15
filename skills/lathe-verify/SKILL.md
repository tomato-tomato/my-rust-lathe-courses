---
name: lathe-verify
description: 在会话中通过在全新的临时目录中端到端执行来验证已存储的 Lathe 教程是否确实可用。当用户调用 /lathe-verify 并附带 slug 时使用，如 "/lathe-verify digital-synth-zig"。
---

# Lathe — 验证教程

像读者一样在一个临时目录中按照已存储的教程逐步执行，并记录它是否确实可用。由 `/lathe-verify <slug>` 触发。

本技能**对教程内容是只读的**：绝不编辑部分。唯一的写入是更新 `verify-result.json`。

> **路径解析：** 本技能中所有 `<TUTORIALS_DIR>` 均从 `~/.lathe/config.json` 的 `tutorials_base_path` 字段读取。如果配置文件不存在，使用默认值 `~/others/lathe_tutorials`。

## 协议

1. **阅读教程元数据**，路径为 `<TUTORIALS_DIR>/<slug>/metadata.json`，获取部分列表、工具和当前状态。

2. **标记为进行中。** 在 `<TUTORIALS_DIR>/<slug>/` 中写入 `verify-result.json`：
   ```json
   {
     "status": "verifying",
     "checked_at": "<当前 ISO 8601 时间戳>"
   }
   ```

3. **创建全新的临时目录并在其中工作：**
   ```bash
   cd "$(mktemp -d)"
   ```
   教程要求读者创建的所有内容都在这里进行，不在用户的项目中。

4. **按顺序跟随每个部分。** 从 `part-01` 开始阅读 `<TUTORIALS_DIR>/<slug>/part-NN.md`。安装前置条件，按顺序创建文件并粘贴代码块，然后运行 `## 检查点` 命令并与声明的预期输出比较。
   - **跳过教学性和溯源性标注** —— `> [!PREDICT]`、`> [!RECALL]` 和 `> [!UNVERIFIED]` 不是可验证的步骤。
   - 将检查点命令和代码块视为可执行面。

5. **通过更新** `<TUTORIALS_DIR>/<slug>/verify-result.json` **记录结果**：

   - **一切正常：**
     ```json
     {
       "status": "verified",
       "checked_at": "<ISO 8601 时间戳>"
     }
     ```

   - **缺少必需工具**（这不是失败——意味着"无法在此处运行"）：
     ```json
     {
       "status": "skipped",
       "checked_at": "<ISO 8601 时间戳>",
       "error": "未安装必需工具：<工具名>"
     }
     ```

   - **确实出现问题**（输出错误、代码无法编译、步骤自相矛盾）：
     ```json
     {
       "status": "failed",
       "part": "part-NN.md",
       "failed_step": 2,
       "error": "<错误信息或不匹配的输出>",
       "checked_at": "<ISO 8601 时间戳>"
     }
     ```

6. **向用户报告**发生了什么——验证通过、跳过（以及缺少哪个工具），或者确切失败在哪里。

## 边界

- **对教程部分只读。** 绝不编辑 `part-NN.md`。唯一的写入是 `verify-result.json`。
- **跳过 ≠ 失败。** 缺少工具链是 `skipped`。将 `failed` 保留给教程确实有问题的情况。
