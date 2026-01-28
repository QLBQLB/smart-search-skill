# smart-search-skill

[![English](https://img.shields.io/badge/lang-English-blue.svg)](README.md) [![Chinese](https://img.shields.io/badge/lang-中文-red.svg)](README.zh-CN.md)

Intelligent routing service - automatically select the optimal MCP search engine based on query content.

## Overview

A Claude Code skill that routes search queries to the most appropriate MCP service without manual specification. Supports 7 major MCP engines with intelligent priority-based routing.

## Supported MCP Services

| Priority | MCP | Use Case | Keywords |
|----------|-----|----------|----------|
| P0 | **Exa** | Code, API docs | code, programming, API, framework, library |
| P1 | **Metaso** | Academic search | academic, paper, research, study, journal |
| P2 | **Brave** | News & time-sensitive | today, this week, latest, recent, news |
| P3 | **Bocha** | Chinese content | 国内, 中文, 搜索, 百科 |
| P4 | **GitHub** | Repo operations | github, repository, issue, PR, commit |
| P5 | **Zai MCP** | Media analysis | image, video, screenshot, OCR |
| P6 | **Fetch** | Direct URL | http://, https:// |

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

## MCP Setup

Each MCP service needs to be configured separately:

### Exa (Code Search)
```bash
claude mcp add exa npx -y exa-mcp-server
```

### Metaso (Academic Search)
```bash
# Clone and run (requires Docker or Node.js)
git clone https://github.com/metaso-ai/search-server-metaso.git
cd search-server-metaso
docker-compose up -d  # or npm start
claude mcp add metaso http://localhost:3000/sse
```

### Brave (News Search)
```bash
# Get API Key from: https://api.search.brave.com/app/keys
claude mcp add brave-search -s user --env BRAVE_API_KEY=YOUR_API_KEY npx -y @modelcontextprotocol/server-brave-search
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
            "prompt": "User message: $ARGUMENTS\\n\\nIf the user's message contains search, query, or fetch intent, route to appropriate MCP service:\\n\\n| Priority | MCP | Keywords |\\n|----------|-----|----------|\\n| P0 | mcp__exa__get_code_context_exa | code, programming, API, framework |\\n| P1 | mcp__metaso__metaso_web_search | academic, paper, research, study |\\n| P2 | brave_web_search | today, this week, latest, recent, news |\\n| P3 | mcp__bocha__search | chinese, 国内, 中文内容 |\\n| P4 | mcp__github__* | github, repository, issue, PR, commit |\\n| P5 | mcp__zai-mcp-server__* | image, video, screenshot, OCR |\\n| P6 | mcp__fetch__fetch | http://, https:// URL |\\n\\nIf no search intent, respond normally.",
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
Compare Chinese AI models
```

## Features

- **Smart Routing**: Auto-select best MCP based on query analysis
- **Priority-Driven**: 9-level priority system (P0-P8)
- **Cost Optimization**: Prefer free MCPs (Metaso > Bocha > Brave > Exa)
- **Graceful Degradation**: Auto-fallback on failure
- **Token Control**: Dynamic response sizing based on query complexity
- **Source Attribution**: News results include source links

## Token Control

| Complexity | Exa | Metaso | Brave | Bocha | GitHub |
|------------|-----|--------|-------|-------|--------|
| Quick | 2000-3000 | 5 | 5 | 8 | 5 |
| Regular | 5000 | 10 | 10 | 18 | 10 |
| Deep | 8000+ | 20 | 20 | 30 | 50 |

## Checklist

Before using this skill, ensure:

- [ ] Exa MCP installed: `claude mcp add exa npx -y exa-mcp-server`
- [ ] Metaso MCP configured (Docker or local)
- [ ] Brave Search MCP configured with API Key
- [ ] Bocha MCP configured
- [ ] GitHub Copilot logged in
- [ ] Zai MCP installed: `claude mcp add zai npx -y @z_ai/mcp-server`
- [ ] Fetch MCP installed: `claude mcp add fetch uvx mcp-server-fetch`
- [ ] Skill cloned to `~/.claude/skills/smart-search`
- [ ] (Optional) Hooks configured for auto-trigger

## Version History

- **v2.1.0** (2026-01-28): Add Bocha as P3, extend to 9-level priority system
- **v2.0.0** (2026-01-28): Add Brave Search, add source attribution
- **v1.3.0** (2026-01-17): Optimized structure, added bilingual README
- **v1.2.0** (2026-01-17): Add token usage control
- **v1.0.0** (2026-01-17): Initial release

## License

MIT License
