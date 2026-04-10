---
name: knowledge-distiller
description: 将领域原始材料（文档、截图、经验笔记）蒸馏提炼为结构化知识包（Knowledge Pack），可通过 AI Workspace Dashboard 分享安装。触发场景：用户说"创建知识包"、"蒸馏知识"、"打包领域知识"、"distill knowledge"。
---

# Knowledge Distiller

将散落的领域知识材料（文档、设计稿、截图、经验总结）蒸馏提炼为结构化的知识包（Knowledge Pack），供团队共享，安装后 AI 立即获得领域专业能力。

## 何时使用

- 用户想把某个业务领域的知识整理成可分享的知识包
- 用户说"创建知识包"、"蒸馏知识"、"打包领域知识"
- 用户想将 `knowledge/` 目录下的内容打包发布

## 知识包格式

每个知识包是一个目录，包含 `PACK.md` 定义文件和知识文件：

```
<pack-name>/
├── PACK.md                    ← 包定义文件（必须）
├── business/                  ← 业务知识
│   ├── overview.md
│   └── terminology.md
├── page-map/                  ← 页面地图
│   └── flow-map.md
├── design-system/             ← 设计规范
│   └── patterns.md
├── data/                      ← 数据定义
│   └── metrics.md
└── assets/                    ← 引用的图片资源（可选）
    └── decision-tree.png
```

### PACK.md 格式

```yaml
---
name: <pack-name>
description: 一句话中文描述
author: <作者名>
version: 1.0.0
domain: <领域标识，如 provisioning、consumables、ota>
tags: [tag1, tag2, tag3]
files:
  - business/overview.md
  - business/terminology.md
  - page-map/flow-map.md
---

# <知识包名称>

## 包含内容
- ...

## 适用场景
- ...

## 知识来源
- ...
```

### registry.json 条目格式

```json
{
  "name": "<pack-name>",
  "description": "中文描述",
  "author": "<作者名>",
  "version": "1.0.0",
  "domain": "<领域>",
  "tags": ["tag1", "tag2"],
  "files": ["PACK.md", "business/overview.md", ...],
  "fileCount": 5,
  "totalSizeKB": 30
}
```

## 蒸馏工作流

### 第一步：收集原始材料

确认原始材料来源。可能的来源：

| 来源类型 | 位置 | 说明 |
|----------|------|------|
| 现有知识库 | `knowledge/` 子目录 | 已有的结构化知识 |
| 工作产出 | `work/` 下的任务目录 | PRD、调研报告、数据分析 |
| 设计资源 | `assets/` | 截图、设计稿、设计 token |
| 记忆文件 | `memory/AI_MEMORY.md` | 跨会话积累的事实 |
| 会话日志 | `session-logs/` | 重要工作的记录 |
| 外部文档 | 用户指定 | 飞书文档、Confluence 等 |

与用户确认：
1. **领域范围**：这个知识包覆盖什么业务领域？
2. **材料清单**：有哪些原始材料？列出文件路径或链接
3. **目标受众**：谁会使用这个知识包？需要什么前置知识？
4. **包名**：用英文短横线命名，如 `provisioning-domain`

### 第二步：分析原始材料

逐一阅读所有原始材料，提取以下维度的知识：

#### 2.1 核心概念与术语

- 领域专有名词及其定义
- 概念之间的关系（层级、依赖、互斥）
- 容易混淆的相似概念
- 缩写和全称对照

#### 2.2 业务逻辑与规则

- 功能的设计初衷和业务背景
- 用户场景和使用流程
- 决策规则（如 sctype 映射、分叉条件）
- 边界条件和特殊情况处理

#### 2.3 数据模型与指标

- 关键数据字段定义
- 核心业务指标和计算口径
- 数据埋点和追踪事件
- 常用数据查询模式

#### 2.4 页面结构与交互

- 页面层级结构和导航关系
- 核心页面的功能布局
- 关键交互流程（步骤、分支、异常处理）
- 设计规范和组件复用

