# Skills Registry

[AI Workspace Dashboard](https://github.com/DamonAmber/ai-workspace-dashboard) 的公开技能注册中心。在 Dashboard 中浏览技能，一键安装到 Claude Code、Codex 或 Kiro。

## 可用技能

| 技能 | 描述 | 标签 |
|------|------|------|
| **replicate-design** | 将 App 截图高保真复刻到 Pencil (.pen) 设计文件，使用设计师+审核员双 Agent 迭代循环 | design, pencil, multi-agent, ui |
| **mijia-sentiment** | 米家App舆情分析周报生成工具，自动筛选负面反馈、27类问题分类、生成Excel周报 | sentiment, analytics, excel, mijia |
| **weekly-report-summary** | 米家App周报简述生成工具，从飞书周报明细按业务领域提取关键进展生成精炼汇总 | weekly-report, feishu, summary, mijia |
| **knowledge-distiller** | 将领域原始材料蒸馏提炼为结构化知识包，可通过 Dashboard 分享安装 | knowledge, distill, knowledge-pack, mijia |
| **feishu** | 通过 CLI 管控飞书全量资源：文档、知识库、云盘、多维表格、表格、权限、日历日程、任务待办 | feishu, lark, docs, bitable, calendar, task, cli |

## 使用方法

### 通过 AI Workspace Dashboard

1. 打开 Dashboard → Skills 标签 → Marketplace 区域
2. 默认已配置本仓库，点击 **刷新** 即可加载技能列表
3. 点击 **安装**，选择目标工具（Claude Code / Codex / Kiro），确认即可

### 手动安装

```bash
git clone https://github.com/DamonAmber/skills-registry.git
cp -R skills-registry/replicate-design ~/.claude/skills/
```

## 添加新技能

1. 在仓库根目录创建以技能名命名的文件夹
2. 至少添加一个 `SKILL.md` 文件，包含 YAML 前置元数据：
   ```yaml
   ---
   name: my-skill
   description: 技能功能描述及触发条件
   ---
   ```
3. 添加子文件（agents/、references/、scripts/）
4. 更新 `registry.json` — 在 `skills` 数组中添加条目，`files` 列出所有文件路径
5. 上传到腾讯云 COS：`node tools/ai-workspace-dashboard/scripts/upload-registry-to-cos.js --type skills --dir tools/skills-registry`
6. 提交到 Git（版本管理）

## 注册中心格式

`registry.json` 是 Dashboard 读取的技能清单：

```json
{
  "version": 1,
  "updated": "2026-04-09",
  "skills": [
    {
      "name": "技能名称",
      "description": "一句话描述",
      "author": "GitHub 用户名",
      "version": "1.0.0",
      "tags": ["标签1", "标签2"],
      "files": ["SKILL.md", "agents/agent.md"]
    }
  ]
}
```

文件托管在腾讯云 COS（`https://awd-1253816841.cos.ap-beijing.myqcloud.com/skills-registry/`），本 Git 仓库用于版本管理。
