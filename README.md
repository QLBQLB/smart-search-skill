# smart-search-skill

[![English](https://img.shields.io/badge/lang-English-blue.svg)](README.md) [![Chinese](https://img.shields.io/badge/lang-中文-red.svg)](README.zh-CN.md)

Intelligent routing service - automatically select the optimal MCP search engine based on query content.

## Overview

A Claude Code skill that routes search queries to the most appropriate MCP service without manual specification. Supports 6 major MCP engines with intelligent priority-based routing.

## Supported MCP Services

| Priority | MCP | Use Case | Keywords |
|----------|-----|----------|----------|
| P0 | **Exa** | Code, API docs | code, programming, API, framework, library |
| P1 | **Metaso** | Academic search | academic, paper, research, study, journal |
| P2 | **Bocha** | Chinese news | today, this week, latest, recent, news |
| P3 | **GitHub** | Repo operations | github, repository, issue, PR, commit |
| P4 | **Zai MCP** | Media analysis | image, video, screenshot, OCR |
| P5 | **Fetch** | Direct URL | http://, https:// |

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

### Bocha (Chinese News)
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
            "prompt": "User message: $ARGUMENTS\n\nIf the user's message contains search, query, or fetch intent, route to appropriate MCP service:\n\n| Priority | MCP | Keywords |\n|----------|-----|----------|\n| P0 | mcp__exa__get_code_context_exa | code, programming, API, framework |\n| P1 | mcp__metaso__metaso_web_search | academic, paper, research, study |\n| P2 | mcp__bocha__search | today, this week, latest, recent, news |\n| P3 | mcp__github__* | github, repository, issue, PR, commit |\n| P4 | mcp__zai-mcp-server__* | image, video, screenshot, OCR |\n| P5 | mcp__fetch__fetch | http://, https:// URL |\n\nIf no search intent, respond normally.",
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

## Features

- **Smart Routing**: Auto-select best MCP based on query analysis
- **Priority-Driven**: 7-level priority system (P0-P7)
- **Cost Optimization**: Prefer free MCPs (Metaso > Bocha > Exa)
- **Graceful Degradation**: Auto-fallback on failure
- **Token Control**: Dynamic response sizing based on query complexity

## Token Control

| Complexity | Exa | Metaso | Bocha | GitHub |
|------------|-----|--------|-------|--------|
| Quick | 2000-3000 | 5 | 8 | 5 |
| Regular | 5000 | 10 | 18 | 10 |
| Deep | 8000+ | 20 | 30 | 50 |

## Checklist

Before using this skill, ensure:

- [ ] Exa MCP installed: `claude mcp add exa npx -y exa-mcp-server`
- [ ] Metaso MCP configured (Docker or local)
- [ ] Bocha MCP configured
- [ ] GitHub Copilot logged in
- [ ] Zai MCP installed: `claude mcp add zai npx -y @z_ai/mcp-server`
- [ ] Fetch MCP installed: `claude mcp add fetch uvx mcp-server-fetch`
- [ ] Skill cloned to `~/.claude/skills/smart-search`
- [ ] (Optional) Hooks configured for auto-trigger

## Version History

- **v1.3.0** (2026-01-17): Optimized structure, added bilingual README
- **v1.2.1** (2026-01-17): Adjust Bocha default to 18 items
- **v1.2.0** (2026-01-17): Add token usage control
- **v1.0.0** (2026-01-17): Initial release

## License

MIT License
