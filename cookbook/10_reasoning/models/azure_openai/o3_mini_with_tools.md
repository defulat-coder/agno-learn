# o3_mini_with_tools.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/azure_openai/o3_mini_with_tools.py`

## 概述

本示例展示通过 **Azure OpenAI 部署的 o3-mini + `YFinanceTools`** 进行金融分析。与直接使用 OpenAI API 的版本功能相同，但通过 Azure 企业级部署访问 o3-mini，适合企业合规要求。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureOpenAI(id="o3-mini")` | 通过 Azure 部署的 o3-mini |
| `tools` | `[YFinanceTools()]` | 金融数据工具 |
| `instructions` | `"Use tables to display data."` | 表格格式化 |
| `markdown` | `True` | Markdown 格式化 |

## 核心组件解析

### AzureOpenAI vs OpenAIChat

| 特性 | `OpenAIChat` | `AzureOpenAI` |
|------|-------------|---------------|
| API 端点 | api.openai.com | your-resource.openai.azure.com |
| 认证 | OPENAI_API_KEY | AZURE_OPENAI_API_KEY + endpoint |
| 适用场景 | 开发/个人 | 企业/合规 |
| 数据处理 | OpenAI 政策 | Azure 政策（无训练） |

### Azure OpenAI 环境配置

需要以下环境变量：
- `AZURE_OPENAI_API_KEY`
- `AZURE_OPENAI_ENDPOINT`
- `AZURE_OPENAI_API_VERSION`

部署名称（id）对应 Azure 资源中的 deployment name。

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.1 | `instructions` | `"Use tables to display data."` | 是 |
| 3.2.1 | `markdown` | `True` | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/openai_chat.py` | `AzureOpenAI` | Azure OpenAI Chat 模型 |
| `agno/tools/yfinance.py` | `YFinanceTools` | Yahoo Finance 工具 |
