---
name: smart-search
description: Intelligent routing service that automatically selects the optimal search engine (Exa for code/docs, Metaso for academic, Brave for news, Bocha for Chinese, GitHub for repos, Zai for media, Fetch for URLs) based on query type. Use when users need to: (1) Search or query the web, (2) Fetch specific URL content, (3) Search GitHub repositories/issues/PRs, (4) Analyze images/videos/screenshots, (5) Get daily AI news digest, (6) Find academic papers or research.
license: Apache-2.0
---

# Smart Search - Intelligent Search Routing

Automatically route queries to the most appropriate search service using keyword-based priority rules.

---

# Auto Integration (自动集成)

## search-aggregator MCP Server 自动调用

**IMPORTANT**: 当 `search-aggregator` MCP Server 可用时，Deep Research 模式必须自动使用它。

### 检测 search-aggregator 可用性

首先检查是否有 `search-aggregator` MCP Server 可用：
- 查找工具: `hybrid_research`, `aggregate_search`, `quick_search`
- 如果可用 → 使用自动集成流程
- 如果不可用 → 回退到手动流程

### Deep Research 自动流程 (使用 search-aggregator)

```yaml
Step 1: 模式检测
  调用: mcp__search_aggregator__hybrid_research(query)
  返回: { mode, engines, instructions }

Step 2: 并发调用搜索引擎
  根据 engines 列表，并发调用:
  - mcp__exa__get_code_context_exa
  - mcp__brave-search__brave_web_search
  - mcp__bocha__search
  - mcp__metaso__metaso_web_search
  - mcp__github__search_repositories

Step 3: 聚合去重
  调用: mcp__search_aggregator__aggregate_search(
    query,
    engine_results: { Exa: [...], Brave: [...], ... },
    max_per_domain: 2
  )
  返回: { results, stats: { totalOriginal, totalAfterDedup, dedupRate } }

Step 4: 格式化输出
  使用下方定义的输出格式
```

### Quick Search 流程

```yaml
使用 P0-P8 规则选择引擎 → 直接调用 → 返回结果
(可选) 调用 mcp__search_aggregator__quick_search() 缓存结果
```

### 回退流程 (search-aggregator 不可用时)

如果 search-aggregator 不可用，则：
1. 使用下方 Hybrid Recall Mode 手动检测
2. 手动并发调用搜索引擎
3. 手动执行 URL 去重 (参考下方 URL Deduplication 规则)
4. 手动格式化输出

---

# Hybrid Recall Mode (混合召回模式)

## Mode Selection

首先判断查询类型，选择 **Quick Search** 或 **Deep Research**：

```
查询输入
    │
    ├─ 特殊类型检测
    │   ├─ URL (http/https) ────────────────→ Fetch
    │   ├─ 图片/视频/截图 ───────────────────→ Zai MCP
    │   └─ GitHub (repo/issue/PR) ───────────→ GitHub MCP
    │
    ├─ Quick Search 触发
    │   ├─ 怎么 / 如何 / what is / how to
    │   ├─ 定义 / 是什么 / meaning
    │   ├─ 快速 / 简单 / quick
    │   └─ 单一事实查询
    │
    ├─ Deep Research 触发
    │   ├─ 分析 / 评估 / 对比 / 比较
    │   ├─ 趋势 / 研究 / 综述
    │   ├─ 全面 / 深入 / comprehensive
    │   ├─ overview / analysis / evaluate
    │   └─ 跨领域问题
    │
    └─ 默认 → Quick Search
```

## Quick Search (快速搜索)

**目标**: 简单查询，快速响应，单路召回

| 特性 | 说明 |
|------|------|
| **触发关键词** | 怎么, 如何, 什么, what is, how to, 定义, 快速 |
| **召回策略** | 单路最优引擎 (P0-P8) |
| **结果数量** | 3-5 条 |
| **响应速度** | < 5 秒 |
| **Token预算** | 3000-5000 |
| **URL去重** | 不需要 |

**示例**:
```
"Python 怎么打印"      → Exa (P0)
"React useEffect 是什么"  → Exa (P0)
"今日 AI 新闻"        → Brave + Metaso
```

