---
name: smart-search
description: Intelligent routing service that automatically selects the optimal MCP search engine (Exa, Metaso, Bocha, GitHub, Zai MCP, Fetch) based on query type. Use this when users need to search, query, or fetch information from the web.
---

# Smart Search - Intelligent MCP Routing

Automatically route search queries to the most appropriate MCP service based on content analysis and priority rules.

---

# Core Principles

- **Smart Routing**: Auto-select optimal MCP without manual specification
- **Priority-Driven**: Follow 7-level priority rules (P0-P7)
- **Cost Optimization**: Prefer free MCPs (Metaso > Bocha > Exa)
- **Graceful Degradation**: Auto-fallback to alternatives when primary MCP fails
- **Token Control**: Dynamically adjust response size based on query complexity

---

# Decision Flow

```
User Input
    │
    ├─ Contains http:// or https:// ─────────────────────→ Fetch (P5)
    │
    ├─ Image/Video/Screenshot/OCR ───────────────────────→ Zai MCP (P4)
    │
    ├─ GitHub/Repository/Issue/PR/Commit ────────────────→ GitHub MCP (P3)
    │
    ├─ Academic/Paper/Research/Study/Journal ────────────→ Metaso (P1)
    │
    ├─ Code/Programming/API/Function/Framework/Library ───→ Exa (P0)
    │
    ├─ Today/This Week/Latest/Recent/News ────────────────→ Bocha (P2)
    │
    ├─ English technical content ─────────────────────────→ Exa (P7)
    │
    └─ Default ───────────────────────────────────────────→ Bocha (P6)
```

---

# MCP Services Configuration

| Priority | MCP | Use Case | Keywords | Token Limit |
|----------|-----|----------|----------|-------------|
| P0 | **Exa** | Code, API docs | code, programming, API, function, framework, library, tutorial, example, syntax, bug | 5000 (2000-10000) |
| P1 | **Metaso** | Academic search | academic, paper, research, study, journal, literature, citation, thesis | 10 (5-20) |
| P2 | **Bocha** | Chinese news | today, this week, latest, recent, news, time-filtered | 18 (8-30) |
| P3 | **GitHub** | Repo operations | github, repository, issue, PR, commit, branch | 10 (5-50) |
| P4 | **Zai MCP** | Media analysis | image, video, screenshot, OCR, analyze picture | N/A |
| P5 | **Fetch** | Direct URL | http://, https:// | 5000 (3000-20000) |
| P6 | **Bocha** | Default fallback | (other cases) | 18 |
| P7 | **Exa** | English content | english technical docs | 3000 |

---

# Token Control Strategy

## Complexity Levels

| Level | Exa tokensNum | Metaso size | Bocha count | GitHub perPage | Use When |
|-------|---------------|-------------|-------------|----------------|----------|
| **Quick** | 2000-3000 | 5 | 8 | 5 | Simple queries, basic definitions |
| **Regular** (default) | 5000 | 10 | 18 | 10 | Common usage, examples, troubleshooting |
| **Deep** | 8000+ | 20 | 30 | 50 | Multi-condition, architecture, comparisons |

## Quick Examples

- `"Python 教程"` → Quick (2000-3000 tokens)
- `"Python FastAPI 异步编程"` → Regular (5000 tokens)
- `"FastAPI JWT + Database + Async complete project"` → Deep (8000+ tokens)

---

# Usage Examples

## ✅ Exa (P0) - Code & Programming

```
"搜索 Python FastAPI 代码示例"
"React hooks tutorial"
"How to use async/await in JavaScript"
"Spring Boot configuration example"
"解决 CSS 居中问题"
```

## ✅ Metaso (P1) - Academic & Research

```
"查找 AI 相关的学术论文"
"大模型 Transformer 研究"
"深度学习最新研究进展"
"机器学习期刊论文"
"知识图谱综述"
```

## ✅ Bocha (P2/P6) - Chinese News & Info

```
"今天的科技新闻"
"国内 AI 大模型对比"
"搜索 2025 年最新 AI 技术"
"本周热门话题"
"最新手机推荐"
```

## ✅ GitHub (P3) - Repository Operations

```
"查看我的仓库"
"创建一个新 Issue"
"列出最近的 Commits"
"分析这个 PR"
"搜索 Python 开源项目"
```

## ✅ Zai MCP (P4) - Media Analysis

```
"分析这张图片"
"提取视频中的文字"
"识别截图中的代码"
"描述这张图表"
"读取截图错误信息"
```

## ✅ Fetch (P5) - Direct URL

```
"获取 https://example.com 的内容"
"读取 https://api.github.com/users/claude"
"抓取 https://docs.python.org/3/"
```

---

# Usage Guidelines

## DO

- Use single MCP when query matches one category clearly
- Analyze query keywords and context before routing
- Implement fallback on timeout (30s) or failure
- Prefer free MCPs when multiple options exist

## DON'T

- Use Exa for `"今天的新闻"` → Bocha (P2)
- Use Metaso for `"代码示例"` → Exa (P0)
- Use GitHub for web search → Search MCPs
- Combine MCPs for redundant results

---

# Fallback Strategy

| Primary | Fallback 1 | Fallback 2 |
|---------|-----------|------------|
| Exa | Bocha | Metaso |
| Metaso | Bocha | Exa |
| Bocha | Exa | - |
| GitHub | Fetch | - |
| Zai MCP | - | - |
| Fetch | Exa (search related) | - |

---

# Combination Scenarios

| Scenario | MCP Combo | Rationale |
|----------|-----------|-----------|
| Technical research | Exa + GitHub | Code examples + repository implementation |
| Comprehensive understanding | Bocha + Exa | Chinese news + English docs |
| Deep research | Metaso + Fetch | Academic summary + detailed content |

**Combination Rules**:
- Only for complementary scenarios
- Max 3 concurrent MCP requests
- Avoid duplicate content from different MCPs
