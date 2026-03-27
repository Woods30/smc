# Format — 多格式适配

## 触发条件

被 article 工作流调用，在 humanize 之后。

## 输入

- `.smc/drafts/07-humanize.md`（去AI味后稿件）

## 输出格式

### Markdown 输出（默认）
- 标准 Markdown 格式
- 正确的 H1/H2/H3 层级
- 有序/无序列表正确使用
- 代码块适当使用
- 链接和图片引用规范

### 各平台格式适配（调用 platform 技能）

不在此技能处理，由 platform 技能负责。

## 输出

保存到 `.smc/drafts/08-format.md`：
- 格式化后的标准 Markdown

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "format",
  "format": {
    "completedAt": "<timestamp>",
    "format": "markdown"
  }
}
```
