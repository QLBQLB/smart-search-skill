# smart-search-skill

[![English](https://img.shields.io/badge/lang-English-blue.svg)](README.md) [![Chinese](https://img.shields.io/badge/lang-中文-red.svg)](README.zh-CN.md)

智能路由搜索服务 - 根据查询内容自动选择最合适的 MCP 搜索引擎。

## 概述

这是一个 Claude Code 技能，能够根据用户查询类型自动选择最合适的 MCP 搜索服务，无需手动指定。支持 7 个主要的 MCP 引擎，采用 9 级智能优先级路由。

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

### Brave (英文新闻)
```bash
claude mcp add brave npx -y @modelcontextprotocol/server-brave-search
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

### search-aggregator (可选，推荐)

用于自动聚合多个搜索引擎结果并去重：

```bash
# 克隆 search-aggregator 技能
git clone https://github.com/QLBQLB/search-aggregator-skill.git ~/.claude/skills/search-aggregator

# 配置 MCP Server
cd ~/.claude/skills/search-aggregator
npm install
npm run build
claude mcp add search-aggregator node C:\\Users\\uiqia\\.claude\\skills\\search-aggregator\\dist\\index.js
```

**search-aggregator 提供的工具**:

| 工具 | 用途 |
|------|------|
| `hybrid_research` | 自动检测查询类型，返回应调用的引擎列表 |
| `aggregate_search` | 聚合多引擎结果并自动去重 |
| `quick_search` | 简单查询的快速搜索（带缓存） |
| `cache_info` | 查看和管理缓存 |

## 与 search-aggregator 搭配使用

### 架构关系

```
┌─────────────────────────────────────────────────────────┐
│                    smart-search (路由层)                  │
│              决定使用哪个/哪些搜索引擎                      │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│              search-aggregator (聚合层)                   │
│         并发调用 + 自动去重 + 结果缓存                     │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│   Exa    │  Brave   │  Metaso  │  Bocha   │  GitHub  │
│ (代码/API)│  (新闻)  │  (学术)  │  (中文)  │  (项目)  │
└──────────┴──────────┴──────────┴──────────┴──────────┘
```

### 自动集成流程

当 `search-aggregator` MCP Server 可用时，自动使用以下流程：

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

Step 3: 聚合去重
  调用: mcp__search_aggregator__aggregate_search(
    query,
    engine_results: { Exa: [...], Brave: [...], ... },
    max_per_domain: 2
  )
  返回: { results, stats: { totalOriginal, totalAfterDedup, dedupRate } }

Step 4: 格式化输出
  使用智能分析的标准输出格式
```

### 模式检测示例

| 查询 | 返回模式 | 调用引擎 |
|------|----------|----------|
| "Python 怎么打印" | Quick | Exa |
| "分析 Rust 采用趋势" | Mode-A | Exa + Brave + Metaso |
| "国内大模型竞争格局" | Mode-R | Brave + Bocha + Metaso |
| "Next.js vs Remix" | Mode-T | Exa + GitHub |
| "全面分析 Agent 技术" | Mode-F | 全部引擎 |

### 去重效果示例

```yaml
输入: 15 条原始结果
  - Exa: 5 条
  - Brave: 6 条
  - Metaso: 4 条

去重处理:
  - URL 规范化 (移除 ?ref=*, &utm_*)
  - 域名分组 (同域名合并)
  - 相似度判断 (>80% 视为重复)

输出: 8 条去重结果
  - 去重率: 46.7%
  - 同域名最多保留 2 条
```

### 缓存策略

| 场景 | TTL | 说明 |
|------|-----|------|
| 新闻类查询 | 3 分钟 | 时效性高 |
| 技术文档 | 10 分钟 | 相对稳定 |
| 学术论文 | 30 分钟 | 长期有效 |

### 回退机制

当 `search-aggregator` 不可用时：

1. 使用智能路由的 Hybrid Recall Mode 手动检测
2. 手动并发调用搜索引擎
3. 手动执行 URL 去重
4. 手动格式化输出

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
            "prompt": "用户消息: $ARGUMENTS\n\n如果用户消息包含搜索、查询、获取网络信息等意图，则根据以下规则选择合适的MCP服务：\n\n| 优先级 | MCP服务 | 触发关键词 |\n|--------|---------|------------|\n| P0 | mcp__exa__get_code_context_exa | 代码、编程、API、函数、框架、tutorial、syntax |\n| P1 | mcp__metaso__metaso_web_search | 学术、论文、research、study、journal、academic |\n| P2 | mcp__brave-search__brave_web_search | today, this week, latest, recent, news |\n| P3 | mcp__bocha__search | 中文、国内、百科、Chinese content |\n| P4 | mcp__github__* | github、仓库、issue、PR、commit、branch |\n| P5 | mcp__zai-mcp-server__* | 图片、视频、截图、OCR、analyze picture |\n| P6 | mcp__fetch__fetch | http://、https:// URL |\n| P7 | mcp__brave-search__brave_web_search | 默认回退 |\n| P8 | mcp__exa__get_code_context_exa | 英文技术文档 |\n\n如果用户消息不涉及搜索，则正常响应用户。",
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
- **双模式搜索**：Quick Search (快速) 和 Deep Research (深度)
- **并发召回**：支持 Mode-A/R/T/F 四种并发模式
- **URL 去重**：自动规范化 URL 并去重，同域名最多保留 2 条
- **时效性过滤**：新闻类查询只保留 48 小时内的内容
- **优雅降级**：失败时自动切换备选方案
- **Token 控制**：根据查询复杂度和并发状态动态调整

## Token 控制

### 单路召回 (Quick Search)

| 引擎 | 快速 | 常规 | 深度 |
|------|------|------|------|
| Exa | 3000 | 5000 | 8000+ |
| Metaso | 5 | 10 | 20 |
| Brave | 5 | 10 | 20 |
| Bocha | 8 | 18 | 30 |
| GitHub | 5 | 10 | 50 |

### 并发召回 (Deep Research)

| 引擎 | 单路预算 | 并发预算 | 说明 |
|------|----------|----------|------|
| Exa | 5000 | 3000 | 减半避免超限 |
| Brave | 10 | 5 | 控制结果数 |
| Bocha | 18 | 5 | 控制结果数 |
| Metaso | 10 | 3 | 精选高质量 |
| GitHub | 10 | 10 | 保持原量 |
| Fetch | 5000 | 3000 | 按需使用 |

**并发后总预算**: 约 8000-10000 tokens (去重后实际更少)

## URL 去重规则

并发模式下自动执行以下去重流程：

1. **URL 规范化**：移除追踪参数 (?ref=*, &utm_*)，统一协议
2. **域名分组**：按主域名分组结果
3. **相似度判断**：路径相似度 > 80% 视为同一文章
4. **数量限制**：同一域名最多保留 2 条（按相关性排序）

## 时效性判断

新闻类查询的时效性过滤规则：

| 指标 | 新鲜 | 过期 |
|------|------|------|
| 发布日期 | 48 小时内 | 超过 7 天 |
| URL 日期字符串 | 包含当前日期 | 无日期或旧日期 |
| 内容关键词 | "今日", "最新", "just announced" | "历史", "回顾", "总览" |

## AI 新闻输出格式

当查询"今日AI新闻"时，使用以下格式：

```markdown
# 今日AI资讯 Top 8

### 1. [新闻标题]

**摘要：** [新闻内容摘要，50-100字]

**来源：** [来源名](链接)

**标签：** #标签1 #标签2 #标签3

**相关性：** ★★★★★

---
```

## 回退策略

| 主引擎 | 回退1 | 回退2 |
|--------|-------|-------|
| Exa | Brave | Metaso |
| Metaso | Bocha | Brave |
| Brave | Bocha | Exa |
| Bocha | Brave | Exa |
| GitHub | Fetch | - |
| Fetch | Brave | - |

## 检查清单

使用本技能前，请确认：

### 基础 MCP 服务
- [ ] 已安装 Exa MCP: `claude mcp add exa npx -y exa-mcp-server`
- [ ] 已安装 Brave MCP: `claude mcp add brave npx -y @modelcontextprotocol/server-brave-search`
- [ ] 已配置 Metaso MCP（Docker 或本地）
- [ ] 已配置 Bocha MCP
- [ ] 已登录 GitHub Copilot
- [ ] 已安装 Zai MCP: `claude mcp add zai npx -y @z_ai/mcp-server`
- [ ] 已安装 Fetch MCP: `claude mcp add fetch uvx mcp-server-fetch`

### search-aggregator (推荐)
- [ ] 已克隆 search-aggregator 技能到 `~/.claude/skills/search-aggregator`
- [ ] 已配置 search-aggregator MCP Server
- [ ] 验证工具可用: `hybrid_research`, `aggregate_search`, `quick_search`, `cache_info`

### 技能配置
- [ ] smart-search 技能已克隆到 `~/.claude/skills/smart-search`
- [ ] （可选）已配置 hooks 实现自动触发

## 版本历史

- **v2.1.0** (2026-02-03): 添加 search-aggregator 集成说明和搭配使用指南
- **v2.0.0** (2026-02-03): 重构搜索模式，添加并发召回、URL去重、时效性过滤
- **v1.3.0** (2026-01-17): 优化结构，添加中英双语 README
- **v1.2.1** (2026-01-17): 调整 Bocha 默认值为 18 条
- **v1.2.0** (2026-01-17): 添加 Token 用量控制
- **v1.0.0** (2026-01-17): 初始版本

## 许可证

MIT License
