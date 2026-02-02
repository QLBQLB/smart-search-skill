---
name: smart-search
description: Intelligent routing service that automatically selects the optimal search engine (Exa for code/docs, Metaso for academic, Brave for news, Bocha for Chinese, GitHub for repos, Zai for media, Fetch for URLs) based on query type. Use when users need to: (1) Search or query the web, (2) Fetch specific URL content, (3) Search GitHub repositories/issues/PRs, (4) Analyze images/videos/screenshots, (5) Get daily AI news digest, (6) Find academic papers or research.
license: Apache-2.0
---

# Smart Search - Intelligent Search Routing

Automatically route queries to the most appropriate search service using keyword-based priority rules.

---

## Quick Reference

| Priority | Service | Trigger Keywords | Token/Count |
|----------|---------|-----------------|-------------|
| **P0** | Exa | code, API, function, framework, library, tutorial, syntax | 5000 |
| **P1** | Metaso | paper, research, study, journal, thesis, academic | 10 |
| **P2** | Brave | today, this week, latest, recent, news | 10 |
| **P3** | Bocha | 中文, 国内, 百科, Chinese content | 18 |
| **P4** | GitHub | github, repo, issue, PR, commit, branch | 10 |
| **P5** | Zai MCP | image, video, screenshot, OCR, analyze picture | N/A |
| **P6** | Fetch | http://, https:// (direct URL) | 5000 |
| **P7** | Brave | (default fallback) | 10 |
| **P8** | Exa | English technical docs | 3000 |

---

## Decision Flow

```
User Input
    │
    ├─ Contains http:// or https:// ──────────────→ Fetch (P6)
    │
    ├─ Image/Video/Screenshot/OCR ─────────────────→ Zai MCP (P5)
    │
    ├─ GitHub/Repo/Issue/PR/Commit ────────────────→ GitHub MCP (P4)
    │
    ├─ Academic/Paper/Research/Thesis ──────────────→ Metaso (P1)
    │
    ├─ Code/API/Framework/Library ──────────────────→ Exa (P0)
    │
    ├─ Today/Latest/News + "AI" ───────────────────→ Brave + Metaso + Top 8 digest
    │
    ├─ 中文/国内 ───────────────────────────────────→ Bocha (P3)
    │
    └─ Default ─────────────────────────────────────→ Brave (P7)
```

---

## Token Control

Adjust search depth based on query complexity:

| Level | Tokens/Count | Use Case |
|-------|--------------|----------|
| **Quick** | 5-10 | Simple queries, definitions |
| **Regular** | 10-20 | Common usage, examples (default) |
| **Deep** | 30-50 | Multi-condition, architecture |

---

# News Output Format (AI新闻输出格式)

When user searches for daily AI news (今日/当天 AI新闻/资讯), use **Brave + Metaso** combined and output:

```
# 今日AI资讯 Top 8

### 1. [新闻标题]

**摘要：** [新闻内容摘要，50-100字]

**来源：** [来源名](链接)

**标签：** #标签1 #标签2 #标签3

**相关性：** ★★★★★

---

### 2. [新闻标题]

**摘要：** ...
```

## Timeliness Judgment (时效性判断)

Before including news, verify timeliness:

| Indicator | Fresh | Stale |
|-----------|-------|-------|
| Publish date | Within 2 days | Older than 7 days |
| Date string in URL | Contains current date (2026-02-*) | No date or old date |
| Source freshness | News sites, blogs | Static docs |
| Content keywords | "今日", "最新", "just announced" | "历史", "回顾", "总览" |

**Priority**: Prefer news with publish dates within 2 days. Stale news (>7 days) should be excluded.

**Source attribution**: Always include clickable source links.

---

# Usage Guidelines

## DO

- Analyze keywords before routing
- Implement fallback on failure (30s timeout)
- **For AI news: Use Brave + Metaso combined**
- Format each news item: 标题 + 摘要 + 来源 + 标签 + 相关性
- **Filter to recent 2 days**: exclude news older than 48 hours
- Always include clickable source links

## DON'T

- Use Exa for news → Brave
- Use Metaso for code → Exa
- Include news older than 2 days in "今日" digest
- Omit source attribution for news
- Combine MCPs redundantly

---

# Fallback Strategy

| Primary | Fallback 1 | Fallback 2 |
|---------|-----------|------------|
| Exa | Brave | Metaso |
| Metaso | Bocha | Brave |
| Brave | Bocha | Exa |
| Bocha | Brave | Exa |
| GitHub | Fetch | - |
| Fetch | Brave | - |

---

# Combination Rules

Only for complementary scenarios (max 3 concurrent):

| Scenario | Combo | Rationale |
|----------|-------|-----------|
| **AI News Digest** | Brave + Metaso | English + Chinese comprehensive coverage |
| Technical research | Exa + GitHub | Code + repos |
| Deep research | Metaso + Fetch | Academic + content |
| Comprehensive | Brave + Bocha | News + Chinese |
