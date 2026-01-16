# smart-search-skill

智能路由搜索服务 - 根据查询内容自动选择最合适的MCP搜索引擎

## 简介

这是一个 Claude Code 技能，能够根据用户查询类型自动选择最合适的 MCP 搜索引擎，无需手动指定。

## 支持的 MCP 服务

| 优先级 | MCP服务 | 主要用途 | 关键词触发 |
|--------|---------|----------|------------|
| P0 | **Exa** | 代码搜索、英文文档、API参考 | 代码、编程、API、函数、框架 |
| P1 | **Metaso** | 学术搜索、AI总结、深度研究 | 学术、论文、research、study |
| P2 | **Bocha** | 中文资讯、时间过滤、极速搜索 | 今天、本周、最新、最近、news |
| P3 | **GitHub** | 仓库操作、Issue管理、PR审查 | github、仓库、issue、PR、commit |
| P4 | **Zai MCP** | 图像分析、视频分析、OCR | 图片、视频、截图、分析图像、OCR |
| P5 | **Fetch** | 直接获取URL内容 | http://、https:// |

## 安装

### 方式一：手动安装

1. 克隆此仓库到你的技能目录：

```bash
# macOS / Linux
git clone https://github.com/QLBQLB/smart-search-skill.git ~/.claude/skills/smart-search

# Windows
git clone https://github.com/QLBQLB/smart-search-skill.git C:\Users\你的用户名\.claude\skills\smart-search
```

### 方式二：自动安装（推荐）

运行以下命令自动克隆到正确的目录：

```bash
# macOS / Linux
git clone https://github.com/QLBQLB/smart-search-skill.git ~/.claude/skills/smart-search

# Windows (PowerShell)
git clone https://github.com/QLBQLB/smart-search-skill.git $env:USERPROFILE\.claude\skills\smart-search
```

---

## 使用方式

### 方式一：手动调用（无需配置）

安装后，在 Claude Code 中直接使用：

```
/smart-search 搜索内容
```

**示例**：
```
/smart-search Python FastAPI 教程
/smart-search 深度学习最新论文
/smart-search 今天的科技新闻
```

### 方式二：自动触发（推荐）

通过配置 `hooks`，让每次会话自动检测搜索意图并选择合适的 MCP 服务。

#### 配置步骤

1. **找到你的 Claude Code 设置文件**：

   - Windows: `C:\Users\你的用户名\.claude\settings.json`
   - macOS / Linux: `~/.claude/settings.json`

2. **在 `settings.json` 中添加 `hooks` 配置**：

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "用户消息: $ARGUMENTS\n\n如果用户消息包含搜索、查询、获取网络信息等意图（如：搜索、查找、search、百度、google、最新、今天、新闻、论文、学术、代码示例、API文档、github仓库等），则根据以下规则选择合适的MCP服务并调用：\n\n| 优先级 | MCP服务 | 触发关键词 |\n|--------|---------|------------|\n| P0 | mcp__exa__get_code_context_exa | 代码、编程、API、函数、框架、library、tutorial、example、syntax、bug |\n| P1 | mcp__metaso__metaso_web_search | 学术、论文、research、study、期刊、文献、引用 |\n| P2 | mcp__bocha__search | 今天、本周、最新、最近、news、时间范围、中文资讯 |\n| P3 | mcp__github__* | github、仓库、issue、PR、commit、分支 |\n| P4 | mcp__zai-mcp-server__* | 图片、视频、截图、分析图像、OCR |\n| P5 | mcp__fetch__fetch | http://、https:// URL直接获取 |\n\n如果用户消息不涉及搜索，则正常响应用户。如果涉及搜索，请先输出分析结果（使用哪个MCP及原因），然后执行搜索。",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

3. **完整配置示例**（与现有配置合并）：

```json
{
  "model": "opus",
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

4. **重启 Claude Code** 使配置生效。

#### 配置后的效果

配置完成后，你可以直接发送搜索请求，无需加前缀：

```
搜索 Python FastAPI 教程
查找深度学习最新论文
今天的科技新闻
搜索 Python 爬虫开源项目
```

系统会自动识别意图并选择对应的 MCP 服务执行搜索。

---

## 智能路由规则

| 查询类型 | 示例 | 路由到 |
|----------|------|--------|
| **代码搜索** | Python async await 示例 | Exa Code Context |
| **学术搜索** | 深度学习最新论文 | Metaso |
| **中文资讯** | 今天的科技新闻 | Bocha |
| **GitHub 操作** | 搜索 Python 爬虫项目 | GitHub MCP |
| **图片分析** | 分析这张图片 | Zai MCP |
| **URL 获取** | 获取 https://example.com | Fetch |

---

## 前置要求

要使用自动路由功能，你需要先配置以下 MCP 服务：

| MCP 服务 | 安装命令 | 说明 |
|----------|----------|------|
| **Exa** | `npx -y exa-mcp-server` | 代码搜索 |
| **Metaso** | 自建或使用第三方服务 | 学术搜索 |
| **Bocha** | 自建或使用第三方服务 | 中文资讯 |
| **GitHub** | 内置（需 GitHub Copilot） | GitHub 操作 |
| **Zai MCP** | `npx -y @z_ai/mcp-server` | 图像/视频分析 |
| **Fetch** | `uvx mcp-server-fetch` | URL 获取 |

配置 MCP 服务的方法：
```bash
claude mcp add exa npx -y exa-mcp-server
claude mcp add fetch uvx mcp-server-fetch
claude mcp add zai npx -y @z_ai/mcp-server
```

---

## 特性

- **智能路由**：根据查询内容自动选择最优 MCP
- **优先级驱动**：严格遵循 7 级优先级规则
- **成本优化**：优先使用免费 MCP
- **智能降级**：首选 MCP 失败时自动切换备选方案
- **自动触发**：配置 hooks 后无需手动调用

---

## 版本历史

- **v1.1.0** (2026-01-17): 添加自动触发配置说明
- **v1.0.0** (2026-01-17): 初始版本，定义 6 个 MCP 的智能路由规则

---

## 许可证

MIT License
