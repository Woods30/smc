---
name: humanize
description: 去AI味技能 - 支持 StealthWriter、UndetectableAI 第三方插件槽
---

# Humanize — 去AI味

## 触发条件

1. 被 article 工作流在 seo 之后调用（最终抛光）
2. 被 social 工作流在 drafting 之后调用
3. 配置决定使用内置还是第三方

## 输入

- `.smc/drafts/06-seo.md`（article）或 `.smc/drafts/03-draft.md`（social）
- 配置（从 config.json 读取 humanize.provider）

## 内置模式（默认 / stealthwriter）

使用高级提示词工程进行去AI味改写：
- 深度句式重构
- 词汇多样化替换
- 人类写作特征注入（犹豫、强调、不完美）

### 改写原则
1. 保持原意 100%
2. 改变句式结构
3. 替换高频AI词汇
4. 增加人类写作痕迹

## 第三方模式（provider: undetectableai）

调用 UndetectableAI API 进行去AI味。

## 输出

保存到 `.smc/drafts/07-humanize.md`：
- 去AI味后的完整稿件

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "humanize",
  "humanize": {
    "completedAt": "<timestamp>",
    "provider": "<used provider>",
    "wordCountBefore": <before>,
    "wordCountAfter": <after>
  }
}
```