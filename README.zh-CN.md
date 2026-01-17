# smart-search-skill

[![English](https://img.shields.io/badge/lang-English-blue.svg)](README.md) [![Chinese](https://img.shields.io/badge/lang-中文-red.svg)](README.zh-CN.md)

智能路由搜索服务 - 根据查询内容自动选择最合适的 MCP 搜索引擎。

## 概述

这是一个 Claude Code 技能，能够根据用户查询类型自动选择最合适的 MCP 搜索服务，无需手动指定。支持 6 个主要的 MCP 引擎，采用智能优先级路由。

## 支持的 MCP 服务

| 优先级 | MCP | 用途 | 关键词 |
|----------|-----|----------|----------|
| P0 | **Exa** | 代码、API 文档 | 代码、编程、API、框架、库 |
| P1 | **Metaso** | 学术搜索 | 学术、论文、研究、期刊 |
| P2 | **Bocha** | 中文资讯 | 今天、本周、最新、最近、新闻 |
| P3 | **GitHub** | 仓库操作 | github、仓库、issue、PR、commit |
| P4 | **Zai MCP** | 多媒体分析 | 图片、视频、截图、OCR |
| P5 | **Fetch** | 直接 URL | http://, https:// |

## 安装

### 方法一：Claude Code Marketplace

```bash
/plugin marketplace add QLBQLB/smart-search-skill
```

然后安装：
```bash
/plugin install smart-search@smart-search-skill
```

### 方法二：手动克隆

```bash
# macOS / Linux
git clone https://github.com/QLBQLB/smart-search-skill.git ~/.claude/skills/smart-search

# Windows (PowerShell)
git clone https://github.com/QLBQLB/smart-search-skill.git $env:USERPROFILE\.claude\skills\smart-search
```

## MCP 配置

每个 MCP 服务需要单独配置：

### Exa (代码搜索)
```bash
claude mcp add exa npx -y exa-mcp-server
```

### Metaso (学术搜索)
```bash
# 克隆并运行（需要 Docker 或 Node.js）
git clone https://github.com/metaso-ai/search-server-metaso.git
cd search-server-metaso
docker-compose up -d  # 或 npm start
claude mcp add metaso http://localhost:3000/sse
```

### Bocha (中文资讯)
```bash
# 配置你的 Bocha 搜索服务器
claude mcp add bocha http://localhost:3001/sse
```

### GitHub (内置)
需要 GitHub Copilot 订阅。在 Claude Code 中自动可用。

### Zai MCP (多媒体分析)
```bash
claude mcp add zai npx -y @z_ai/mcp-server
```

### Fetch (URL 内容获取)
```bash
claude mcp add fetch uvx mcp-server-fetch
```

## 使用方式

### 直接调用
```
/smart-search Python FastAPI 教程
```

### 自动触发（推荐）

配置 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "用户消息: $ARGUMENTS\n\n如果用户消息包含搜索、查询、获取网络信息等意图，则根据以下规则选择合适的MCP服务：\n\n| 优先级 | MCP服务 | 触发关键词 |\n|--------|---------|------------|\n| P0 | mcp__exa__get_code_context_exa | 代码、编程、API、函数、框架 |\n| P1 | mcp__metaso__metaso_web_search | 学术、论文、research、study |\n| P2 | mcp__bocha__search | 今天、本周、最新、最近、news |\n| P3 | mcp__github__* | github、仓库、issue、PR、commit |\n| P4 | mcp__zai-mcp-server__* | 图片、视频、截图、OCR |\n| P5 | mcp__fetch__fetch | http://、https:// URL |\n\n如果用户消息不涉及搜索，则正常响应用户。",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

然后直接输入：
```
搜索 Python FastAPI 异步编程
查找深度学习最新论文
今天的科技新闻
```

## 特性

- **智能路由**：根据查询分析自动选择最佳 MCP
- **优先级驱动**：7 级优先级系统 (P0-P7)
- **成本优化**：优先使用免费 MCP (Metaso > Bocha > Exa)
- **优雅降级**：失败时自动切换备选方案
- **Token 控制**：根据查询复杂度动态调整响应大小

## Token 控制

| 复杂度 | Exa | Metaso | Bocha | GitHub |
|------------|-----|--------|-------|--------|
| 快速 | 2000-3000 | 5 | 8 | 5 |
| 常规 | 5000 | 10 | 18 | 10 |
| 深度 | 8000+ | 20 | 30 | 50 |

## 检查清单

使用本技能前，请确认：

- [ ] 已安装 Exa MCP: `claude mcp add exa npx -y exa-mcp-server`
- [ ] 已配置 Metaso MCP（Docker 或本地）
- [ ] 已配置 Bocha MCP
- [ ] 已登录 GitHub Copilot
- [ ] 已安装 Zai MCP: `claude mcp add zai npx -y @z_ai/mcp-server`
- [ ] 已安装 Fetch MCP: `claude mcp add fetch uvx mcp-server-fetch`
- [ ] 技能已克隆到 `~/.claude/skills/smart-search`
- [ ] （可选）已配置 hooks 实现自动触发

## 版本历史

- **v1.3.0** (2026-01-17): 优化结构，添加中英双语 README
- **v1.2.1** (2026-01-17): 调整 Bocha 默认值为 18 条
- **v1.2.0** (2026-01-17): 添加 Token 用量控制
- **v1.0.0** (2026-01-17): 初始版本

## 许可证

MIT License
