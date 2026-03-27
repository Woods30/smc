---
name: content-workflow-orchestrator
description: 内容创作工作流编排器
---

# 内容创作工作流 - 编排器

主入口技能，负责任务分解、流程调度、状态维护。

## 启动方式

用户输入 `/content-workflow` 或 `开始内容创作` 触发。

## 初始流程

1. 检查 `.smc/` 目录是否存在
2. 不存在 → 初始化项目结构
3. 存在 → 读取 `state.json`，恢复上次进度
4. 询问用户是否继续当前项目或开始新项目

## 命令

| 命令 | 说明 |
|------|------|
| `开始新项目` | 初始化新项目 |
| `继续` | 从当前步骤继续 |
| `跳到 {步骤}` | 跳转到指定步骤 |
| `状态` | 显示当前进度 |
| `重置` | 重置项目 |

## 步骤列表

1. research - 深度主题研究
2. outline - 制作优化大纲
3. draft - 撰稿协作
4. edit - 编辑修改
5. verify - 信息验证审核
6. seo - SEO优化
7. humanize - 去AI味
8. format - 多格式适配
9. platform - 平台适配

## 执行流程

```
开始新项目?
  → 询问项目名称和主题
  → 初始化 .smc/ 结构
  → 进入 research

继续?
  → 读取 state.json
  → 进入 current_step

跳到 {步骤}?
  → 更新 current_step
  → 执行该步骤及后续步骤
```

## 子技能调用

每个步骤调用对应 skill:
- 使用 `invoke_skill` 调用子技能
- 传递当前 state 和前序步骤的草稿
- 子技能完成后更新状态
