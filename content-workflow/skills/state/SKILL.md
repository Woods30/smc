---
name: content-workflow-state
description: 内容创作工作流的状态管理
---

# 内容创作工作流 - 状态管理

管理整个工作流的状态流转。

## 状态结构 (state.json)

```json
{
  "project": {
    "name": "项目名称",
    "created_at": "ISO时间",
    "updated_at": "ISO时间"
  },
  "current_step": "research",
  "step_status": {
    "research": "completed|in_progress|pending",
    "outline": "pending",
    "draft": "pending",
    "edit": "pending",
    "verify": "pending",
    "seo": "pending",
    "humanize": "pending",
    "format": "pending",
    "platform": "pending"
  },
  "summaries": {
    "research": "研究摘要（200字以内）",
    "outline": "大纲摘要",
    ...
  },
  "metadata": {}
}
```

## 步骤定义

| 步骤 | 文件 | 说明 |
|------|------|------|
| research | 01-research.md | 深度主题研究 |
| outline | 02-outline.md | 制作优化大纲 |
| draft | 03-draft.md | 撰稿协作 |
| edit | 04-edit.md | 编辑修改 |
| verify | 05-verify.md | 信息验证审核 |
| seo | 06-seo.md | SEO优化 |
| humanize | 07-humanize.md | 去AI味 |
| format | 08-format.md | 多格式适配 |
| platform | 09-platform.md | 平台适配 |

## 流转规则

1. 初始状态: `current_step = "research"`, 所有步骤为 `pending`
2. 步骤完成: 更新 `step_status[step] = "completed"`
3. 步骤开始: 更新 `current_step` 和 `step_status[step] = "in_progress"`
4. 支持跳转: 用户可指定跳转任意 `pending` 步骤