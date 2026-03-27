# Content Workflow — Article 工作流实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现完整的长文章创作工作流，从研究到多平台发布。

**Architecture:** article 技能作为顶层编排器，按顺序调用 research→outline→drafting→edit→verify→seo→humanize→format→platform 子技能。必须先完成 research 才能进行后续步骤。

**Tech Stack:** SKILL.md（Markdown 技能定义）+ JSON 状态管理

---

## 文件结构

```
content-workflow/skills/article/
└── SKILL.md            # 长文章完整流程编排

修正（相对 Plan A）：
- article/SKILL.md 移至顶层，不是 orchestrator/article/
- social/SKILL.md 移至顶层，不是 orchestrator/social/
```

---

## Task 1: 创建完整的 article 工作流技能

**Files:**
- Create: `content-workflow/skills/article/SKILL.md`
- Modify: `content-workflow/skills/orchestrator/SKILL.md`（更新引用路径）

- [ ] **Step 1: Write complete article/SKILL.md**

```markdown
# Article Workflow — 长文章创作完整流程

## 触发条件

用户输入 `/article` 或 `/new-article` 时触发。

## 前置检查

### 必须有研究环节
article 工作流**强制要求**先完成 research 环节。
- 如果 state.json 中无 research 记录，必须先调用 research 技能
- 用户不能跳过研究直接写文章

### 项目初始化
如果 `.smc/state.json` 不存在，先创建：

```json
{
  "type": "article",
  "topic": "<用户主题>",
  "currentStep": null,
  "steps": {
    "research": null,
    "outline": null,
    "drafting": null,
    "edit": null,
    "verify": null,
    "seo": null,
    "humanize": null,
    "format": null,
    "platform": null
  },
  "createdAt": "<timestamp>",
  "updatedAt": "<timestamp>"
}
```

## 流程步骤

### Step 1: research（必须）

**确认节点 1：开始研究前**

向用户确认：
> "开始对「<主题>」进行深度研究。研究将收集相关信息源、关键数据和核心论点。确认开始？"

用户确认后：
1. 调用 `skills/research/SKILL.md`
2. 传入用户主题
3. 读取 `config.json` 获取 research.provider 设置
4. 研究结果保存到 `.smc/drafts/01-research.md`
5. 更新 state.json

### Step 2: outline

基于研究结果生成大纲：
1. 调用 `skills/outline/SKILL.md`
2. 读取 `.smc/drafts/01-research.md`
3. 大纲保存到 `.smc/drafts/02-outline.md`

**确认节点 2：大纲确认**

向用户展示大纲：
- 标题候选（3个选项）
- H1/H2/H3 结构
- 各章节摘要
- 关键词布局
- 字数估算

> "大纲已完成，请确认结构是否需要调整。"

用户确认或修改后，进入下一步。

### Step 3: drafting

基于大纲和研究的完整撰稿：
1. 调用 `skills/drafting/SKILL.md`
2. 读取 `.smc/drafts/02-outline.md` 和 `.smc/drafts/01-research.md`
3. 应用嵌入式去AI味策略
4. 草稿保存到 `.smc/drafts/03-draft.md`

### Step 4: edit

编辑修改初稿：
1. 调用 `skills/edit/SKILL.md`
2. 读取 `.smc/drafts/03-draft.md`
3. 编辑后保存到 `.smc/drafts/04-edit.md`

### Step 5: verify

信息验证审核：
1. 调用 `skills/verify/SKILL.md`
2. 读取 `.smc/drafts/04-edit.md` 和 `.smc/drafts/01-research.md`
3. 验证报告保存到 `.smc/drafts/05-verify.md`

### Step 6: seo

SEO 优化：
1. 调用 `skills/seo/SKILL.md`
2. 读取 `.smc/drafts/05-verify.md` 和 `.smc/drafts/02-outline.md`
3. SEO 优化后保存到 `.smc/drafts/06-seo.md`

### Step 7: humanize

去AI味最终抛光：
1. 调用 `skills/humanize/SKILL.md`
2. 读取 `config.json` 获取 humanize.provider
3. 去AI味后保存到 `.smc/drafts/07-humanize.md`

### Step 8: format

格式化为标准 Markdown：
1. 调用 `skills/format/SKILL.md`
2. 读取 `.smc/drafts/07-humanize.md`
3. 格式化后保存到 `.smc/drafts/08-format.md`

### Step 9: platform

多平台适配输出：
1. 调用 `skills/platform/SKILL.md`
2. 读取 `.smc/drafts/08-format.md`
3. 按平台分别输出到 `.smc/output/` 目录

**确认节点 3：终稿确认**

向用户展示：
- 最终文章
- 各平台适配版本
- SEO 元信息

> "文章已完成！请确认发布。"

用户确认后，更新 state.json 标记 `status: "completed"`。

## 状态流转

```
research → outline → drafting → edit → verify → seo → humanize → format → platform → [completed]
```

每个步骤完成后更新 `state.json`：
```json
{
  "currentStep": "outline",
  "steps": {
    "research": { "completedAt": "<ts>", "status": "done" },
    "outline": { "completedAt": "<ts>", "status": "in_progress" },
    ...
  }
}
```

## 错误处理

- 如果任何步骤失败，更新 state.json 标记失败步骤
- 向用户报告错误和恢复建议
- 支持用户从失败步骤重新开始

## 流程跳过

article 工作流**不允许跳过任何步骤**。
- research 必须
- 所有后续步骤必须按顺序执行
```

