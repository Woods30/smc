# Orchestrator — 内容创作工作流总入口

## 触发条件

当用户输入包含 `/article` 或 `/social` 时触发。

## 分发逻辑

检查用户意图：
- `/article` → 调用 article/SKILL.md
- `/social` → 调用 social/SKILL.md

如果用户只说"写文章"但未指定类型，询问用户选择 article 还是 social。

## 流程说明

- `/article`: 完整长文章流程（研究→大纲→撰稿→编辑→验证→SEO→去AI味→格式→发布）
- `/social`: 社交内容流程（研究→大纲→撰稿→去AI味→平台适配）

## 状态初始化

创建项目状态文件 `.smc/state.json`：
```json
{
  "type": null,
  "currentStep": null,
  "createdAt": "<timestamp>",
  "updatedAt": "<timestamp>"
}
```

## 第三方插件槽

通过 `config.json` 配置：
- research: deep-research / tavily / firecrawl
- humanize: stealthwriter / undetectableai

读取配置并传递给对应技能。
