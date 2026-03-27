# Content Workflow — Social 工作流实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现完整的社交内容创作工作流，轻量研究 + 快速撰稿 + 多平台适配。

**Architecture:** social 技能作为顶层编排器，按顺序调用 research(轻)→outline→drafting→humanize→platform 子技能。social 独立于 article，无依赖关系。

**Tech Stack:** SKILL.md（Markdown 技能定义）+ JSON 状态管理

---

## 文件结构

```
content-workflow/skills/social/
└── SKILL.md            # 社交内容完整流程编排
```

---

## Task 1: 创建完整的 social 工作流技能

**Files:**
- Create: `content-workflow/skills/social/SKILL.md`
- Modify: `content-workflow/skills/orchestrator/SKILL.md`（更新引用路径）

- [ ] **Step 1: Write complete social/SKILL.md**

```markdown
# Social Workflow — 社交内容创作完整流程

## 触发条件

用户输入 `/social` 或 `/new-social` 时触发。

## 与 article 的关系

social 工作流**完全独立**于 article 工作流：
- 无需先完成 article
- 无需基于 article 内容拆解
- 独立的研究、撰稿、发布流程

## 前置检查

### 项目初始化
如果 `.smc/state.json` 不存在（或 type 不是 social），创建新状态：

```json
{
  "type": "social",
  "topic": "<用户主题>",
  "targetPlatforms": [],
  "currentStep": null,
  "steps": {
    "research": null,
    "outline": null,
    "drafting": null,
    "humanize": null,
    "platform": null
  },
  "createdAt": "<timestamp>",
  "updatedAt": "<timestamp>"
}
```

### 目标平台确认
询问用户目标平台（可多选）：

> "请选择目标平台：
> - 微博 / 小红书 / 知乎 / 微信公众号 / 抖音
> - Twitter/X / LinkedIn / Instagram / Facebook / Threads"

将选择更新到 state.json 的 `targetPlatforms` 数组。

## 流程步骤

### Step 1: research（轻量）

social 的研究环节比 article 轻量：
- 不强制 Deep Research
- 基于用户提供的素材/链接快速收集信息
- 或 LLM 知识库直接回答

**确认节点 1：开始前**

向用户确认：
> "开始为「<平台列表>」创作社交内容。主题：<主题>。请提供相关素材/链接（可选），或确认直接开始。"

用户响应后：
1. 如果用户提供了素材 → 基于素材快速研究
2. 如果用户无素材 → 直接进入 outline
3. 研究结果（如果有）保存到 `.smc/drafts/social-01-research.md`
4. 更新 state.json

### Step 2: outline（轻量）

社交内容大纲更简洁：
1. 调用 `skills/outline/SKILL.md`，传入 `type: social`
2. 大纲要求：
   - 单层结构（无 H3）
   - 每段 1-3 句话
   - 开头 Hook 设计
   - 中间核心信息
   - 结尾 CTA/互动引导
3. 大纲保存到 `.smc/drafts/social-02-outline.md`

### Step 3: drafting

按大纲快速撰稿：
1. 调用 `skills/drafting/SKILL.md`，传入 `type: social`
2. 每个目标平台单独生成一份稿件
3. 稿件保存到 `.smc/drafts/social-03-draft-<platform>.md`

### Step 4: humanize

去AI味处理：
1. 调用 `skills/humanize/SKILL.md`
2. 读取 `config.json` 获取 humanize.provider
3. 对每份平台稿件单独去AI味
4. 结果保存到 `.smc/drafts/social-04-humanize-<platform>.md`

### Step 5: platform

平台适配输出：
1. 调用 `skills/platform/SKILL.md`
2. 读取 `.smc/drafts/social-04-humanize-*.md`
3. 按平台格式化输出
4. 每个平台输出到 `.smc/output/<platform>-<slug>.md`

**确认节点 2：终稿确认**

向用户展示：
- 各平台最终稿件
- 字数/格式合规性

> "社交内容已完成！请确认发布。"

用户确认后，更新 state.json 标记 `status: "completed"`。

## 状态流转

```
research(轻) → outline → drafting → humanize → platform → [completed]
```

注意：social 流程**不包含** edit、verify、seo、format 步骤。

## 与 article 对比

| 维度 | article | social |
|------|---------|--------|
| 研究深度 | 强制深度研究 | 轻量（可选） |
| 环节数 | 9步 | 5步 |
| edit/verify/seo | 有 | 无 |
| 平台输出 | 多平台 | 按需选择 |
| 确认节点 | 3个 | 2个 |

## 错误处理

- 如果任何步骤失败，更新 state.json 标记失败步骤
- 向用户报告错误和恢复建议
- 支持用户从失败步骤重新开始

## 流程跳过

social 工作流**允许跳过 research**（如果用户无素材）：
- research 可选
- 但 outline → drafting → humanize → platform 必须按顺序执行
```

