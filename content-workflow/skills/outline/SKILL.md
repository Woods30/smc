# Outline — 制作优化大纲

## 触发条件

被 article 或 social 工作流调用，在 research 之后。

## 输入

- `.smc/drafts/01-research.md`（研究结果）
- 工作流类型（article / social）

## article 大纲要求

- 多级标题结构（H1 / H2 / H3）
- 每个 H2 下有 2-3 句章节摘要
- 标注关键章节的关键词
- 结尾有总结段落规划
- 字数估算：每章节 200-500 字

## social 大纲要求

- 简洁单层结构
- 每段 1-3 句话
- 开头抓住注意力（Hook）
- 中间核心信息
- 结尾 CTA 或互动引导
- 字数：微博(140-280) / 小红书(300-800) / 公众号(500-1500)

## 输出

保存到 `.smc/drafts/02-outline.md`：

```markdown
# 大纲 — <主题>

## 元信息
- 工作流类型: <article|social>
- 目标字数: <估算>
- 预计阅读时间: <N分钟>

## 标题候选
<3个标题选项>

## 正文结构

### H1: <主标题>
#### H2: <章节1标题>
<2-3句摘要>
##### H3: <要点1>
##### H3: <要点2>

### H2: <章节2标题>
...

## 关键词标注
- <关键词1>: H2/H3 位置
- <关键词2>: H2/H3 位置
```

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "outline",
  "outline": {
    "completedAt": "<timestamp>",
    "type": "article|social",
    "wordCount": <estimate>
  }
}
```