#### 2.5 经验与陷阱

- 常见的误解和错误
- 已踩过的坑和解决方案
- 性能/兼容性注意事项
- 与其他模块的耦合关系

### 第三步：结构化蒸馏

将提取的知识组织为标准目录结构。遵循以下原则：

**精炼原则**：
- 去除临时性信息（会议记录、进度更新）
- 去除重复内容，合并同一主题的多处描述
- 保留"为什么"而不仅是"是什么"
- 用表格和列表代替大段文字
- 数据和事实标注来源和时间

**组织原则**：
- 每个文件聚焦一个主题，标题清晰
- 文件名用英文短横线，如 `provisioning-sctype-taxonomy.md`
- 按 `knowledge/` 子目录分类：`business/`、`page-map/`、`design-system/`、`data/`、`user-research/`
- 最重要的概览文件排在最前

**格式规范**：
- 每个 .md 文件开头用一句话说明内容和用途
- 术语首次出现时用加粗标注并给出定义
- 引用数据时标注来源和置信度
- 不确定的内容标注"待验证"

### 第四步：输出到暂存目录

将知识包写到 `tmp/knowledge-pack-staging/<pack-name>/` 目录：

```bash
# 输出结构示例
tmp/knowledge-pack-staging/provisioning-domain/
├── PACK.md
├── business/
│   ├── provisioning-overview.md
│   ├── provisioning-sctype-taxonomy.md
│   └── provisioning-terminology.md
├── page-map/
│   └── provisioning-flow-map.md
└── data/
    └── provisioning-metrics.md
```

生成完毕后，告知用户：
1. 暂存位置
2. 文件清单和每个文件的简要说明
3. 总文件数和预估大小

### 第五步：用户审核

请用户审核知识包内容：
- 检查是否有遗漏的重要知识
- 检查是否有不准确的描述
- 检查是否有不应公开的敏感信息
- 确认 PACK.md 的元数据是否正确

根据用户反馈修改，直到用户满意。

### 第六步：发布到 Registry（可选）

用户确认后，如果要发布到 knowledge-registry：

1. 将暂存目录复制到 knowledge-registry 仓库根目录
2. 更新 `registry.json`，在 `packs` 数组中追加条目
3. 更新 `README.md` 的可用知识包列表
4. 提交并推送

```bash
# 示例命令（用户需自行操作或确认）
cp -r tmp/knowledge-pack-staging/<pack-name> <registry-repo-path>/
cd <registry-repo-path>
# 手动更新 registry.json 和 README.md
git add . && git commit -m "feat: add <pack-name> knowledge pack" && git push
```

## 质量检查清单

蒸馏完成后，对照检查：

- [ ] PACK.md frontmatter 字段完整（name, description, author, version, domain, tags, files）
- [ ] files 列表与实际文件一一对应
- [ ] 每个 .md 文件有清晰的开头说明
- [ ] 术语有定义，数据有来源
- [ ] 不包含本地绝对路径（`/Users/xxx/`）
- [ ] 不包含敏感信息（密钥、内部 URL 等）
- [ ] 不包含临时性内容（会议纪要、进度记录）
- [ ] 文件名全小写英文短横线
- [ ] 总体积合理（建议单包 < 200KB 文本）

## 示例对话

**用户**：帮我把配网相关的知识整理成知识包

**AI**：
1. 扫描 `knowledge/business/`、`knowledge/page-map/`、`work/` 下配网相关文件
2. 列出找到的原始材料，请用户确认和补充
3. 执行蒸馏流程
4. 输出到 `tmp/knowledge-pack-staging/provisioning-domain/`
5. 请用户审核

**用户**：把这个目录下的文档打包成知识包 → `work/08-prd/2026-03-31-offline-diagnosis-v2/`

**AI**：
1. 读取该目录下所有文件
2. 提取核心知识，去除 PRD 格式化内容
3. 按主题分类蒸馏
4. 输出知识包
