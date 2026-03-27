# 内容创作工作流技能 - 架构设计

## 1. 核心架构

```
内容创作编排器 (orchestrator)
├── 深度主题研究技能
├── 制作优化大纲技能
├── 撰稿协作技能 (含去AI味)
├── 编辑修改技能
├── 信息验证审核技能
├── SEO优化技能
├── 去AI味技能
├── 多格式适配技能
└── 平台适配技能
```

**交互模式**: 共享状态（`.smc/state.json`），支持任意步骤跳转。

---

## 2. 存储结构

```
project/
├── .smc/
│   ├── state.json          # 统一状态（当前步骤、整体进度）
│   ├── chunks/             # 分块存储
│   │   └── manifest.json  # 分块索引
│   └── drafts/             # 各步骤草稿（Markdown）
│       ├── 01-research.md
│       ├── 02-outline.md
│       ├── 03-draft.md
│       ├── 04-edit.md
│       ├── 05-verify.md
│       ├── 06-seo.md
│       ├── 07-humanize.md
│       ├── 08-format.md
│       └── 09-platform.md
```

---

## 3. Context 控制机制

| 策略 | 触发条件 | 实现 |
|------|----------|------|
| 分块 | 单个内容 > 2000 tokens | 自动拆分，存入 `chunks/` |
| 摘要 | 每个检查点完成 | 摘要存入 `state.json` |
| 索引 | 分块时自动更新 | `chunks/manifest.json` |

---

## 4. 模块职责

### 4.1 编排器 (Orchestrator)
- 负责任务分解和流程调度
- 维护 `state.json` 中的当前步骤和进度
- 支持用户跳转任意步骤

### 4.2 深度主题研究
- 使用 Deep Research 进行多源信息收集
- 输出：研究摘要、关键发现、信息来源列表
- 状态：`drafts/01-research.json`

### 4.3 制作优化大纲
- 基于研究结果生成结构化大纲
- 支持多级标题、章节摘要、关键词标注
- 状态：`drafts/02-outline.json`

### 4.4 撰稿协作（含去AI味）
- 按大纲生成初稿
- **嵌入式去AI味**：句式变化、词汇多样性、节奏控制
- 状态：`drafts/03-draft.json`

### 4.5 编辑修改
- 语法检查、用词优化、逻辑强化
- 状态：`drafts/04-edit.json`

### 4.6 信息验证审核
- 事实核查、来源追溯、数据验证
- 状态：`drafts/05-verify.json`

### 4.7 SEO优化
- 关键词布局、标题优化、元描述生成
- 状态：`drafts/06-seo.json`

### 4.8 去AI味
- 独立检测 + 改写模块（可选，作为二次打磨）
- 状态：`drafts/07-humanize.json`

### 4.9 多格式适配
- 输出 Markdown
- 状态：`drafts/08-format.json`

### 4.10 平台适配
- 平台特定格式调整（标题党适配、字数控制等）
- 状态：`drafts/09-platform.json`

---

## 5. AI模型分配

| 模块 | 模型 | 说明 |
|------|------|------|
| 深度主题研究 | Deep Research | 专业研究能力 |
| 制作优化大纲 | Claude | 结构化思考 |
| 撰稿协作 | Claude | 写作质量 |
| 编辑修改 | Claude | 精细编辑 |
| 信息验证审核 | Claude | 事实核查 |
| SEO优化 | Claude | 内容优化 |
| 去AI味 | Claude | 语言多样性 |
| 多格式适配 | Claude | 格式转换 |
| 平台适配 | Claude | 平台适配 |

---

## 6. 状态流转

```
[开始] → 研究 → 大纲 → 撰稿 → 编辑 → 验证 → SEO → 去AI味 → 格式 → 平台 → [完成]
         ↓       ↓      ↓       ↓       ↓       ↓       ↓        ↓       ↓
       drafts  drafts  drafts  drafts  drafts  drafts  drafts   drafts   drafts
```

用户可随时跳转到任意步骤，编排器从该步骤重新执行并更新后续状态。

---

## 7. 设计约束总结

| 维度 | 选择 |
|------|------|
| 存储 | 文件存储（JSON） |
| Context 控制 | 分块 + 摘要压缩 |
| 模块交互 | 共享状态（灵活跳转） |
| AI模型 | 按任务分配 |
| 去AI味 | 撰稿时嵌入式注入 |
| 输出格式 | 纯文本（Markdown） |
| 架构风格 | 技能编排器模式 |
