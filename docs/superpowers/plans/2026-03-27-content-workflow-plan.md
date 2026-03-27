# 内容创作工作流技能 - 实现规划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现一套完整的内容创作工作流技能，包含9个模块和1个编排器，支持状态保存、context控制和灵活跳转。

**Architecture:** 技能编排器模式，每个子技能独立又能被编排器调度。存储层抽象化，状态以Markdown和JSON文件保存。

**Tech Stack:** Superpowers 技能框架、Claude API (Deep Research + Claude)、文件系统存储

---

## 文件结构

```
skills/content-workflow/
├── orchestrator.md           # 编排器技能（主入口）
├── storage.md                # 存储抽象接口
├── state.md                  # 状态管理模块
├── context.md                # Context控制模块（分块+摘要）
│
├── skills/
│   ├── research.md           # 深度主题研究
│   ├── outline.md            # 制作优化大纲
│   ├── drafting.md           # 撰稿协作（含去AI味）
│   ├── edit.md               # 编辑修改
│   ├── verify.md             # 信息验证审核
│   ├── seo.md                # SEO优化
│   ├── humanize.md           # 去AI味
│   ├── format.md             # 多格式适配
│   └── platform.md           # 平台适配
│
└── README.md                 # 总览文档
```

---

## 实现顺序

### 阶段1：基础设施（存储 + 状态 + Context）

### Task 1: 存储抽象接口

**Files:**
- Create: `skills/content-workflow/storage.md`

- [ ] **Step 1: 创建存储抽象接口 skill**

```markdown
---
name: content-workflow-storage
description: 内容创作工作流的存储抽象接口
---

# 内容创作工作流 - 存储接口

定义统一的存储接口，各技能通过此接口读写状态和草稿。

## 接口定义

### write_draft(step, content, metadata)
- `step`: 步骤名 (research, outline, draft, etc.)
- `content`: Markdown 内容
- `metadata`: dict (可选)

写入 `.smc/drafts/{step}.md`

### read_draft(step) -> str
读取指定步骤的草稿内容。

### write_state(state_dict)
写入 `.smc/state.json`

### read_state() -> dict
读取当前状态。

### write_chunk(chunk_id, content)
- `chunk_id`: 分块ID
- content: 分块内容

写入 `.smc/chunks/{chunk_id}.md`

### read_chunk(chunk_id) -> str
读取指定分块。

### update_manifest(chunk_id, ref, summary)
更新 `.smc/chunks/manifest.json`

### read_manifest() -> dict
读取分块索引。

## 文件位置

所有文件存储在项目根目录的 `.smc/` 下。
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/storage.md
git commit -m "feat: add storage abstraction interface for content workflow"
```

---

### Task 2: 状态管理模块

**Files:**
- Create: `skills/content-workflow/state.md`

- [ ] **Step 1: 创建状态管理 skill**

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/state.md
git commit -m "feat: add state management skill"
```

---

### Task 3: Context 控制模块

**Files:**
- Create: `skills/content-workflow/context.md`

- [ ] **Step 1: 创建 Context 控制 skill**

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/context.md
git commit -m "feat: add context control skill"
```

---

### 阶段2：编排器

### Task 4: 编排器技能

**Files:**
- Create: `skills/content-workflow/orchestrator.md`

- [ ] **Step 1: 创建编排器 skill**

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/orchestrator.md
git commit -m "feat: add orchestrator skill"
```

---

### 阶段3：内容创作技能

### Task 5: 深度主题研究技能

**Files:**
- Create: `skills/content-workflow/skills/research.md`

- [ ] **Step 1: 创建研究技能**

```markdown
---
name: content-workflow-research
description: 深度主题研究 - 使用Deep Research进行多源信息收集
---

# 深度主题研究

## 输入

- 项目主题/关键词
- 研究目标（了解/对比/深度分析）
- 参考资源（可选）

## 过程

1. 使用 Deep Research 进行多源搜索
2. 收集权威来源（学术、官方、专业媒体）
3. 整理关键发现和数据点
4. 生成信息来源列表

## 输出结构

```markdown
# 研究报告

## 研究主题
[主题名称]

## 研究目标
[目标描述]

## 关键发现
1. [发现1]
2. [发现2]
...

## 数据支撑
- [数据点1] - [来源]
- [数据点2] - [来源]

## 信息来源
1. [来源1](url) - 权威度: 高/中/低
2. [来源2](url) - 权威度: 高/中/低

## 待深入问题
- [问题1]
- [问题2]
```

