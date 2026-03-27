# Verify — 信息验证审核

## 触发条件

被 article 工作流调用，在 edit 之后。social 不调用此技能。

## 输入

- `.smc/drafts/04-edit.md`（编辑后稿件）
- `.smc/drafts/01-research.md`（研究结果，含来源）

## 验证维度

### 事实核查
- 统计数据与来源一致
- 引用的研究/报告真实存在
- 数字、日期、名称准确

### 来源追溯
- 每个引用claim有对应来源
- URL 可访问（抽样验证）
- 来源权威性评估

### 逻辑自洽
- 全文核心论点无矛盾
- 数据解读无误导

### 合规检查
- 无版权问题文字
- 无明显虚假宣传

## 输出

保存到 `.smc/drafts/05-verify.md`：

```markdown
# 验证报告

## 通过项
- <检查项1>: ✓
- <检查项2>: ✓

## 风险项（需人工确认）
- <风险描述>

## 建议修改
- <建议1>
```

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "verify",
  "verify": {
    "completedAt": "<timestamp>",
    "status": "passed|warning",
    "issues": [<issue list>]
  }
}
```
