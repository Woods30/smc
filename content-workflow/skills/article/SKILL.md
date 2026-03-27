# Article Workflow — 长文章创作流程编排

## 触发条件

被 orchestrator 调用，或用户直接输入 `/article`

## 流程步骤

1. research — 深度研究（必须）
2. outline — 大纲生成
3. drafting — 撰稿（含嵌入式去AI味）
4. edit — 编辑修改
5. verify — 信息验证
6. seo — SEO优化
7. humanize — 去AI味（最终抛光）
8. format — 格式适配
9. platform — 平台发布

## 确认节点

1. 开始研究前确认主题
2. 大纲生成后确认结构
3. 终稿完成后确认发布

## 状态更新

每步完成后更新 `.smc/state.json` 中的 currentStep。
