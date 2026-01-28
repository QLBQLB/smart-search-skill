---
name: smart-search
description: Intelligent routing service that automatically selects the optimal MCP search engine (Exa, Metaso, Brave, Bocha, GitHub, Zai MCP, Fetch) based on query type. Use this when users need to search, query, or fetch information from the web.
---

# Smart Search - Intelligent MCP Routing

Automatically route search queries to the most appropriate MCP service based on content analysis and priority rules.

---

# Core Principles

- **Smart Routing**: Auto-select optimal MCP without manual specification
- **Priority-Driven**: Follow 9-level priority rules (P0-P8)
- **Cost Optimization**: Prefer free MCPs (Metaso > Bocha > Brave > Exa)
- **Graceful Degradation**: Auto-fallback to alternatives when primary MCP fails
- **Token Control**: Dynamically adjust response size based on query complexity
- **Source Attribution**: News results MUST include source links (出处链接)

---

# Decision Flow

```
User Input
    │
    ├─ Contains http:// or https:// ─────────────────────→ Fetch (P6)
    │
    ├─ Image/Video/Screenshot/OCR ───────────────────────→ Zai MCP (P5)
    │
    ├─ GitHub/Repository/Issue/PR/Commit ────────────────→ GitHub MCP (P4)
    │
    ├─ Academic/Paper/Research/Study/Journal ────────────→ Metaso (P1)
    │
    ├─ Code/Programming/API/Function/Framework/Library ───→ Exa (P0)
    │
    ├─ Today/This Week/Latest/Recent/News ────────────────→ Brave (P2)
    │
    ├─ Chinese content / 国内 / 中文 ─────────────────────→ Bocha (P3)
    │
    ├─ English technical content ─────────────────────────→ Exa (P8)
    │
    └─ Default ───────────────────────────────────────────→ Brave (P7)
```

---

# MCP Services Configuration

| Priority | MCP | Use Case | Keywords | Token Limit |
|----------|-----|----------|----------|-------------|
| P0 | **Exa** | Code, API docs | code, programming, API, function, framework, library, tutorial, example, syntax, bug | 5000 (2000-10000) |
| P1 | **Metaso** | Academic search | academic, paper, research, study, journal, literature, citation, thesis | 10 (5-20) |
| P2 | **Brave** | News & time-sensitive | today, this week, latest, recent, news, time-filtered | 10 (5-20) |
| P3 | **Bocha** | Chinese content | 国内, 中文, 搜索, 百科, 中文内容 | 18 (8-30) |
| P4 | **GitHub** | Repo operations | github, repository, issue, PR, commit, branch | 10 (5-50) |
| P5 | **Zai MCP** | Media analysis | image, video, screenshot, OCR, analyze picture | N/A |
| P6 | **Fetch** | Direct URL | http://, https:// | 5000 (3000-20000) |
| P7 | **Brave** | Default fallback | (other cases) | 10 |
| P8 | **Exa** | English content | english technical docs | 3000 |

---

# Token Control Strategy

## Complexity Levels

| Level | Exa tokensNum | Metaso size | Brave count | Bocha count | GitHub perPage | Use When |
|-------|---------------|-------------|-------------|-------------|----------------|----------|
| **Quick** | 2000-3000 | 5 | 5 | 8 | 5 | Simple queries, basic definitions |
| **Regular** (default) | 5000 | 10 | 10 | 18 | 10 | Common usage, examples, troubleshooting |
| **Deep** | 8000+ | 20 | 20 | 30 | 50 | Multi-condition, architecture, comparisons |

## Quick Examples

- `"Python 教程"` → Quick (2000-3000 tokens)
- `"Python FastAPI 异步编程"` → Regular (5000 tokens)
- `"FastAPI JWT + Database + Async complete project"` → Deep (8000+ tokens)

---

# Usage Examples

## ✅ Exa (P0/P8) - Code & Programming

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

## ✅ Brave (P2/P7) - News & General Search

```
"今天的科技新闻"
"搜索 2025 年最新 AI 技术"
"本周热门话题"
"最新手机推荐"
```

## ✅ Bocha (P3) - Chinese Content

```
"国内 AI 大模型对比"
"搜索中文资料"
"百度百科"
"国内科技公司"
```

## ✅ GitHub (P4) - Repository Operations

```
"查看我的仓库"
"创建一个新 Issue"
"列出最近的 Commits"
"分析这个 PR"
"搜索 Python 开源项目"
```

## ✅ Zai MCP (P5) - Media Analysis

```
"分析这张图片"
"提取视频中的文字"
"识别截图中的代码"
"描述这张图表"
"读取截图错误信息"
```

## ✅ Fetch (P6) - Direct URL

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
- **Always include source links when displaying news results**

## DON'T

- Use Exa for `"今天的新闻"` → Brave (P2)
- Use Metaso for `"代码示例"` → Exa (P0)
- Use GitHub for web search → Search MCPs
- Combine MCPs for redundant results
- Omit source attribution for news content

---

# Fallback Strategy

| Primary | Fallback 1 | Fallback 2 |
|---------|-----------|------------|
| Exa | Brave | Metaso |
| Metaso | Bocha | Brave |
| Brave | Bocha | Exa |
| Bocha | Brave | Exa |
| GitHub | Fetch | - |
| Zai MCP | - | - |
| Fetch | Brave (search related) | - |

---

# Combination Scenarios

| Scenario | MCP Combo | Rationale |
|----------|-----------|-----------|
| Technical research | Exa + GitHub | Code examples + repository implementation |
| Comprehensive understanding | Brave + Bocha + Exa | News + Chinese + English docs |
| Deep research | Metaso + Fetch | Academic summary + detailed content |

**Combination Rules**:
- Only for complementary scenarios
- Max 3 concurrent MCP requests
- Avoid duplicate content from different MCPs