## Context 控制

- 完整报告存入 `01-research.md`
- 摘要（≤200字）存入 `state.json summaries.research`
- 大内容自动分块
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/research.md
git commit -m "feat: add research skill"
```

---

### Task 6: 制作优化大纲技能

**Files:**
- Create: `skills/content-workflow/skills/outline.md`

- [ ] **Step 1: 创建大纲技能**

```markdown
---
name: content-workflow-outline
description: 制作优化大纲 - 基于研究结果生成结构化大纲
---

# 制作优化大纲

## 输入

- 研究报告 (`01-research.md`)
- 用户对大纲的偏好（可选）

## 过程

1. 分析研究内容，提取核心主题
2. 设计章节结构
3. 为每个章节生成摘要和关键词
4. 优化整体逻辑 flow

## 输出结构

```markdown
# 内容大纲

## 主题
[内容主题]

## 目标读者
[目标描述]

## 核心观点
1. [观点1]
2. [观点2]
...

## 章节结构

### 第一章: [标题]
- 章节摘要: [50字以内]
- 关键词: [关键词1], [关键词2]
- 主要内容点:
  - [点1]
  - [点2]

### 第二章: [标题]
...

## 内容亮点
- [亮点1]
- [亮点2]

## 预计字数
[字数范围]
```

## Context 控制

- 完整大纲存入 `02-outline.md`
- 摘要存入 `state.json summaries.outline`
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/outline.md
git commit -m "feat: add outline skill"
```

---

### Task 7: 撰稿协作技能（含去AI味）

**Files:**
- Create: `skills/content-workflow/skills/drafting.md`

- [ ] **Step 1: 创建撰稿技能**

```markdown
---
name: content-workflow-drafting
description: 撰稿协作 - 按大纲生成初稿，嵌入式去AI味
---

# 撰稿协作

## 输入

- 内容大纲 (`02-outline.md`)
- 去AI味要求

## 去AI味策略（嵌入式）

撰稿时即注入人味，不事后弥补：

### 句式多样性
- 长短句交替
- 避免规整的排比句
- 自然的口语化插入语

### 词汇多样性
- 同义词替换（"非常"→"简直"、"相当"）
- 避免重复使用相同连接词
- 具体词汇替代泛化词汇

### 节奏控制
- 段落长短变化
- 适当留白
- 制造悬念/反差

### 表达个性
- 添加个人视角/感受
- 使用具体案例代替抽象论述
- 适度幽默或情感共鸣

## 输出结构

```markdown
# [文章标题]

## 元信息
- 关键词: [关键词列表]
- 目标读者: [读者描述]
- 字数: [字数]

## 正文

[使用去AI味策略撰写的完整内容]

## 创作笔记
- 引用来源: [列表]
- 个人风格注入点: [说明]
```

## Context 控制

- 完整草稿存入 `03-draft.md`
- 摘要存入 `state.json summaries.draft`
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/drafting.md
git commit -m "feat: add drafting skill with embedded humanization"
```

---

### Task 8: 编辑修改技能

**Files:**
- Create: `skills/content-workflow/skills/edit.md`

- [ ] **Step 1: 创建编辑技能**

```markdown
---
name: content-workflow-edit
description: 编辑修改 - 语法检查、用词优化、逻辑强化
---

# 编辑修改

## 输入

- 草稿 (`03-draft.md`)
- 编辑重点（可指定）

## 编辑维度

### 1. 语法检查
- 主谓一致
- 标点规范
- 句式完整

### 2. 用词优化
- 消除重复用词
- 精准词汇选择
- 冗余表达精简

### 3. 逻辑强化
- 段落衔接
- 论据支撑
- 结论推导

### 4. 表达提升
- 句式多样化
- 关键信息突出
- 阅读体验优化

## 输出结构

```markdown
# [文章标题] (编辑后)

## 编辑报告
- 修改总数: N处
- 主要修改:
  - [修改1]
  - [修改2]

## 正文（编辑后）
[修改后的完整内容]
```

## Context 控制

- 编辑后内容存入 `04-edit.md`
- 摘要存入 `state.json summaries.edit`
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/edit.md
git commit -m "feat: add edit skill"
```

---

### Task 9: 信息验证审核技能

**Files:**
- Create: `skills/content-workflow/skills/verify.md`

- [ ] **Step 1: 创建验证技能**

```markdown
---
name: content-workflow-verify
description: 信息验证审核 - 事实核查、来源追溯、数据验证
---

