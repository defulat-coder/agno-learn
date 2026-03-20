# load_data.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Load F1 Data - Downloads F1 data (1950-2020) and loads into PostgreSQL.

Usage: python -m agents.dash.scripts.load_data
"""

from io import StringIO

import httpx
import pandas as pd
from db import db_url
from sqlalchemy import create_engine

S3_URI = "https://agno-public.s3.amazonaws.com/f1"

TABLES = {
    "constructors_championship": f"{S3_URI}/constructors_championship_1958_2020.csv",
    "drivers_championship": f"{S3_URI}/drivers_championship_1950_2020.csv",
    "fastest_laps": f"{S3_URI}/fastest_laps_1950_to_2020.csv",
    "race_results": f"{S3_URI}/race_results_1950_to_2020.csv",
    "race_wins": f"{S3_URI}/race_wins_1950_to_2020.csv",
}

if __name__ == "__main__":
    engine = create_engine(db_url)
    total = 0

    for table, url in TABLES.items():
        print(f"Loading {table}...", end=" ", flush=True)
        response = httpx.get(url, timeout=30.0)
        df = pd.read_csv(StringIO(response.text))
        df.to_sql(table, engine, if_exists="replace", index=False)
        print(f"{len(df):,} rows")
        total += len(df)

    print(f"\nDone! {total:,} total rows")
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/01_demo/agents/dash/scripts/load_data.py`

## 概述

本脚本从 **S3 公开 CSV** 下载 F1 数据，用 **pandas + SQLAlchemy** 写入 **`db_url`** 指向的 PostgreSQL（`if_exists="replace"`）。**独立运维脚本**，不参与 Agent `run` 或 system 拼装。

**核心配置一览：** 无 Agent；使用环境变量 **`DATABASE_URL`**（经 `db.db_url`）。

## 架构分层

```
httpx GET CSV → pandas.read_csv → DataFrame.to_sql → PostgreSQL 表
```

## 核心组件解析

`TABLES` 字典映射表名到 URL；`__main__` 循环加载并打印行数（`load_data.py` L24-L36）。

### 运行机制与因果链

1. **路径**：网络 → 本地 DataFrame → DB。
2. **副作用**：**覆盖**已有表；行数累计打印。
3. **分支**：网络失败则异常退出（未在脚本内捕获）。

## System Prompt 组装

不适用。

## 完整 API 请求

仅 **HTTP GET** CSV，无 LLM。

## Mermaid 流程图

```mermaid
flowchart TD
    A["S3 CSV"] --> B["【关键】pandas.to_sql"]
    B --> C["PostgreSQL"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `cookbook/01_demo/db.py` | `db_url` | 连接串 |
| `load_data.py` | `__main__` L24+ | 批量灌表 |
