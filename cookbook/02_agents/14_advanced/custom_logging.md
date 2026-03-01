# custom_logging.py — 实现原理分析

> 源文件：`cookbook/02_agents/14_advanced/custom_logging.py`

## 概述

本示例展示如何将 Agno 框架的**日志系统**替换为自定义 `logging.Logger`：调用 `configure_agno_logging(custom_default_logger=custom_logger)` 后，Agno 内部所有通过 `agno.utils.log` 的日志输出将使用该自定义 logger。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | 默认（无显式配置） | Agent 默认模型 |
| `custom_logger` | `logging.getLogger("custom_logger")` | Python 标准 logger |

## 核心代码模式

```python
import logging
from agno.utils.log import configure_agno_logging, log_info

def get_custom_logger():
    custom_logger = logging.getLogger("custom_logger")
    handler = logging.StreamHandler()
    formatter = logging.Formatter("[CUSTOM_LOGGER] %(levelname)s: %(message)s")
    handler.setFormatter(formatter)
    custom_logger.addHandler(handler)
    custom_logger.setLevel(logging.INFO)
    custom_logger.propagate = False  # 不传播给父 logger
    return custom_logger

custom_logger = get_custom_logger()

# 全局配置：Agno 所有 log_xxx() 调用改走此 logger
configure_agno_logging(custom_default_logger=custom_logger)

# 验证：这条日志使用自定义格式
log_info("This is using our custom logger!")

agent = Agent()
agent.print_response("What can I do to improve my sleep?")
# Agent 内部日志也走 [CUSTOM_LOGGER] 格式
```

## configure_agno_logging 效果

| 功能 | 使用自定义 logger 前 | 使用后 |
|------|---------------------|--------|
| Agno 内部 `log_info()` | 默认 Agno logger 格式 | `[CUSTOM_LOGGER] INFO: ...` |
| Agent run 日志 | 默认格式 | 自定义格式 |
| 日志目的地 | stderr（默认） | handler 定义（此处为 StreamHandler） |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/utils/log.py` | `configure_agno_logging()` | 全局 logger 替换入口 |
| `agno/utils/log.py` | `log_info/log_debug/log_error` | 内部日志函数 |
