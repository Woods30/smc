# Content Workflow — 子技能基础设施实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 建立内容创作工作流的原子技能层和插件槽机制，9个独立子技能可被 article/social 工作流组合调用。

**Architecture:** 每个子技能是独立 SKILL.md，通过 state.json 共享状态。插件槽通过配置文件选择第三方实现（research/humanize）。

**Tech Stack:** SKILL.md（Markdown 技能定义）+ JSON 配置文件 + .smc/ 存储

---

## 文件结构

```
content-workflow/
├── plugin.json                  # 插件定义：/article, /social 入口
├── skills/
│   ├── orchestrator/
│   │   └── SKILL.md            # 总分发器
│   ├── research/
│   │   └── SKILL.md            # 深度研究（支持第三方槽）
│   ├── outline/
│   │   └── SKILL.md            # 大纲生成
│   ├── drafting/
│   │   └── SKILL.md            # 撰稿（含嵌入式去AI味）
│   ├── edit/
│   │   └── SKILL.md            # 编辑修改
│   ├── verify/
│   │   └── SKILL.md            # 信息验证
│   ├── seo/
│   │   └── SKILL.md            # SEO优化
│   ├── humanize/
│   │   └── SKILL.md            # 去AI味（支持第三方槽）
│   ├── format/
│   │   └── SKILL.md            # 格式适配
│   └── platform/
│       └── SKILL.md            # 平台发布
└── config.json                 # 插件槽配置

project/.smc/                   # 用户项目中的状态存储
├── state.json
├── chunks/
│   └── manifest.json
└── drafts/
```

---

## Task 1: 创建目录结构和 plugin.json

**Files:**
- Create: `content-workflow/plugin.json`
- Create: `content-workflow/config.json`

- [ ] **Step 1: Create plugin.json**

```json
{
  "name": "content-workflow",
  "version": "1.0.0",
  "description": "内容创作工作流 - 长文章和社交内容全流程",
  "entryPoints": [
    {
      "command": "/article",
      "description": "开始长文章创作流程"
    },
    {
      "command": "/social",
      "description": "开始社交内容创作流程"
    }
  ],
  "skills": [
    "skills/orchestrator",
    "skills/article",
    "skills/social",
    "skills/research",
    "skills/outline",
    "skills/drafting",
    "skills/edit",
    "skills/verify",
    "skills/seo",
    "skills/humanize",
    "skills/format",
    "skills/platform"
  ]
}
```

- [ ] **Step 2: Create config.json with plugin slots**

```json
{
  "plugins": {
    "research": {
      "provider": "deep-research",
      "apiKeyEnv": "DEEP_RESEARCH_API_KEY"
    },
    "humanize": {
      "provider": "stealthwriter",
      "apiKeyEnv": "STEALTHWRITER_API_KEY"
    }
  },
  "storage": {
    "basePath": ".smc"
  }
}
```

- [ ] **Step 3: Create directory structure**

```bash
mkdir -p content-workflow/skills/{orchestrator,research,outline,drafting,edit,verify,seo,humanize,format,platform}
mkdir -p content-workflow/.claude-plugin
```

- [ ] **Step 4: Commit**

```bash
git add content-workflow/plugin.json content-workflow/config.json
git commit -m "feat: add plugin.json and config.json with research/humanize plugin slots"
```

---

## Task 2: 创建 orchestrator 分发器

**Files:**
- Create: `content-workflow/skills/orchestrator/SKILL.md`
- Create: `content-workflow/skills/orchestrator/article/SKILL.md`
- Create: `content-workflow/skills/orchestrator/social/SKILL.md`

- [ ] **Step 1: Write orchestrator SKILL.md**

```markdown
# Orchestrator — 内容创作工作流总入口

## 触发条件

当用户输入包含 `/article` 或 `/social` 时触发。

## 分发逻辑

检查用户意图：
- `/article` → 调用 article/SKILL.md
- `/social` → 调用 social/SKILL.md

如果用户只说"写文章"但未指定类型，询问用户选择 article 还是 social。

## 流程说明

- `/article`: 完整长文章流程（研究→大纲→撰稿→编辑→验证→SEO→去AI味→格式→发布）
- `/social`: 社交内容流程（研究→大纲→撰稿→去AI味→平台适配）

## 状态初始化

创建项目状态文件 `.smc/state.json`：
```json
{
  "type": null,
  "currentStep": null,
  "createdAt": "<timestamp>",
  "updatedAt": "<timestamp>"
}
```

## 第三方插件槽

通过 `config.json` 配置：
- research: deep-research / tavily / firecrawl
- humanize: stealthwriter / undetectableai

读取配置并传递给对应技能。
```

