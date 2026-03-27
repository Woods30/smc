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