# 信息验证审核

## 输入

- 编辑后内容 (`04-edit.md`)
- 研究阶段的来源列表

## 验证维度

### 1. 事实核查
- 核心论点是否有数据支撑
- 引用数据是否准确
- 时间、地点、人物是否正确

### 2. 来源追溯
- 引用来源是否权威
- 来源是否为最新
- 是否有断章取义

### 3. 数据验证
- 统计数据来源
- 百分比计算
- 趋势判断

### 4. 逻辑一致性
- 论点论据是否匹配
- 是否有逻辑漏洞
- 结论是否过度推断

## 输出结构

```markdown
# 验证报告

## 验证结果
- 总计声明: N项
- 已验证: N项
- 待核实: N项
- 存在问题: N项

## 问题列表
1. [问题描述] - [建议修改]
2. ...

## 核实声明
- ✅ [已核实的声明]
- ⚠️ [待核实的声明]
- ❌ [存在问题]

## 最终建议
[综合评估和建议]
```

## Context 控制

- 验证报告存入 `05-verify.md`
- 摘要存入 `state.json summaries.verify`
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/verify.md
git commit -m "feat: add verify skill"
```

---

### Task 10: SEO优化技能

**Files:**
- Create: `skills/content-workflow/skills/seo.md`

- [ ] **Step 1: 创建SEO技能**

```markdown
---
name: content-workflow-seo
description: SEO优化 - 关键词布局、标题优化、元描述生成
---

# SEO优化

## 输入

- 已验证内容 (`05-verify.md`)
- 目标关键词（可指定）

## 优化维度

### 1. 关键词布局
- 标题含关键词
- 首段出现关键词
- 关键词密度: 1-2%
- 语义相关词覆盖

### 2. 标题优化
- H1: 核心关键词 + 吸引眼球
- H2/H3: 长尾关键词
- 标题长度: 30-60字符

### 3. 元信息
- Meta描述: 150-160字符
- 关键词标签
- URL结构建议

### 4. 内容结构
- 段落清晰
- 列表/表格使用
- 图片Alt文本

## 输出结构

```markdown
# SEO优化报告

## 关键词分析
- 核心关键词: [关键词]
- 长尾关键词: [列表]
- 关键词密度: [百分比]

## 标题优化
- H1: [优化后标题]
- H2: [列表]
- H3: [列表]

## Meta信息
- Meta描述: [150-160字符]
- 关键词: [标签列表]

## 技术建议
- [建议1]
- [建议2]

## 优化后内容
[应用SEO优化的内容]
```

## Context 控制

- 优化报告存入 `06-seo.md`
- 摘要存入 `state.json summaries.seo`
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/seo.md
git commit -m "feat: add SEO skill"
```

---

### Task 11: 去AI味技能

**Files:**
- Create: `skills/content-workflow/skills/humanize.md`

- [ ] **Step 1: 创建去AI味技能**

```markdown
---
name: content-workflow-humanize
description: 去AI味 - 检测和改写AI生成内容，使其更有人味
---

# 去AI味

## 输入

- SEO优化后内容 (`06-seo.md`)
- 人化程度（轻度/中度/重度）

## AI味检测指标

### 句式特征
- 句式过于规整
- 短句连续重复
- 连接词过度使用 (因此、然而、此外)

### 词汇特征
- 重复用词
- 泛化表达 (非常、十分、相当)
- 缺乏具体词汇

### 结构特征
- 完美的对称结构
- 机械的过渡
- 过于流畅/通顺

## 改写策略

### 轻度
- 调整句式长度
- 替换过度使用的词汇
- 添加自然过渡

### 中度
- 重写AI感强的段落
- 添加个人化表达
- 引入具体案例

### 重度
- 保留核心信息，完全重写表达
- 注入个人风格
- 添加情感元素

## 输出结构

```markdown
# 去AI味报告

## 检测结果
- AI味得分: [0-100]
- 主要问题:
  - [问题1]
  - [问题2]

## 改写统计
- 总改写处: N处
- 句式调整: N处
- 词汇替换: N处
- 段落重写: N处

## 改写后内容
[去AI味后的内容]

## 风格说明
[说明本次人化处理的要点]
```

## Context 控制

