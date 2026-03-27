# 内容创作工作流

一套完整的内容创作工作流技能，支持从研究到发布的全流程。

## 模块

| 技能 | 说明 |
|------|------|
| **编排器** (orchestrator) | 任务分解和流程调度，主入口 |
| **存储接口** (storage) | 统一的存储抽象接口 |
| **状态管理** (state) | 工作流状态流转管理 |
| **Context控制** (context) | 分块 + 摘要压缩 |
| **深度主题研究** (research) | Deep Research 多源信息收集 |
| **制作优化大纲** (outline) | 结构化大纲生成 |
| **撰稿协作** (drafting) | 含嵌入式去AI味 |
| **编辑修改** (edit) | 语法、用词、逻辑优化 |
| **信息验证审核** (verify) | 事实核查、来源追溯 |
| **SEO优化** (seo) | 关键词布局、元信息优化 |
| **去AI味** (humanize) | 检测和改写 |
| **多格式适配** (format) | Markdown 输出 |
| **平台适配** (platform) | 多平台发布 |

## 安装

### 方式一：本地 skills.paths（开发/测试用）

在 `~/.claude/settings.json` 中添加：

```json
{
  "skills": {
    "paths": [
      "/path/to/content-workflow/skills"
    ]
  }
}
```

### 方式二：作为插件发布

1. 将此仓库打包分发
2. 用户通过插件系统安装

## 使用

1. 加载编排器技能：`/content-workflow` 或 `开始内容创作`
2. 选择「开始新项目」或「继续」
3. 按流程逐步完成各步骤
4. 支持随时跳转任意步骤

## 状态存储

所有中间结果保存在 `.smc/` 目录：

```
.smc/
├── state.json           # 统一状态
├── chunks/              # 分块内容
│   └── manifest.json    # 索引
└── drafts/              # 各步骤草稿
    ├── 01-research.md
    ├── 02-outline.md
    └── ...
```

## 工作流

```
研究 → 大纲 → 撰稿 → 编辑 → 验证 → SEO → 去AI味 → 格式 → 平台
```

支持任意步骤跳转，状态自动保存。
