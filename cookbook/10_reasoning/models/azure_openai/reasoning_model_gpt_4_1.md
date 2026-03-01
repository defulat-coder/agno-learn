# reasoning_model_gpt_4_1.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/azure_openai/reasoning_model_gpt_4_1.py`

## 概述

本示例展示 **Azure OpenAI 上的异模型两阶段推理**：`model=AzureOpenAI(gpt-4o-mini)`（响应生成）+ `reasoning_model=AzureOpenAI(gpt-4.1)`（推理），完全通过 Azure 企业级部署实现，与 `openai/reasoning_model_gpt_4_1.py` 功能对等但使用 Azure 路由。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureOpenAI(id="gpt-4o-mini")` | 响应生成（Azure 部署） |
| `reasoning_model` | `AzureOpenAI(id="gpt-4.1")` | 推理模型（Azure 部署） |

## 核心组件解析

### 纯 Azure 两阶段推理

与 `openai/reasoning_model_gpt_4_1.py` 对比：

| 特性 | OpenAI 版 | Azure 版 |
|------|---------|---------|
| 推理模型 | `OpenAIResponses(gpt-4.1)` | `AzureOpenAI(gpt-4.1)` |
| 生成模型 | `OpenAIResponses(gpt-4o-mini)` | `AzureOpenAI(gpt-4o-mini)` |
| 数据处理 | OpenAI 政策 | Azure 政策（无训练） |
| 合规 | 基础合规 | SOC2、HIPAA、企业级 |

所有 API 调用均通过 Azure OpenAI 服务路由，数据不离开 Azure 租户。

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 所有配置 | 均未设置 | — | 否 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/openai_chat.py` | `AzureOpenAI` | Azure OpenAI Chat 模型 |
| `agno/agent/_response.py` | `handle_reasoning()` L70 | 两阶段推理协调 |
