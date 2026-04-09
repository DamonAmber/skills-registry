# CLAUDE.md — Skills Registry 开发规范

## 仓库定位

这是 AI Workspace Dashboard 的公开技能商店，通过 `raw.githubusercontent.com` 提供技能文件下载。

## 目录结构（严格遵守）

```
/
├── registry.json              ← 技能清单（Dashboard 读取入口）
├── README.md
├── CLAUDE.md
├── <skill-name>/              ← 每个技能一个目录，直接在仓库根目录下
│   ├── SKILL.md               ← 必须有，技能定义文件
│   ├── agents/                ← 可选，子 Agent 定义
│   ├── references/            ← 可选，参考资料
│   └── scripts/               ← 可选，脚本文件
└── <another-skill>/
    └── SKILL.md
```

**关键约束：技能目录必须放在仓库根目录下，禁止嵌套在 `skills/` 或任何其他子目录中。**

原因：Dashboard 的下载 URL 格式为 `${registryUrl}/${skillName}/${filePath}`，如果目录层级不对，安装时会 404。

## 添加新技能的标准流程

### 1. 创建技能目录

```bash
# 正确 ✓
mkdir <skill-name>

# 错误 ✗ — 不要放在 skills/ 子目录下
mkdir -p skills/<skill-name>
```

### 2. 复制 SKILL.md

从源位置（通常是 `~/.claude/skills/<skill-name>/SKILL.md`）复制到仓库根目录下的同名目录：

```bash
cp ~/.claude/skills/<skill-name>/SKILL.md ./<skill-name>/SKILL.md
```

如果 skill 有子文件（agents/、references/ 等），也一并复制，保持相对路径结构。

### 3. 清理路径

SKILL.md 中不应包含本地绝对路径（如 `/Users/xxx/`、`~/Documents/`）。上传前检查并移除。

### 4. 更新 registry.json

在 `skills` 数组中追加条目：

```json
{
  "name": "<skill-name>",
  "description": "中文一句话描述，含功能和触发场景",
  "author": "DamonAmber",
  "version": "1.0.0",
  "tags": ["tag1", "tag2"],
  "files": ["SKILL.md"]
}
```

**files 字段规则：**
- 列出技能目录内的所有文件，使用相对于技能目录的路径
- 例如 `["SKILL.md", "agents/designer.md", "references/classification.md"]`
- 不要包含技能目录名本身（Dashboard 会自动拼接 `<skill-name>/` 前缀）

同时更新 `updated` 日期字段。

### 5. 更新 README.md

在"可用技能"表格中追加新行。

### 6. 提交并推送

```bash
git add <skill-name>/ registry.json README.md
git commit -m "feat: 添加 <skill-name> 技能到商店"
git push origin main
```

注意：GitHub raw CDN 有约 5 分钟缓存，推送后 Dashboard 刷新可能需要等几分钟才能看到最新内容（已加 cache-busting 参数缓解）。

## 常见错误

| 错误 | 后果 | 正确做法 |
|------|------|---------|
| 把技能目录放在 `skills/` 下 | 安装时文件下载 404 | 放在仓库根目录 |
| files 字段包含技能目录名前缀 | 下载路径重复，404 | 只写相对于技能目录的路径 |
| 忘记更新 registry.json | 商店里看不到新技能 | 必须在 skills 数组追加条目 |
| SKILL.md 包含本地绝对路径 | 其他用户安装后路径无效 | 上传前清理本地路径 |
| 忘记更新 README.md | 人类浏览仓库时看不到新技能 | 同步更新可用技能表格 |

## registry.json 完整格式

```json
{
  "version": 1,
  "updated": "YYYY-MM-DD",
  "skills": [
    {
      "name": "技能名称（与目录名一致）",
      "description": "中文描述",
      "author": "DamonAmber",
      "version": "1.0.0",
      "tags": ["标签数组"],
      "files": ["SKILL.md", "子目录/文件.md"]
    }
  ]
}
```