## Deep Research (深度研究)

**目标**: 复杂分析，多维度召回，并发执行

| 特性 | 说明 |
|------|------|
| **触发关键词** | 分析, 评估, 对比, 趋势, 全面, 深入, 研究, 综述 |
| **召回策略** | 多路并发召回 (Mode-A/R/T/F) |
| **结果数量** | 8-15 条 (去重后) |
| **响应速度** | 10-20 秒 |
| **Token预算** | 8000-12000 |
| **URL去重** | 执行完整去重流程 |

**并发模式选择** (详见下方 Concurrent Recall Mode):

| 场景 | 触发词 | 并发组合 |
|------|--------|----------|
| **技术分析** | 分析, 趋势, 评估 | Exa + Brave + Metaso |
| **行业调研** | 市场, 格局, 竞争 | Brave + Bocha + Metaso |
| **技术选型** | 对比, vs, 比较 | Exa + GitHub |
| **全量召回** | 全面, 深入, 综述 | Exa + Brave + Bocha + Metaso |

**示例**:
```
"分析 Rust 在 2025 的采用趋势"    → Mode-A (Exa + Brave + Metaso)
"对比 Next.js vs Remix"          → Mode-T (Exa + GitHub)
"全面分析 Agent 技术发展现状"     → Mode-F (全量召回)
```

## Output Format Comparison

### Quick Search Output
```markdown
## 搜索结果

1. **[标题]**
   摘要：[50-100字]
   来源：[链接](url)
```

### Deep Research Output
```markdown
# 分析结果：[主题]

📊 **召回统计**: 原始 XX 条 → 去重后 YY 条 (去重率 ZZ%)

## 核心发现
- [综合要点]
- [趋势结论]

### [按来源分组]
1. **[标题]** `[来源]`
   摘要：[100-150字]
   来源：[链接](url)
   相关性：★★★★★

## 综合结论
[分析结论]
```

---

## Quick Reference (单路优先级 - Quick Search 使用)

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

---

# Concurrent Recall Mode (并发召回模式)

## Mode Detection

先判断查询类型，决定使用**单路模式**还是**并发模式**：

| Query Type | Trigger Keywords | Mode |
|------------|------------------|------|
| **简单查询** | 怎么, 如何, what is, how to, 快速问 | 单路 P0-P8 |
| **分析型** | 分析, 评估, 对比, 比较, 研究, 趋势, overview, analysis, comparison, evaluate | 并发 Mode-A |
| **综合型** | 全面, 整体, 深入, 综述, comprehensive, deep dive, in-depth | 并发 Mode-F |
| **时效+深度** | 最新 + 研究, 2025 + 趋势, recent + analysis | 并发 Mode-A |
| **跨领域** | 技术 + 商业, 学术 + 应用, industry + adoption | 并发 Mode-F |

**判断优先级**: 先检测并发模式关键词，若无则回退到 P0-P8 单路模式。

## Concurrent Modes

### Mode-A: Technical Analysis (技术分析)
```
Concurrent: Exa + Brave + Metaso
├── Exa (tokensNum=3000): 代码/API/文档
├── Brave (count=5): 最新资讯/动态
└── Metaso (size=3): 学术研究/论文

Example: "分析 Rust 在 2025 的采用趋势"
Output: 8-12 条去重结果
```

### Mode-R: Industry Research (行业调研)
```
Concurrent: Brave + Bocha + Metaso
├── Brave (count=5): 英文资讯
├── Bocha (count=5): 中文资讯
└── Metaso (size=3): 深度分析

Example: "分析国内大模型市场竞争格局"
Output: 8-12 条去重结果
```

### Mode-T: Technology Selection (技术选型)
```
Concurrent: Exa + GitHub
├── Exa (tokensNum=4000): 文档/教程
└── GitHub (perPage=10): 实战项目/Stars/Issues

Example: "对比 Next.js vs Remix 的优缺点"
Output: 8-12 条去重结果
```

