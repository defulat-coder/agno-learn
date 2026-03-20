# structured_output.py — 实现原理分析

> 源文件：`cookbook/90_models/deepseek/structured_output.py`

## 概述

**DeepSeek + `output_schema` + `use_json_mode=True`**；`MovieScript` 与 `description="You help people write movie scripts."`。`DeepSeek` 标注 `supports_native_structured_outputs: bool = False`（`deepseek.py` L33-34），JSON 依赖提示与解析。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `DeepSeek(id="deepseek-chat")` | |
| `description` | `You help people write movie scripts.` | 字面量 |
| `output_schema` | `MovieScript` | |
| `use_json_mode` | `True` | |

## System Prompt 组装

### 还原后的完整 System 文本

```text
You help people write movie scripts.

（以及 get_json_output_prompt 等动态段）
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["use_json_mode + schema"] --> B["【关键】JSON 模式请求"]
    B --> C["解析 MovieScript"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/_messages.py` | `# 3.3.15` | JSON 提示 |
| `agno/models/deepseek/deepseek.py` | `supports_native_structured_outputs` | False |
