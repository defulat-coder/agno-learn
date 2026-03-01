# image_input.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/structured_io/image_input.py`

## 概述

本示例展示 Agno Workflow **图像输入处理**：通过 `workflow.print_response(images=[Image(...)])` 将图像传入 Workflow，Step 1（Image Analyzer Agent）分析图像内容，Step 2（News Researcher Agent + WebSearch）基于分析结果搜索相关新闻，实现图像理解与网络搜索的串联工作流。

**核心配置：**

| 配置 | 值 | 说明 |
|------|------|------|
| `workflow.print_response(images=[...])` | `Image(url=...)` | 图像作为 Workflow 输入 |
| 模型 | `gpt-4o` | 多模态模型支持图像分析 |

## 核心组件解析

```python
from agno.media import Image

media_workflow = Workflow(
    steps=[analysis_step, research_step],
    db=SqliteDb(db_file="tmp/workflow.db"),
)

media_workflow.print_response(
    input="Please analyze this image and find related news",
    images=[
        Image(url="https://upload.wikimedia.org/wikipedia/commons/0/0c/GoldenGateBridge-001.jpg")
    ],
    markdown=True,
)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/media/__init__.py` | `Image` | 图像媒体对象 |
| `agno/workflow/workflow.py` | `Workflow.print_response(images=...)` | 图像输入支持 |