- [ ] **Step 2: Write article SKILL.md stub**

```markdown
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
```

- [ ] **Step 3: Write social SKILL.md stub**

```markdown
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
```

- [ ] **Step 4: Commit**

```bash
git add content-workflow/skills/orchestrator/
git commit -m "feat: add orchestrator dispatcher with article/social entry points"
```

---

## Task 3: 创建 research 技能（含第三方槽）

**Files:**
- Create: `content-workflow/skills/research/SKILL.md`

- [ ] **Step 1: Write research SKILL.md**

```markdown
# Research — 深度主题研究

## 触发条件

被 article 或 social 工作流调用。article 必须触发。

## 输入

- 主题（用户提供的文章/内容主题）
- 配置（从 config.json 读取 research.provider）

## 流程

### Deep Research 模式（默认 / deep-research）

1. 使用官方 Deep Research 能力搜索主题相关资料
2. 收集来源：网页、文章、数据源
3. 提取关键发现、核心论点、支持数据
4. 输出来源列表（URL + 摘要）

### Tavily 模式（provider: tavily）

1. 调用 Tavily API 搜索主题
2. 聚合搜索结果
3. 提取结构化信息

### Firecrawl 模式（provider: firecrawl）

1. 使用 Firecrawl 爬取相关网页
2. 提取页面核心内容
3. 去重和内容质量过滤

## 输出

保存到 `.smc/drafts/01-research.md`：

```markdown
# 研究报告

## 主题
<用户主题>

## 关键发现
- <发现1>
- <发现2>
...

## 核心数据/统计
- <数据1>
- <数据2>
...

## 信息来源
1. <来源1> — <URL>
2. <来源2> — <URL>
...

## 研究时间
<timestamp>
```

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "research",
  "research": {
    "completedAt": "<timestamp>",
    "provider": "<used provider>",
    "sourceCount": <number>
  }
}
```
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/research/SKILL.md
git commit -m "feat: add research skill with deep-research/tavily/firecrawl plugin slots"
```

---

## Task 4: 创建 outline 技能

**Files:**
- Create: `content-workflow/skills/outline/SKILL.md`

- [ ] **Step 1: Write outline SKILL.md**

```markdown
# Outline — 制作优化大纲

## 触发条件

被 article 或 social 工作流调用，在 research 之后。

## 输入

- `.smc/drafts/01-research.md`（研究结果）
- 工作流类型（article / social）

## article 大纲要求

- 多级标题结构（H1 / H2 / H3）
- 每个 H2 下有 2-3 句章节摘要
- 标注关键章节的关键词
- 结尾有总结段落规划
- 字数估算：每章节 200-500 字

## social 大纲要求

- 简洁单层结构
- 每段 1-3 句话
- 开头抓住注意力（Hook）
- 中间核心信息
- 结尾 CTA 或互动引导
- 字数：微博(140-280) / 小红书(300-800) / 公众号(500-1500)

## 输出

保存到 `.smc/drafts/02-outline.md`：

```markdown
# 大纲 — <主题>

## 元信息
- 工作流类型: <article|social>
- 目标字数: <估算>
- 预计阅读时间: <N分钟>

## 标题候选
<3个标题选项>

## 正文结构

### H1: <主标题>
#### H2: <章节1标题>
<2-3句摘要>
##### H3: <要点1>
##### H3: <要点2>

### H2: <章节2标题>
...

## 关键词标注
- <关键词1>: H2/H3 位置
- <关键词2>: H2/H3 位置
```

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "outline",
  "outline": {
    "completedAt": "<timestamp>",
    "type": "article|social",
    "wordCount": <estimate>
  }
}
```
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/outline/SKILL.md
git commit -m "feat: add outline skill for article and social workflows"
```

---

## Task 5: 创建 drafting 技能（含嵌入式去AI味）

**Files:**
- Create: `content-workflow/skills/drafting/SKILL.md`

- [ ] **Step 1: Write drafting SKILL.md**

