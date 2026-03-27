---
name: social
description: 社交内容创作流程编排 - 轻量流程：研究→大纲→撰稿→去AI味→平台适配
---

# Social Workflow — 社交内容创作流程编排

## 触发条件

被 orchestrator 调用，或用户直接输入 `/social`

## 流程步骤

1. research — 轻量研究（基于素材）
2. outline — 轻量大纲
3. drafting — 撰稿
4. humanize — 去AI味
5. platform — 平台适配

## 确认节点

1. 开始前确认主题和平台目标
2. 终稿确认

## 状态更新

每步完成后更新 `.smc/state.json` 中的 currentStep。
