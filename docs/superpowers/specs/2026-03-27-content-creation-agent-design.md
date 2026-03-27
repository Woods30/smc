# 内容创作 Agent 技能体系 — 架构设计

## 1. 整体架构

```
orchestrator/           # 总入口，分发到 article 或 social
├── article/            # 长文章流程编排
└── social/             # 社交内容流程编排

子技能（独立，可被组合调用）:
├── research/           # 深度研究（第三方槽：Deep Research / Tavily / Firecrawl）
├── outline/            # 大纲生成
├── drafting/           # 撰稿（含嵌入式去AI味）
├── edit/               # 编辑修改
├── verify/             # 信息验证
├── seo/                # SEO优化
├── humanize/           # 去AI味（第三方槽：StealthWriter / UndetectableAI）
├── format/             # 格式适配
└── platform/           # 平台发布
```

**入口命令**：`/article` 和 `/social`

---

## 2. article 流程

```
research → outline → drafting → edit → verify → seo → humanize → format → platform
```

- 研究环节**必须**触发，调用第三方 Deep Research 或用户配置的第三方服务
- 去AI味**嵌入式**注入在 drafting 环节
- 全自动端到端，三节点确认：①开始研究前 ②大纲确认 ③终稿发布

---

## 3. social 流程

```
research(轻) → outline → drafting → humanize → platform
```

- 社交内容走独立轻量流程
- research 为轻量版（基于用户提供素材或快速搜索）
- 直接输出平台适配文案

---

## 4. 第三方技能槽机制

每个支持第三方的技能有一个**技能槽**，可通过配置切换：

```json
{
  "plugins": {
    "research": "deep-research" | "tavily" | "firecrawl",
    "humanize": "stealthwriter" | "undetectableai"
  }
}
```

- 默认使用官方能力（Deep Research、内置去AI味提示词）
- 用户配置后自动切换到第三方
- 第三方 API key 通过环境变量或配置文件注入

---

## 5. 存储结构

```
project/.smc/
├── state.json           # 统一状态
├── chunks/              # 长内容分块
│   └── manifest.json
└── drafts/              # 各环节草稿
    ├── 01-research.md
    ├── 02-outline.md
    ├── ...
```

每个项目独立文件夹，状态随项目持久化，支持 git 版本化。

---

## 6. article 和 social 独立性

- 两者**完全独立**，无依赖关系
- 用户可单独使用 `/article` 或 `/social`
- 不存在"从长文章拆解出社交内容"的自动流程
- 社交内容有自己完整的研究→撰稿→发布链路

---

## 7. 关键约束

| 原则 | 说明 |
|------|------|
| article 必须先研究 | 不可跳过 |
| 去AI味嵌入式 | drafting 环节已注入，不依赖独立环节 |
| 第三方可插拔 | 研究和去AI味支持切换实现 |
| 完全独立 | article 和 social 无依赖关系 |
| 端到端自动化 | 用户只给主题，agent 完成全流程 |

---

## 8. 平台支持（social / platform）

支持国内 + 海外全平台：
- 国内：微信公众号、微博、小红书、知乎、抖音/快手文案
- 海外：Twitter/X、LinkedIn、Instagram、Facebook、Threads

---

## 9. 适用场景

| 入口 | 场景 |
|------|------|
| `/article` | 长文章/博客写作（2000+ 字） |
| `/social` | 社交媒体内容批量生产 |

---

## 10. 与 Superpowers 的关系

参考 Superpowers 的技能编排模式：
- 每个技能一个独立 SKILL.md
- orchestrator 做分发调度
- 子技能可被不同流程组合调用
- 通过 hooks 实现自动触发
