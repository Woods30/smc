---
name: research
description: 深度主题研究技能 - 支持 Deep Research、Tavily、Firecrawl 第三方插件槽
---

# Research — 深度主题研究

## 触发条件

被 article 或 social 工作流调用。article 必须触发。

## 输入

- 主题（用户提供的文章/内容主题）
- 配置（从 config.json 读取 research.provider）

## 流程

### Deep Research 模式（默认 / deep-research）

1. 使用官方 Deep Research 能力搜索主题相关资料
2. 收集来源：网页、文章、数据源
3. 提取关键发现、核心论点、支持数据
4. 输出来源列表（URL + 摘要）

### Tavily 模式（provider: tavily）

1. 调用 Tavily API 搜索主题
2. 聚合搜索结果
3. 提取结构化信息

### Firecrawl 模式（provider: firecrawl）

1. 使用 Firecrawl 爬取相关网页
2. 提取页面核心内容
3. 去重和内容质量过滤

## 输出

保存到 `.smc/drafts/01-research.md`：

```markdown
# 研究报告

## 主题
<用户主题>

## 关键发现
- <发现1>
- <发现2>
...

## 核心数据/统计
- <数据1>
- <数据2>
...

## 信息来源
1. <来源1> — <URL>
2. <来源2> — <URL>
...

## 研究时间
<timestamp>
```

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "research",
  "research": {
    "completedAt": "<timestamp>",
    "provider": "<used provider>",
    "sourceCount": <number>
  }
}
```