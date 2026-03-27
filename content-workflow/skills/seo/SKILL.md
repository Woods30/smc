# SEO — SEO优化

## 触发条件

被 article 工作流调用，在 verify 之后。social 不调用此技能。

## 输入

- `.smc/drafts/05-verify.md`（验证后稿件）
- `.smc/drafts/02-outline.md`（大纲，含关键词标注）

## SEO 优化维度

### 关键词布局
- 标题（H1）必须包含主关键词
- 首段前100字必须出现主关键词
- H2/H3 包含关键词变体
- 全文关键词密度 1-2%
- 关键词自然分布，不堆砌

### 元信息优化
- Meta Title（50-60字符）
- Meta Description（150-160字符）
- URL 友好化（英文/拼音 slug）

### 内容结构
- 内部链接建议（锚文本）
- 图片 alt 文本（描述性）
- 段落长度控制（便于 NLP 解析）

## 输出

保存到 `.smc/drafts/06-seo.md`：
- SEO 优化后的完整稿件
- 包含元信息区块

```markdown
---
title: <SEO标题>
description: <Meta描述>
slug: <url-slug>
keywords: <关键词列表>
---

<完整正文>
```

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "seo",
  "seo": {
    "completedAt": "<timestamp>",
    "metaTitle": "<生成标题>",
    "metaDescription": "<生成描述>"
  }
}
```
