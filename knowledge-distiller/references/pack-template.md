# PACK.md 模板

以下是知识包定义文件的完整模板，创建知识包时复制并填写。

```markdown
---
name: <pack-name>
description: <一句话中文描述，说明知识包覆盖的领域和内容>
author: <作者名>
version: 1.0.0
domain: <领域标识>
tags: [tag1, tag2, tag3]
files:
  - business/<file1>.md
  - business/<file2>.md
  - page-map/<file3>.md
---

# <知识包名称>

<2-3 句话概述知识包的核心内容和价值>

## 包含内容

- **业务知识**：<描述>
- **页面地图**：<描述>
- **数据定义**：<描述>
- **设计规范**：<描述>

## 适用场景

- <场景 1>
- <场景 2>
- <场景 3>

## 知识来源

- <来源 1，如：产品文档、PRD>
- <来源 2，如：设计稿、交互规范>
- <来源 3，如：数据分析报告>

## 前置知识

使用此知识包前，建议先了解：
- <前置知识 1>
- <前置知识 2>

## 更新记录

| 版本 | 日期 | 变更说明 |
|------|------|---------|
| 1.0.0 | YYYY-MM-DD | 初始版本 |
```

## domain 标识参考

| domain | 含义 |
|--------|------|
| provisioning | 配网 |
| consumables | 耗材 |
| ota | OTA 升级 |
| network | 网络场景 |
| family | 家庭/角色 |
| home-tab | 首页 |
| device-detail | 设备详情 |
| general | 通用/跨领域 |

## tags 参考

常用 tag：`business`、`page-map`、`design`、`data`、`user-research`、`provisioning`、`consumables`、`ota`、`network`、`family`、`mijia`
