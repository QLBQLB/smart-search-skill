# smart-search-skill

[![English](https://img.shields.io/badge/lang-English-blue.svg)](README.md) [![Chinese](https://img.shields.io/badge/lang-中文-red.svg)](README.zh-CN.md)

智能路由搜索服务 - 根据查询内容自动选择最合适的 MCP 搜索引擎。

> **与 [search-aggregator-skill](https://github.com/QLBQLB/search-aggregator-skill) 搭配使用，实现智能路由 + 自动聚合的完整搜索解决方案。**

---

## 概述

这是一个 Claude Code 技能，能够根据用户查询类型自动选择最合适的 MCP 搜索服务，无需手动指定。支持 7 个主要的 MCP 引擎，采用 9 级智能优先级路由。

---

## 架构关系

```
┌─────────────────────────────────────────────────────────┐
│              smart-search (路由层)                       │
│         智能选择搜索引擎 + 查询模式检测                   │
│   https://github.com/QLBQLB/smart-search-skill          │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│           search-aggregator (聚合层)                     │
│        并发调用 + 自动去重 + 结果缓存                     │
│      https://github.com/QLBQLB/search-aggregator-skill  │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│   Exa    │  Brave   │  Metaso  │  Bocha   │  GitHub  │
│ (代码/API)│  (新闻)  │  (学术)  │  (中文)  │  (项目)  │
└──────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## 支持的 MCP 服务

| 优先级 | MCP | 用途 | 关键词 |
|----------|-----|----------|----------|
| P0 | **Exa** | 代码、API 文档 | code, API, function, framework, library, tutorial, syntax |
| P1 | **Metaso** | 学术搜索 | paper, research, study, journal, thesis, academic |
| P2 | **Brave** | 英文新闻 | today, this week, latest, recent, news |
| P3 | **Bocha** | 中文资讯 | 中文, 国内, 百科, Chinese content |
| P4 | **GitHub** | 仓库操作 | github, repo, issue, PR, commit, branch |
| P5 | **Zai MCP** | 多媒体分析 | image, video, screenshot, OCR, analyze picture |
| P6 | **Fetch** | 直接 URL | http://, https:// |
| P7 | **Brave** | 默认回退 | (default fallback) |
| P8 | **Exa** | 英文技术文档 | English technical docs |

---

## 完整安装流程

```bash
# 1. 安装 smart-search 技能 (路由层)
git clone https://github.com/QLBQLB/smart-search-skill.git ~/.claude/skills/smart-search

# 2. 安装 search-aggregator 技能 + MCP Server (聚合层)
git clone https://github.com/QLBQLB/search-aggregator-skill.git ~/.claude/skills/search-aggregator
cd ~/.claude/skills/search-aggregator
npm install
npm run build

# 3. 配置 MCP Server
claude mcp add search-aggregator node C:\Users\uiqia\.claude\skills\search-aggregator\dist\index.js
```

---

## 搜索模式

### Quick Search (快速搜索)

**目标**: 简单查询，快速响应，单路召回

| 特性 | 说明 |
|------|------|
| **触发关键词** | 怎么, 如何, 什么, what is, how to, 定义, 快速 |
| **召回策略** | 单路最优引擎 (P0-P8) |
| **结果数量** | 3-5 条 |
| **响应速度** | < 5 秒 |
| **Token预算** | 3000-5000 |

### Deep Research (深度研究)

**目标**: 复杂分析，多维度召回，并发执行

| 特性 | 说明 |
|------|------|
| **触发关键词** | 分析, 评估, 对比, 趋势, 全面, 深入, 研究, 综述 |
| **召回策略** | 多路并发召回 (Mode-A/R/T/F) |
| **结果数量** | 8-15 条 (去重后) |
| **响应速度** | 10-20 秒 |
| **Token预算** | 8000-12000 |
| **URL去重** | 执行完整去重流程 |

### 并发召回模式

| 模式 | 组合 | 适用场景 | 示例 |
|------|------|----------|------|
| **Mode-A** | Exa + Brave + Metaso | 技术分析 | "分析 Rust 在 2025 的采用趋势" |
| **Mode-R** | Brave + Bocha + Metaso | 行业调研 | "分析国内大模型市场竞争格局" |
| **Mode-T** | Exa + GitHub | 技术选型 | "对比 Next.js vs Remix 的优缺点" |
| **Mode-F** | Exa + Brave + Bocha + Metaso | 全量召回 | "全面分析 Agent 技术的发展现状" |

---

## 与 search-aggregator 搭配使用

详细文档请参阅 [search-aggregator-skill](https://github.com/QLBQLB/search-aggregator-skill)。

### 工作流程

```yaml
用户输入查询
    ↓
smart-search 分析查询类型
    ↓
search-aggregator.hybrid_research(query)
    → 返回: { mode: "Mode-A", engines: ["Exa", "Brave", "Metaso"] }
    ↓
并发调用各搜索引擎 MCP
    ├─ mcp__exa__get_code_context_exa
    ├─ mcp__brave-search__brave_web_search
    └─ mcp__metaso__metaso_web_search
    ↓
search-aggregator.aggregate_search(results)
    → URL 规范化
    → 域名分组
    → 相似度去重
    → 返回去重结果
    ↓
格式化输出给用户
```

---

## 使用方式

### 直接调用
```
/smart-search Python FastAPI 教程
```

### 自动触发（推荐）

配置 `~/.claude/settings.json` 后，直接输入搜索查询即可自动触发。

---

## 特性

- **智能路由**：根据查询分析自动选择最佳 MCP
- **双模式搜索**：Quick Search (快速) 和 Deep Research (深度)
- **并发召回**：支持 Mode-A/R/T/F 四种并发模式
- **URL 去重**：自动规范化 URL 并去重，同域名最多保留 2 条
- **时效性过滤**：新闻类查询只保留 48 小时内的内容
- **优雅降级**：失败时自动切换备选方案

---

## 相关项目

| 项目 | 说明 | 链接 |
|------|------|------|
| **search-aggregator-skill** | 搜索聚合服务 | [GitHub](https://github.com/QLBQLB/search-aggregator-skill) |

---

## 版本历史

- **v2.1.0** (2026-02-03): 添加 search-aggregator 集成说明和搭配使用指南
- **v2.0.0** (2026-02-03): 重构搜索模式，添加并发召回、URL去重、时效性过滤

---

## 许可证

MIT License