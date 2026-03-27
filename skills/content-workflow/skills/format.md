---
name: content-workflow-format
description: 多格式适配 - 输出Markdown格式
---

# 多格式适配

## 输入

- 去AI味后内容 (`07-humanize.md`)
- 目标格式

## 输出格式

当前版本: 纯 Markdown

### Markdown 规范
- 标题层级: H1 > H2 > H3
- 列表: 有序/无序混用
- 强调: 粗体/斜体适度
- 引用: 用于重要引述
- 代码块: 用于技术内容

## 输出结构

```markdown
# 最终内容

[标准Markdown格式]

## 附录
- 字数统计
- 关键词分布
```

## Context 控制

- 最终Markdown存入 `08-format.md`
- 摘要存入 `state.json summaries.format`
