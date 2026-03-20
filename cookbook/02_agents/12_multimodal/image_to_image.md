# image_to_image.py — 实现原理分析

> 源文件：`cookbook/02_agents/12_multimodal/image_to_image.py`

## 概述

本示例展示 **`FalTools` 图生图**：`OpenAIResponses` + `tools=[FalTools()]`，`instructions` 强制调用 `image_to_image` 并原样返回 URL。

**核心配置一览：**

| 配置项 | 值 |
|--------|-----|
| `id` / `name` | `"image-to-image"` / `"Image to Image Agent"` |
| `model` | `OpenAIResponses(id="gpt-5.2")` |
| `tools` | `[FalTools()]` |
| `instructions` | 四条（见 `.py`） |

## 运行机制与因果链

模型根据用户 prompt + 参考图 URL 调 Fal API；**不**把结果误标为 markdown 图片链。

## Mermaid 流程图

```mermaid
flowchart TD
    U["文本+参考 URL"] --> F["【关键】FalTools.image_to_image"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/fal` | `FalTools` |