```markdown
# Drafting — 撰稿协作（含嵌入式去AI味）

## 触发条件

被 article 或 social 工作流调用，在 outline 之后。

## 输入

- `.smc/drafts/02-outline.md`（大纲）
- `.smc/drafts/01-research.md`（研究结果，可选）

## 去AI味嵌入式策略

撰稿时直接在生成过程中应用以下技术，避免最终稿有AI味：

### 句式变化
- 长短句交错，不规则节奏
- 主动语态优先，被动语态只在必要时使用
- 疑问句、感叹句适度穿插

### 词汇多样性
- 同义替换常见词（"首先"→"开篇"、"因此"→"于是"）
- 避免重复相同的连接词
- 行业术语自然融入

### 表达自然化
- 口语化短句穿插
- 个人视角代入（"我认为"、"实践中发现"）
- 具体案例替代抽象论述

### 段落节奏
- 避免"首先...其次...最后"公式化结构
- 段落长度不均匀
- 关键信息不在段落开头

## article 撰稿要求

1. 严格按大纲结构写作
2. 每个章节内容丰富，有深度
3. 引用研究数据时标注来源
4. 开头有 Hook，结尾有升华
5. 关键词自然融入全文

## social 撰稿要求

1. 开头必须抓眼球（前3句话决定用户是否继续）
2. 单个平台单篇，不做多平台合并
3. 根据平台调整语气（微博口语化、小红书生活化、公众号正式）

## 输出

保存到 `.smc/drafts/03-draft.md`：
- 完整正文内容
- 在文件头部标注元信息（字数、预计阅读时间）

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "drafting",
  "drafting": {
    "completedAt": "<timestamp>",
    "wordCount": <actual>,
    "humanizeApplied": true
  }
}
```
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/drafting/SKILL.md
git commit -m "feat: add drafting skill with embedded humanize strategy"
```

---

## Task 6: 创建 edit 技能

**Files:**
- Create: `content-workflow/skills/edit/SKILL.md`

- [ ] **Step 1: Write edit SKILL.md**

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/edit/SKILL.md
git commit -m "feat: add edit skill for article workflow"
```

---

## Task 7: 创建 verify 技能

**Files:**
- Create: `content-workflow/skills/verify/SKILL.md`

- [ ] **Step 1: Write verify SKILL.md**

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/verify/SKILL.md
git commit -m "feat: add verify skill for fact-checking"
```

---

## Task 8: 创建 seo 技能

**Files:**
- Create: `content-workflow/skills/seo/SKILL.md`

- [ ] **Step 1: Write seo SKILL.md**

```markdown
# SEO — SEO优化

## 触发条件

被 article 工作流调用，在 verify 之后。social 不调用此技能。

## 输入

- `.smc/drafts/05-verify.md`（验证后稿件）
- `.smc/drafts/02-outline.md`（大纲，含关键词标注）

## SEO 优化维度

### 关键词布局
- 标题（H1）必须包含主关键词
- 首段前100字必须出现主关键词
- H2/H3 包含关键词变体
- 全文关键词密度 1-2%
- 关键词自然分布，不堆砌

### 元信息优化
- Meta Title（50-60字符）
- Meta Description（150-160字符）
- URL 友好化（英文/拼音 slug）

### 内容结构
- 内部链接建议（锚文本）
- 图片 alt 文本（描述性）
- 段落长度控制（便于 NLP 解析）

## 输出

保存到 `.smc/drafts/06-seo.md`：
- SEO 优化后的完整稿件
- 包含元信息区块

```markdown
---
title: <SEO标题>
description: <Meta描述>
slug: <url-slug>
keywords: <关键词列表>
---

<完整正文>
```

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "seo",
  "seo": {
    "completedAt": "<timestamp>",
    "metaTitle": "<生成标题>",
    "metaDescription": "<生成描述>"
  }
}
```
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/seo/SKILL.md
git commit -m "feat: add seo skill for article optimization"
```

---

## Task 9: 创建 humanize 技能（含第三方槽）

**Files:**
- Create: `content-workflow/skills/humanize/SKILL.md`

- [ ] **Step 1: Write humanize SKILL.md**