- [ ] **Step 2: Update orchestrator to reference top-level article**

```markdown
## 分发逻辑

检查用户意图：
- `/article` → 调用 `skills/article/SKILL.md`
- `/social` → 调用 `skills/social/SKILL.md`
```

- [ ] **Step 3: Commit**

```bash
git add content-workflow/skills/article/SKILL.md
git add content-workflow/skills/orchestrator/SKILL.md
git commit -m "feat: implement complete article workflow with checkpoint confirmations"
```

---

## Task 2: 创建 article 工作流自述文件

**Files:**
- Create: `content-workflow/skills/article/README.md`

- [ ] **Step 1: Write article/README.md**

```markdown
# Article Workflow

长文章创作工作流，从主题研究到多平台发布。

## 流程

research → outline → drafting → edit → verify → seo → humanize → format → platform

## 确认节点

1. 开始研究前
2. 大纲确认
3. 终稿发布

## 状态

`.smc/state.json` 追踪当前步骤和进度。

## 输出

- `.smc/drafts/` 各环节草稿
- `.smc/output/` 各平台适配版本
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/article/README.md
git commit -m "docs: add article workflow README"
```

---

## Task 3: 验证 article 工作流完整性

**Files:**
- Review: `content-workflow/skills/orchestrator/SKILL.md`
- Review: `content-workflow/skills/article/SKILL.md`
- Review: `content-workflow/skills/research/SKILL.md`
- Review: `content-workflow/skills/outline/SKILL.md`
- Review: `content-workflow/skills/drafting/SKILL.md`
- Review: `content-workflow/skills/edit/SKILL.md`
- Review: `content-workflow/skills/verify/SKILL.md`
- Review: `content-workflow/skills/seo/SKILL.md`
- Review: `content-workflow/skills/humanize/SKILL.md`
- Review: `content-workflow/skills/format/SKILL.md`
- Review: `content-workflow/skills/platform/SKILL.md`

- [ ] **Step 1: Verify skill references are correct**

检查 article/SKILL.md 中调用的子技能路径是否与实际文件路径一致：
- `skills/research/SKILL.md` ✓
- `skills/outline/SKILL.md` ✓
- `skills/drafting/SKILL.md` ✓
- `skills/edit/SKILL.md` ✓
- `skills/verify/SKILL.md` ✓
- `skills/seo/SKILL.md` ✓
- `skills/humanize/SKILL.md` ✓
- `skills/format/SKILL.md` ✓
- `skills/platform/SKILL.md` ✓

- [ ] **Step 2: Verify state.json schema consistency**

检查所有子技能对 state.json 的读写是否一致：
- currentStep 字段 ✓
- steps.*.completedAt ✓
- steps.*.status ✓

- [ ] **Step 3: Commit**

```bash
git commit -m "test: verify article workflow skill references and state schema"
```

---

## 验证清单

**Spec 覆盖检查：**
- [ ] article 必须先研究 ✓
- [ ] 三节点确认 ✓
- [ ] 全流程 9 个子技能 ✓
- [ ] 嵌入式去AI味在 drafting ✓
- [ ] 第三方插件槽传递 ✓
- [ ] .smc/ 状态管理 ✓

**Placeholder 扫描：**
- [ ] 无 "TBD" / "TODO" ✓
- [ ] 确认节点明确 ✓
- [ ] 流程无歧义 ✓
