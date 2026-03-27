---
name: platform
description: 平台适配与发布技能 - 支持国内外多平台内容适配
---

# Platform — 平台适配与发布

## 触发条件

被 article 或 social 工作流调用，作为最终环节。

## 输入

- `.smc/drafts/08-format.md`（格式化稿件，article）
- `.smc/drafts/07-humanize.md`（social，直接来自 humanize）
- 目标平台列表

## 支持平台

### 国内
- 微信公众号：封面图建议、摘要（不含emoji）、字数控制
- 微博：140字限制、话题标签、@提及
- 小红书：emoji、hashtag、200-800字
- 知乎：专业语气、引用格式
- 抖音/快手：短文案、悬念式开头

### 海外
- Twitter/X：280字符、hashtag
- LinkedIn：专业语气、CTA
- Instagram：caption、hashtag墙
- Facebook：长文案、emoji
- Threads：短内容、互动引导

## 平台适配处理

1. 字数裁剪/扩展
2. 标题党适配（适度）
3. 表情符号处理
4. hashtag 格式转换
5. @提及和链接处理

## 输出

每个平台单独输出文件：
- `.smc/output/wechat-<slug>.md`
- `.smc/output/twitter-<slug>.md`
- `.smc/output/xiaohongshu-<slug>.md`
- ...

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "platform",
  "platform": {
    "completedAt": "<timestamp>",
    "outputs": ["wechat", "twitter", "xiaohongshu"]
  }
}
```

## 标记完成

当 platform 完成后，更新 state.json 标记整个工作流完成：
```json
{
  "status": "completed",
  "completedAt": "<timestamp>"
}
```