### Mode-F: Full Recall (全量召回)
```
Concurrent: Exa + Brave + Bocha + Metaso
├── Exa (tokensNum=3000): 技术文档
├── Brave (count=5): 最新资讯
├── Bocha (count=5): 中文内容
└── Metaso (size=3): 学术研究

Example: "全面分析 Agent 技术的发展现状"
Output: 10-15 条去重结果
```

## URL Deduplication (URL 去重)

### 执行步骤

```yaml
步骤1: 并发调用
  - 在单次响应中并发调用所有相关 MCP 工具
  - 等待所有结果返回

步骤2: 收集与规范化
  - 收集所有结果到统一列表
  - 规范化 URL:
    * 移除追踪参数: ?ref=*, &utm_*, &source=*
    * 统一协议: http → https
    * 移除尾部斜杠: /path/ → /path

步骤3: 域名分组
  - 提取主域名: example.com
  - 按域名分组: domain_results[domain] = [items]

步骤4: 去重处理
  对每个域名分组执行:
    a. 完全相同 URL → 只保留一条
    b. 路径相似度 > 80% → 保留更完整版本
    c. 同一域名最多保留 2 条（按相关性排序）

步骤5: 输出统计
  - 标注: "原始 XX 条 → 去重后 YY 条"
  - 每条标注来源: [Exa] [Brave] [Bocha] [Metaso]
```

### 去重规则表

| 场景 | 判断 | 处理 |
|------|------|------|
| `example.com/post/123` | `example.com/post/123?ref=tw` | 保留无参数版 |
| `example.com/a/b/c` | `example.com/a/b/c/` | 保留规范化版 |
| `blog.example.com/x` | `example.com/y` | 不同子域名，都保留 |
| `example.com/post-abc` | `example.com/post-abc-def` | 相似度>80%，取一 |
| 同一域名 > 2 条 | 按 date/relevance 排序 | 只保留 Top 2 |

### URL 规范化示例

```
原始 URL                          →  规范化后
─────────────────────────────────────────────────────────
https://example.com/post?ref=twitter  →  https://example.com/post
http://example.com/path/               →  https://example.com/path
https://example.com/x&utm_source=gg    →  https://example.com/x
```

### 路径相似度判断

```yaml
相似度计算:
  - 提取路径最后一部分 (slug)
  - 计算编辑距离 / 长度
  - > 80% 视为同一文章

示例:
  /posts/rust-2025  vs  /posts/rust-2025-trends  → 重复
  /docs/api/v1      vs  /docs/api/v2             → 不重复
```

## Result Merging & Output

### 排序优先级

```yaml
1. 时效性: 发布日期 < 48小时  (权重 +30%)
2. 相关性: 标题/摘要与查询匹配度 (权重 +50%)
3. 权威性: 官方文档/顶会/知名博客 (权重 +20%)
```

### 输出格式

```markdown
# 分析结果：[查询主题]

📊 **召回统计**: 原始 XX 条 → 去重后 YY 条 (去重率 ZZ%)

## 核心发现
- [基于多方信息综合的要点]
- [数据/趋势结论]

---

### 技术视角 [Exa]
1. **[标题]** `[Exa]`
   摘要：[100-150字]
   来源：[链接](url)
   相关性：★★★★★

### 市场动态 [Brave]
1. **[标题]** `[Brave]` ⚡️ 新 (2小时内)
   摘要：...
   来源：[链接](url)
   发布时间：2025-XX-XX

### 中文视角 [Bocha]
1. **[标题]** `[Bocha]`
   摘要：...
   来源：[链接](url)

### 学术观点 [Metaso]
1. **[论文标题]** `[Metaso]`
   摘要：...
   来源：[链接](url)

---

## 综合结论
[基于多方信息的分析结论]
```

## Token Budget Control

| Engine | 单路预算 | 并发预算 | 说明 |
|--------|----------|----------|------|
| Exa | 5000 | 3000 | 减半避免超限 |
| Brave | 10 | 5 | 控制结果数 |
| Bocha | 18 | 5 | 控制结果数 |
| Metaso | 10 | 3 | 精选高质量 |
| GitHub | 10 | 10 | 保持原量 |
| Fetch | 5000 | 3000 | 按需使用 |

**并发后总预算**: 约 8000-10000 tokens (去重后实际更少)
