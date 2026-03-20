# prompt_caching.py — 实现原理分析

> 源文件：`cookbook/90_models/anthropic/prompt_caching.py`

## 概述

本示例展示 **`cache_system_prompt=True`** 与 **`system_message`** 从本地大文本文件加载：首轮写入 prompt cache，次轮命中以降低延迟与费用。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Claude(id="claude-sonnet-4-20250514", cache_system_prompt=True)` | 系统 prompt 缓存 |
| `system_message` | 文件 `system_prompt.txt` 全文 | 早退分支 L129–152 |
| `markdown` | `True` | 若仍走默认会加 Markdown；此处自定义 system_message |

## 核心组件解析

### system_message 早退

`get_system_message()`：若 `agent.system_message` 非 `None`，直接返回该内容（可经 `resolve_in_context` 格式化），**不再**拼装默认 instructions 块（见 L129–152）。

### 运行机制与因果链

1. **路径**：`agent.run` 两次；metrics 中 `cache_write_tokens` / `cache_read_tokens` 反映缓存行为。
2. **副作用**：无 db；Anthropic 侧缓存状态。
3. **定位**：与 `prompt_caching_extended.py`（延长 TTL）对照。

## System Prompt 组装

| 组成部分 | 状态 |
|---------|------|
| 自定义 `system_message` | 是，整文件正文 |

### 还原后的完整 System 文本

正文等于下载后的 `system_prompt.txt`。仓库内路径为同目录 `system_prompt.txt`；远程内容为 `agno-public` 上 `system_promt.txt`（注意上游文件名拼写）。**静态分析无法嵌入完整正文**，请打开本地文件或运行 `txt_path.read_text()` 获取。

### 验证

打印 `get_system_message()` 返回的 `Message.content`。

## 完整 API 请求

`claude.py` 中 `cache_system_prompt` 为真时，`system` 带 `cache_control`（见 L531–537）。

## Mermaid 流程图

```mermaid
flowchart TD
    A["system_message 大文本"] --> B["【关键】cache_control 写缓存"]
    B --> C["第二次 run 读缓存"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/_messages.py` | L129–152 | 自定义 system 早退 |
| `agno/models/anthropic/claude.py` | `_prepare_request_kwargs` L530–539 | cache_control |