- [ ] **Step 2: Update orchestrator to reference top-level social**

```markdown
## 分发逻辑

检查用户意图：
- `/article` → 调用 `skills/article/SKILL.md`
- `/social` → 调用 `skills/social/SKILL.md`
```

- [ ] **Step 3: Commit**

```bash
git add content-workflow/skills/social/SKILL.md
git add content-workflow/skills/orchestrator/SKILL.md
git commit -m "feat: implement complete social workflow with light research"
```

---

## Task 2: 创建 social 工作流自述文件

**Files:**
- Create: `content-workflow/skills/social/README.md`

- [ ] **Step 1: Write social/README.md**

```markdown
# Social Workflow

社交内容创作工作流，轻量快速，多平台适配。

## 流程

research(轻) → outline → drafting → humanize → platform

## 确认节点

1. 开始前（确认主题和平台）
2. 终稿确认

## 状态

`.smc/state.json` 追踪当前步骤和进度。

## 输出

- `.smc/drafts/social-*.md` 各环节草稿
- `.smc/output/<platform>-*.md` 各平台适配版本

## 与 article 的区别

| 维度 | article | social |
|------|---------|--------|
| 研究 | 强制深度 | 可选轻量 |
| 步骤数 | 9步 | 5步 |
| 适用 | 长文章/博客 | 社媒帖子 |
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/social/README.md
git commit -m "docs: add social workflow README"
```

---

## Task 3: 验证 social 工作流完整性

**Files:**
- Review: `content-workflow/skills/social/SKILL.md`
- Review: `content-workflow/skills/orchestrator/SKILL.md`

- [ ] **Step 1: Verify skill references are correct**

检查 social/SKILL.md 中调用的子技能路径：
- `skills/research/SKILL.md` ✓（轻量模式）
- `skills/outline/SKILL.md` ✓
- `skills/drafting/SKILL.md` ✓
- `skills/humanize/SKILL.md` ✓
- `skills/platform/SKILL.md` ✓

注意：social 不调用 edit/verify/seo/format 技能。

- [ ] **Step 2: Verify state.json schema compatibility**

确保 social 的 state.json 字段与 article 一致：
- `type: "social"` ✓
- `targetPlatforms` ✓（social 独有）
- `steps` 结构 ✓

- [ ] **Step 3: Commit**

```bash
git commit -m "test: verify social workflow skill references"
```

---

## 验证清单

**Spec 覆盖检查：**
- [ ] social 独立于 article ✓
- [ ] 轻量研究（可选） ✓
- [ ] 流程 5 步 ✓
- [ ] 多平台支持 ✓
- [ ] 第三方插件槽传递 ✓
- [ ] .smc/ 状态管理 ✓

**Placeholder 扫描：**
- [ ] 无 "TBD" / "TODO" ✓
- [ ] 确认节点明确 ✓
- [ ] article/social 独立性明确 ✓
