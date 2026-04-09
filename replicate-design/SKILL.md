---
name: replicate-design
description: 将 App 截图高保真复刻到 Pencil (.pen) 设计文件中，使用设计师 + 审核员双 Agent 迭代循环。当用户说"复刻截图"、"一比一还原"、"把截图画到Pencil"、"replicate design"，或提供截图要求在 Pencil 中还原时触发。
---

# 截图复刻设计

通过双 Agent 迭代循环，将 App 截图在 Pencil 中高保真还原：
- **设计师 Agent** 在 Pencil 中绘制/修改设计稿
- **审核员 Agent** 将结果与参考截图对比，输出差异报告
- 循环持续直至通过审核（或达到最大 5 轮迭代）

## 技能文件

- `agents/designer.md` — 设计师 Agent 角色定义
- `agents/reviewer.md` — 审核员 Agent 角色定义
- `references/design-checklist.md` — 审核维度与严重程度规则

## 准备工作

开始前读取以下文件：
1. `agents/designer.md`
2. `agents/reviewer.md`
3. `references/design-checklist.md`

## 输入

从用户消息中获取：
- **reference_screenshot_path**：参考 App 截图的路径（必需）
- 当前 Pencil 文件通过 `get_editor_state` 自动检测

## 编排循环

```
max_iterations = 5
iteration = 1
main_frame_node_id = null
passed = false

WHILE iteration <= max_iterations AND NOT passed:

  1. [向用户报告] "⏳ 第 {iteration} 轮 — 设计师 Agent 开始{'绘制' if iteration==1 else '修改'}..."

  2. 启动设计师 Agent（子 Agent，通用类型）
     → 读取 agents/designer.md 获取完整指令
     → 传入：reference_screenshot_path, pen_file_path, iteration, 
               target_node_id (iteration>1 时), feedback (iteration>1 时)
     → 等待完成
     → 从响应中提取 main_frame_node_id

  3. [向用户报告] "🔍 第 {iteration} 轮验收 — 截图并对比..."

  4. 获取设计截图：调用 mcp__pencil__get_screenshot
     filePath = pen_file_path, nodeId = main_frame_node_id
     → 保存返回的截图供审核员 Agent 使用

  5. 启动审核员 Agent（子 Agent，通用类型）
     → 读取 agents/reviewer.md 获取完整指令
     → 传入：reference_screenshot_path, design_screenshot_path, iteration,
               checklist_path (references/design-checklist.md)
     → 等待完成
     → 解析 JSON 报告：passed, differences, summary

  6. [向用户报告] 进度更新（见下方"进度报告"）

  7. 若 passed：
       → 退出循环，报告成功

  8. 若 iteration == max_iterations：
       → 退出循环，报告已达最大迭代次数

  9. iteration += 1
     feedback = 审核员报告中的 differences
     继续循环

END LOOP
```

## 进度报告

每轮审核员 Agent 完成后，向用户报告：

**未通过时：**
```
🔄 第 {N} 轮验收：未通过
相似度：{overall_similarity}
差异项：{high_count} 高 / {medium_count} 中 / {low_count} 低
概述：{summary}

正在进入第 {N+1} 轮修改...
```

**通过时：**
```
✅ 第 {N} 轮验收：通过！
相似度：{overall_similarity}
复刻完成 🎉 设计稿 nodeId：{main_frame_node_id}
```

**达到最大迭代次数仍未通过时：**
```
⚠️ 已达最大迭代次数（5轮），终止循环。
最终相似度：{overall_similarity}
剩余差异：{high_count} 高 / {medium_count} 中 / {low_count} 低
设计稿 nodeId：{main_frame_node_id}

建议：可手动在 Pencil 中继续微调，或告知我重点修改方向。
```

## 重要说明

### 获取 pen_file_path

在启动设计师 Agent 之前，先调用 `mcp__pencil__get_editor_state`（`include_schema: true`），从返回结果中提取当前活跃文件路径。

### 设计师 Agent 提示模板

启动设计师 Agent 子任务时，使用以下提示结构：

```
你是一个设计复刻任务的设计师 Agent。读取
~/.claude/skills/replicate-design/agents/designer.md 获取完整指令。

参数：
- reference_screenshot_path: {reference_screenshot_path}
- pen_file_path: {pen_file_path}
- iteration: {iteration}
{如果 iteration > 1: - target_node_id: {main_frame_node_id}}
{如果 iteration > 1: - feedback: {JSON 格式的差异数组}}

严格按照 designer.md 中的指令执行。完成后报告：
"DESIGN COMPLETE - nodeId: {the_nodeId}"
```

### 审核员 Agent 提示模板

启动审核员 Agent 子任务时，使用以下提示结构：

```
你是一个设计复刻任务的审核员 Agent。读取
~/.claude/skills/replicate-design/agents/reviewer.md 获取完整指令。

参数：
- reference_screenshot_path: {reference_screenshot_path}
- design_screenshot_path: {design_screenshot_path}
- iteration: {iteration}
- checklist_path: ~/.claude/skills/replicate-design/references/design-checklist.md

严格按照 reviewer.md 中的指令执行。输出完整的 JSON 报告。
```

### 提取设计师 Agent 的 nodeId

设计师 Agent 会在响应中包含 "DESIGN COMPLETE - nodeId: {id}"。解析此内容以获取截图和下一轮迭代所需的 nodeId。

### 审核员的截图路径

`mcp__pencil__get_screenshot` 返回截图图片。为让审核员 Agent 访问，推荐使用导出方式：

```
mcp__pencil__export_nodes({
  filePath: pen_file_path,
  nodeIds: [main_frame_node_id],
  outputDir: "/tmp/replicate-design-exports/",
  format: "png"
})
→ 返回导出 PNG 的绝对路径
→ 将该路径作为 design_screenshot_path 传给审核员 Agent
```
