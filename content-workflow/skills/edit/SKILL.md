---
name: edit
description: 编辑修改技能 - 语法、用词、逻辑优化
---

# Edit — 编辑修改

## 触发条件

被 article 工作流调用，在 drafting 之后。不被 social 工作流调用（social 直接进 humanize）。

## 输入

- `.smc/drafts/03-draft.md`（初稿）

## 编辑维度

### 语法检查
- 错别字、标点错误
- 语病、句式不通
- 主谓宾一致

### 用词优化
- 替换生硬表达
- 提升词汇精准度
- 去除冗余词汇

### 逻辑强化
- 段落间逻辑连贯
- 论证链条完整
- 过渡自然

### 风格一致性
- 保持全文语气统一
- 检查称呼一致性
- 标题与正文风格匹配

## 输出

保存到 `.smc/drafts/04-edit.md`：
- 修改后的完整稿件
- 文件头部标注修改说明

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "edit",
  "edit": {
    "completedAt": "<timestamp>",
    "changes": "<简要变更说明>"
  }
}
```