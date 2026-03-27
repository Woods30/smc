---
name: content-workflow-context
description: 内容创作工作流的Context控制（分块+摘要）
---

# 内容创作工作流 - Context 控制

防止 Context 溢出，控制每个步骤的输入输出大小。

## 分块规则

触发条件: 内容 > 2000 tokens

分块策略:
- 按章节/段落拆分
- 每个分块独立文件
- 保留分块间引用关系

## 摘要规则

每个检查点完成时生成摘要:
- 长度: ≤ 200 字
- 内容: 核心发现/关键点
- 用途: 注入后续步骤 Context

## Manifest 结构

```json
{
  "version": 1,
  "chunks": {
    "research_01": {
      "file": "chunks/research_01.md",
      "ref": "01-research.md:0-500",
      "summary": "研究摘要",
      "tokens": 1800
    }
  },
  "active_summary": "当前使用的摘要链"
}
```

## 使用流程

1. 输入 > 2000 tokens → 自动分块
2. 分块存入 `chunks/`
3. 更新 `manifest.json`
4. 摘要注入 Context
5. 后续步骤按需读取分块或摘要