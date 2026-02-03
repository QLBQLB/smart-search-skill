# smart-search-skill

[![English](https://img.shields.io/badge/lang-English-blue.svg)](README.md) [![Chinese](https://img.shields.io/badge/lang-中文-red.svg)](README.zh-CN.md)

> Intelligent routing service - automatically select the optimal MCP search engine. Works best with [search-aggregator-skill](https://github.com/QLBQLB/search-aggregator-skill).

---

## Overview

A Claude Code skill that routes search queries to the most appropriate MCP service without manual specification. Supports 7 major MCP engines with 9-level intelligent priority-based routing.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│              smart-search (Routing Layer)                  │
│      Smart search engine selection + query mode detection    │
│   https://github.com/QLBQLB/smart-search-skill          │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│           search-aggregator (Aggregation Layer)             │
│     Concurrent calls + auto deduplication + result cache    │
│      https://github.com/QLBQLB/search-aggregator-skill  │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│   Exa    │  Brave   │  Metaso  │  Bocha   │  GitHub  │
│ (Code/API)│ (News)   │ (Academic)│ (Chinese)│ (Projects)│
└──────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## Supported MCP Services

| Priority | MCP | Use Case | Keywords |
|----------|-----|----------|----------|
| P0 | **Exa** | Code, API docs | code, API, function, framework, library, tutorial, syntax |
| P1 | **Metaso** | Academic search | paper, research, study, journal, thesis, academic |
| P2 | **Brave** | English news | today, this week, latest, recent, news |
| P3 | **Bocha** | Chinese content | 中文, 国内, 百科, Chinese content |
| P4 | **GitHub** | Repo operations | github, repo, issue, PR, commit, branch |
| P5 | **Zai MCP** | Media analysis | image, video, screenshot, OCR, analyze picture |
| P6 | **Fetch** | Direct URL | http://, https:// |
| P7 | **Brave** | Default fallback | (default fallback) |
| P8 | **Exa** | English technical docs | English technical docs |

---

## Complete Installation

```bash
# 1. Install smart-search skill (routing layer)
git clone https://github.com/QLBQLB/smart-search-skill.git ~/.claude/skills/smart-search

# 2. Install search-aggregator skill + MCP Server (aggregation layer)
git clone https://github.com/QLBQLB/search-aggregator-skill.git ~/.claude/skills/search-aggregator
cd ~/.claude/skills/search-aggregator
npm install
npm run build

# 3. Configure MCP Server
claude mcp add search-aggregator node C:\Users\YourName\.claude\skills\search-aggregator\dist\index.js
```

---

## Search Modes

### Quick Search

Simple queries, fast response, single-path recall

| Feature | Description |
|---------|-------------|
| **Trigger Keywords** | how to, what is, define, quick |
| **Strategy** | Single best engine (P0-P8) |
| **Results** | 3-5 items |
| **Response Time** | < 5 seconds |
| **Token Budget** | 3000-5000 |

### Deep Research

Complex analysis, multi-path recall, concurrent execution

| Feature | Description |
|---------|-------------|
| **Trigger Keywords** | analysis, evaluate, compare, trend, comprehensive, research |
| **Strategy** | Multi-path concurrent (Mode-A/R/T/F) |
| **Results** | 8-15 items (after deduplication) |
| **Response Time** | 10-20 seconds |
| **Token Budget** | 8000-12000 |
| **URL Deduplication** | Full deduplication process |

### Concurrent Recall Modes

| Mode | Combination | Use Case | Example |
|------|------------|----------|--------|
| **Mode-A** | Exa + Brave + Metaso | Technical Analysis | "Analyze Rust adoption trends in 2025" |
| **Mode-R** | Brave + Bocha + Metaso | Industry Research | "Analyze domestic LLM market competition" |
| **Mode-T** | Exa + GitHub | Technology Selection | "Compare Next.js vs Remix pros and cons" |
| **Mode-F** | Exa + Brave + Bocha + Metaso | Full Recall | "Comprehensive analysis of Agent technology" |

---

## Installation

### Method 1: Claude Code Marketplace

```bash
/plugin marketplace add QLBQLB/smart-search-skill
```

Then install:
```bash
/plugin install smart-search@smart-search-skill
```

### Method 2: Manual Clone

```bash
# macOS / Linux
git clone https://github.com/QLBQLB/smart-search-skill.git ~/.claude/skills/smart-search

# Windows (PowerShell)
git clone https://github.com/QLBQLB/smart-search-skill.git $env:USERPROFILE\.claude\skills\smart-search
```

---

## MCP Configuration

Each MCP service needs to be configured separately:

### Exa (Code Search)
```bash
claude mcp add exa npx -y exa-mcp-server
```

### Brave (English News)
```bash
# Requires Brave Search API Key: https://api.search.brave.com/app/keys
claude mcp add brave-search -s user --env BRAVE_API_KEY=YourAPIKey npx -y @modelcontextprotocol/server-brave-search
```

### Metaso (Academic Search)
```bash
# Clone and run (requires Docker or Node.js)
git clone https://github.com/metaso-ai/search-server-metaso.git
cd search-server-metaso
docker-compose up -d  # or npm start
claude mcp add metaso http://localhost:3000/sse
```

### Bocha (Chinese Content)
```bash
# Configure your Bocha search server
claude mcp add bocha http://localhost:3001/sse
```

### GitHub (Built-in)
Requires GitHub Copilot subscription. Automatically available in Claude Code.

### Zai MCP (Media Analysis)
```bash
claude mcp add zai npx -y @z_ai/mcp-server
```

### Fetch (URL Content)
```bash
claude mcp add fetch uvx mcp-server-fetch
```

### search-aggregator (Recommended)

For automatic aggregation and deduplication of multiple search engine results:

```bash
# Clone search-aggregator skill
git clone https://github.com/QLBQLB/search-aggregator-skill.git ~/.claude/skills/search-aggregator

# Configure MCP Server
cd ~/.claude/skills/search-aggregator
npm install
npm run build
claude mcp add search-aggregator node C:\Users\YourName\.claude\skills\search-aggregator\dist\index.js
```

**search-aggregator provides:**

| Tool | Purpose |
|------|---------|
| `hybrid_research` | Auto-detect query type, return engine list |
| `aggregate_search` | Aggregate multi-engine results with deduplication |
| `quick_search` | Fast search with caching |
| `cache_info` | View and manage cache |

---

## Integration with search-aggregator

For detailed documentation, see [search-aggregator-skill](https://github.com/QLBQLB/search-aggregator-skill).

### Workflow

```yaml
User input query
    ↓
smart-search analyzes query type
    ↓
search-aggregator.hybrid_research(query)
    → Returns: { mode: "Mode-A", engines: ["Exa", "Brave", "Metaso"] }
    ↓
Concurrent calls to search engine MCPs
    ├─ mcp__exa__get_code_context_exa
    ├─ mcp__brave-search__brave_web_search
    └─ mcp__metaso__metaso_web_search
    ↓
search-aggregator.aggregate_search(results)
    → URL normalization
    → Domain grouping
    → Similarity deduplication
    → Return deduplicated results
    ↓
Format and output to user
```

---

## Usage

### Direct Invocation
```
/smart-search Python FastAPI tutorial
```

### Auto-Trigger (Recommended)

Configure `~/.claude/settings.json`:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "User message: $ARGUMENTS\n\nIf the user's message contains search, query, or fetch intent, route to appropriate MCP service:\n\n| Priority | MCP | Keywords |\n|----------|-----|----------|\n| P0 | mcp__exa__get_code_context_exa | code, programming, API, framework, tutorial, syntax |\n| P1 | mcp__metaso__metaso_web_search | academic, paper, research, study, journal, academic |\n| P2 | mcp__brave-search__brave_web_search | today, this week, latest, recent, news |\n| P3 | mcp__bocha__search | 中文, 国内, 百科, Chinese content |\n| P4 | mcp__github__* | github, repository, issue, PR, commit, branch |\n| P5 | mcp__zai-mcp-server__* | image, video, screenshot, OCR, analyze picture |\n| P6 | mcp__fetch__fetch | http://, https, URL |\n\nIf no search intent, respond normally.",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

Then simply type:
```
Search Python FastAPI async programming
Find latest deep learning papers
Today's tech news
```

---

## Features

- **Smart Routing**: Auto-select best MCP based on query analysis
- **Dual Mode Search**: Quick Search (fast) and Deep Research (deep)
- **Concurrent Recall**: Support Mode-A/R/T/F concurrent modes
- **URL Deduplication**: Auto-normalize URLs and deduplicate, max 2 per domain
- **Timeliness Filtering**: News queries only keep content within 48 hours
- **Graceful Degradation**: Auto-fallback on failure
- **Token Control**: Dynamic sizing based on query complexity and concurrency

---

## Token Control

### Single Path (Quick Search)

| Engine | Quick | Regular | Deep |
|--------|-------|---------|------|
| Exa | 3000 | 5000 | 8000+ |
| Metaso | 5 | 10 | 20 |
| Brave | 5 | 10 | 20 |
| Bocha | 8 | 18 | 30 |
| GitHub | 5 | 10 | 50 |

### Concurrent (Deep Research)

| Engine | Single Budget | Concurrent Budget | Notes |
|--------|---------------|------------------|-------|
| Exa | 5000 | 3000 | Halved to avoid overflow |
| Brave | 10 | 5 | Control result count |
| Bocha | 18 | 5 | Control result count |
| Metaso | 10 | 3 | High quality only |
| GitHub | 10 | 10 | Keep original |
| Fetch | 5000 | 3000 | On demand |

**Total concurrent budget**: ~8000-10000 tokens (less after deduplication)

---

## URL Deduplication Rules

In concurrent mode, the following deduplication process is executed:

1. **URL Normalization**: Remove tracking params (?ref=*, &utm_*), unify protocol
2. **Domain Grouping**: Group results by main domain
3. **Similarity Check**: Path similarity > 80% considered duplicate
4. **Quantity Limit**: Max 2 items per domain (sorted by relevance)

---

## Timeliness Judgment

News query timeliness filtering rules:

| Metric | Fresh | Stale |
|--------|-------|-------|
| Publish Date | Within 48 hours | Over 7 days |
| URL Date String | Contains current date | No date or old date |
| Content Keywords | "今日", "最新", "just announced" | "历史", "回顾", "总览" |

---

## Fallback Strategy

| Primary | Fallback 1 | Fallback 2 |
|--------|-----------|------------|
| Exa | Brave | Metaso |
| Metaso | Bocha | Brave |
| Brave | Bocha | Exa |
| Bocha | Brave | Exa |
| GitHub | Fetch | - |
| Fetch | Brave | - |

---

## Checklist

Before using this skill, ensure:

### Basic MCP Services
- [ ] Exa MCP installed: `claude mcp add exa npx -y exa-mcp-server`
- [ ] Brave MCP installed with API Key: `claude mcp add brave-search -s user --env BRAVE_API_KEY=YourKey npx -y @modelcontextprotocol/server-brave-search`
- [ ] Metaso MCP configured (Docker or local)
- [ ] Bocha MCP configured
- [ ] GitHub Copilot logged in
- [ ] Zai MCP installed: `claude mcp add zai npx -y @z_ai/mcp-server`
- [ ] Fetch MCP installed: `claude mcp add fetch uvx mcp-server-fetch`

### search-aggregator (Recommended)
- [ ] search-aggregator skill cloned to `~/.claude/skills/search-aggregator`
- [ ] search-aggregator MCP Server configured
- [ ] Tools verified: `hybrid_research`, `aggregate_search`, `quick_search`, `cache_info`

### Skill Configuration
- [ ] smart-search skill cloned to `~/.claude/skills/smart-search`
- [ ] (Optional) Hooks configured for auto-trigger

---

## Version History

- **v2.1.0** (2026-02-03): Add search-aggregator integration guide
- **v2.0.0** (2026-02-03): Restructure search modes, add concurrent recall, URL dedup, timeliness filtering
- **v1.3.0** (2026-01-17): Optimize structure, add bilingual README
- **v1.2.0** (2026-01-17): Add token usage control
- **v1.0.0** (2026-01-17): Initial release

---

## Related Projects

| Project | Description | Link |
|---------|-------------|------|
| **search-aggregator-skill** | Search aggregation service | [GitHub](https://github.com/QLBQLB/search-aggregator-skill) |

---

## License

MIT License