```markdown
# Humanize — 去AI味

## 触发条件

1. 被 article 工作流在 seo 之后调用（最终抛光）
2. 被 social 工作流在 drafting 之后调用
3. 配置决定使用内置还是第三方

## 输入

- `.smc/drafts/06-seo.md`（article）或 `.smc/drafts/03-draft.md`（social）
- 配置（从 config.json 读取 humanize.provider）

## 内置模式（默认 / stealthwriter）

使用高级提示词工程进行去AI味改写：
- 深度句式重构
- 词汇多样化替换
- 人类写作特征注入（犹豫、强调、不完美）

### 改写原则
1. 保持原意 100%
2. 改变句式结构
3. 替换高频AI词汇
4. 增加人类写作痕迹

## 第三方模式（provider: undetectableai）

调用 UndetectableAI API 进行去AI味。

## 输出

保存到 `.smc/drafts/07-humanize.md`：
- 去AI味后的完整稿件

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "humanize",
  "humanize": {
    "completedAt": "<timestamp>",
    "provider": "<used provider>",
    "wordCountBefore": <before>,
    "wordCountAfter": <after>
  }
}
```
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/humanize/SKILL.md
git commit -m "feat: add humanize skill with stealthwriter/undetectableai plugin slots"
```

---

## Task 10: 创建 format 技能

**Files:**
- Create: `content-workflow/skills/format/SKILL.md`

- [ ] **Step 1: Write format SKILL.md**

```markdown
# Format — 多格式适配

## 触发条件

被 article 工作流调用，在 humanize 之后。

## 输入

- `.smc/drafts/07-humanize.md`（去AI味后稿件）

## 输出格式

### Markdown 输出（默认）
- 标准 Markdown 格式
- 正确的 H1/H2/H3 层级
- 有序/无序列表正确使用
- 代码块适当使用
- 链接和图片引用规范

### 各平台格式适配（调用 platform 技能）

不在此技能处理，由 platform 技能负责。

## 输出

保存到 `.smc/drafts/08-format.md`：
- 格式化后的标准 Markdown

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "format",
  "format": {
    "completedAt": "<timestamp>",
    "format": "markdown"
  }
}
```
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/format/SKILL.md
git commit -m "feat: add format skill for markdown output"
```

---

## Task 11: 创建 platform 技能

**Files:**
- Create: `content-workflow/skills/platform/SKILL.md`

- [ ] **Step 1: Write platform SKILL.md**

```markdown
# Platform — 平台适配与发布

## 触发条件

被 article 或 social 工作流调用，作为最终环节。

## 输入

- `.smc/drafts/08-format.md`（格式化稿件，article）
- `.smc/drafts/07-humanize.md`（social，直接来自 humanize）
- 目标平台列表

## 支持平台

### 国内
- 微信公众号：封面图建议、摘要（不含emoji）、字数控制
- 微博：140字限制、话题标签、@提及
- 小红书：emoji、hashtag、200-800字
- 知乎：专业语气、引用格式
- 抖音/快手：短文案、悬念式开头

### 海外
- Twitter/X：280字符、hashtag
- LinkedIn：专业语气、CTA
- Instagram：caption、hashtag墙
- Facebook：长文案、emoji
- Threads：短内容、互动引导

## 平台适配处理

1. 字数裁剪/扩展
2. 标题党适配（适度）
3. 表情符号处理
4. hashtag 格式转换
5. @提及和链接处理

## 输出

每个平台单独输出文件：
- `.smc/output/wechat-<slug>.md`
- `.smc/output/twitter-<slug>.md`
- `.smc/output/xiaohongshu-<slug>.md`
- ...

## 状态更新

更新 `.smc/state.json`：
```json
{
  "currentStep": "platform",
  "platform": {
    "completedAt": "<timestamp>",
    "outputs": ["wechat", "twitter", "xiaohongshu"]
  }
}
```

## 标记完成

当 platform 完成后，更新 state.json 标记整个工作流完成：
```json
{
  "status": "completed",
  "completedAt": "<timestamp>"
}
```
```

- [ ] **Step 2: Commit**

```bash
git add content-workflow/skills/platform/SKILL.md
git commit -m "feat: add platform skill for multi-platform output"
```

---

## 验证清单

**Spec 覆盖检查：**
- [ ] orchestrator 分发器 ✓
- [ ] article/social 入口 ✓
- [ ] 9个原子技能 ✓
- [ ] research 第三方槽（deep-research/tavily/firecrawl） ✓
- [ ] humanize 第三方槽（stealthwriter/undetectableai） ✓
- [ ] 嵌入式去AI味在 drafting ✓
- [ ] .smc/ 存储结构 ✓
- [ ] 状态更新机制 ✓

**Placeholder 扫描：**
- [ ] 无 "TBD" / "TODO" ✓
- [ ] 无 "后续实现" / "待补充" ✓
- [ ] 所有流程步骤有具体代码/描述 ✓

**类型一致性：**
- [ ] state.json 字段名统一 ✓
- [ ] drafts 文件命名一致（01-xx, 02-xx...） ✓
- [ ] plugin.json entryPoints 格式正确 ✓