- 改写报告存入 `07-humanize.md`
- 摘要存入 `state.json summaries.humanize`
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/humanize.md
git commit -m "feat: add humanize skill"
```

---

### Task 12: 多格式适配技能

**Files:**
- Create: `skills/content-workflow/skills/format.md`

- [ ] **Step 1: 创建格式适配技能**

```markdown
---
name: content-workflow-format
description: 多格式适配 - 输出Markdown格式
---

# 多格式适配

## 输入

- 去AI味后内容 (`07-humanize.md`)
- 目标格式

## 输出格式

当前版本: 纯 Markdown

### Markdown 规范
- 标题层级: H1 > H2 > H3
- 列表: 有序/无序混用
- 强调: 粗体/斜体适度
- 引用: 用于重要引述
- 代码块: 用于技术内容

## 输出结构

```markdown
# 最终内容

[标准Markdown格式]

## 附录
- 字数统计
- 关键词分布
```

## Context 控制

- 最终Markdown存入 `08-format.md`
- 摘要存入 `state.json summaries.format`
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/format.md
git commit -m "feat: add format skill"
```

---

### Task 13: 平台适配技能

**Files:**
- Create: `skills/content-workflow/skills/platform.md`

- [ ] **Step 1: 创建平台适配技能**

```markdown
---
name: content-workflow-platform
description: 平台适配 - 平台特定格式调整
---

# 平台适配

## 输入

- Markdown内容 (`08-format.md`)
- 目标平台

## 支持平台

| 平台 | 特点 |
|------|------|
| 微信公众号 | 标题格式、字数控制、排版 |
| 知乎 | 问答适配、结构化 |
| 微博/小红书 | 短内容、emoji |
| 独立博客 | 完整内容、SEO |

## 适配维度

### 1. 标题党适配（可选）
- 吸引点击的标题
- 副标题补充

### 2. 字数控制
- 长文/短文转换
- 精华版/完整版

### 3. 平台特定格式
- 平台配图建议
- 标签格式
- 互动引导

## 输出结构

```markdown
# 平台适配报告

## 适配平台
[平台名称]

## 标题
[优化后标题]

## 字数
- 原文字数: N
- 适配后字数: N

## 正文
[平台适配后的内容]

## 平台建议
- 配图: [建议]
- 标签: [列表]
- 互动引导: [文本]
```

## Context 控制

- 适配报告存入 `09-platform.md`
- 摘要存入 `state.json summaries.platform`
```

- [ ] **Step 2: Commit**

```bash
git add skills/content-workflow/skills/platform.md
git commit -m "feat: add platform skill"
```

---

### 阶段4：整合

### Task 14: README 和打包

**Files:**
- Create: `skills/content-workflow/README.md`
- Create: `skills/content-workflow/SKILL.md`

- [ ] **Step 1: 创建总览 README**

```markdown
# 内容创作工作流

一套完整的内容创作工作流技能，支持从研究到发布的全流程。

## 模块

1. **编排器** - 任务分解和流程调度
2. **深度主题研究** - Deep Research 多源信息收集
3. **制作优化大纲** - 结构化大纲生成
4. **撰稿协作** - 含嵌入式去AI味
5. **编辑修改** - 语法、用词、逻辑优化
6. **信息验证审核** - 事实核查、来源追溯
7. **SEO优化** - 关键词布局、元信息优化
8. **去AI味** - 检测和改写
9. **多格式适配** - Markdown 输出
10. **平台适配** - 多平台发布

## 快速开始

`/content-workflow`

## 状态存储

`.smc/` - 所有中间结果和草稿
```

- [ ] **Step 2: 创建入口 SKILL.md**

```markdown
---
name: content-workflow
description: 内容创作工作流 - 从研究到发布的全流程技能
---

加载编排器技能进行内容创作。
```

- [ ] **Step 3: Commit**

```bash
git add skills/content-workflow/README.md skills/content-workflow/SKILL.md
git commit -m "feat: add content-workflow README and entry point"
```

---

## 自检清单

- [ ] 所有 skill 文件已创建
- [ ] 每个 skill 都有 YAML frontmatter
- [ ] 所有步骤都有输出结构定义
- [ ] Context 控制机制已明确
- [ ] 存储位置符合 `.smc/` 结构
- [ ] Commit 历史完整

---

**Plan complete and saved to `docs/superpowers/plans/2026-03-27-content-workflow-plan.md`**

**Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**
