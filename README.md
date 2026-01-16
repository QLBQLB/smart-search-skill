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

---

## MCP 服务详细配置

### 1. Exa - 代码搜索

Exa 提供高质量的代码搜索和 API 文档检索功能。

#### 安装步骤

```bash
claude mcp add exa npx -y exa-mcp-server
```

#### 验证安装

```bash
claude mcp list
```

确认 `exa` 在列表中显示为 `connected`。

#### 可用工具

- `mcp__exa__get_code_context_exa` - 代码上下文搜索
- `mcp__exa__web_search_exa` - Web 搜索

---

### 2. Metaso - 学术搜索

Metaso 专注于学术论文、研究文献的搜索和 AI 总结。

#### 安装步骤

##### 方法 A：使用官方 Docker 服务

```bash
# 1. 克隆仓库
git clone https://github.com/metaso-ai/search-server-metaso.git

# 2. 进入目录
cd search-server-metaso

# 3. 启动服务
docker-compose up -d
```

##### 方法 B：本地 Node.js 运行

```bash
# 1. 克隆仓库
git clone https://github.com/metaso-ai/search-server-metaso.git

# 2. 安装依赖
cd search-server-metaso
npm install

# 3. 启动服务
npm start
```

#### 配置 MCP

服务启动后（默认端口 3000），添加到 Claude Code：

```bash
claude mcp add metasio http://localhost:3000/sse
```

#### 可用工具

- `mcp__metaso__metaso_web_search` - 学术 Web 搜索
- `mcp__metaso__metaso_chat` - AI 对话式问答
- `mcp__metaso__metaso_web_reader` - 网页内容读取

---

### 3. Bocha - 中文资讯搜索

Bocha 专注于中文内容搜索，支持时间过滤和最新资讯获取。

#### 安装步骤

##### 方法 A：使用官方 Docker 服务

```bash
# 1. 克隆仓库
git clone https://github.com/your-org/search-server.git

# 2. 进入目录
cd search-server

# 3. 启动服务
docker-compose up -d
```

##### 方法 B：本地运行

```bash
# 1. 安装依赖
npm install -g @bocha/search-server

# 2. 启动服务
bocha-server --port 3001
```

#### 配置 MCP

```bash
claude mcp add bocha http://localhost:3001/sse
```

#### 可用工具

- `mcp__bocha__search` - 中文搜索（支持时间范围过滤）

---

### 4. GitHub MCP - 仓库操作

GitHub MCP 由 Claude Code 内置支持，需要 GitHub Copilot 订阅。

#### 启用步骤

1. 确保 GitHub Copilot 已订阅并登录
2. 在 Claude Code 设置中确认 GitHub 连接：

```json
{
  "enabledMcpjsonServers": ["github"]
}
```

#### 验证连接

```bash
claude mcp doctor
```

#### 可用工具

- `mcp__github__search_repositories` - 搜索仓库
- `mcp__github__search_code` - 搜索代码
- `mcp__github__create_repository` - 创建仓库
- `mcp__github__push_files` - 推送文件
- `mcp__github__create_pull_request` - 创建 PR
- `mcp__github__issue_write` - 创建/更新 Issue
- `mcp__github__list_commits` - 列出提交记录
- 等等...

---

### 5. Zai MCP - 图像/视频分析

Zai MCP 提供图像分析、视频分析、OCR 等多媒体处理能力。

#### 安装步骤

```bash
claude mcp add zai npx -y @z_ai/mcp-server
```

#### 验证安装

```bash
claude mcp list
```

#### 可用工具

- `mcp__zai-mcp-server__analyze_image` - 通用图像分析
- `mcp__zai-mcp-server__extract_text_from_screenshot` - OCR 文字提取
- `mcp__zai-mcp-server__diagnose_error_screenshot` - 错误截图诊断
- `mcp__zai-mcp-server__ui_to_artifact` - UI 转代码
- `mcp__zai-mcp-server__analyze_video` - 视频内容分析
- `mcp__zai-mcp-server__analyze_data_visualization` - 数据图表分析

---

### 6. Fetch - URL 内容获取

Fetch 用于直接获取指定 URL 的网页内容。

#### 安装步骤

##### 方法 A：使用 uvx（推荐）

```bash
claude mcp add fetch uvx mcp-server-fetch
```

##### 方法 B：使用 npx

```bash
claude mcp add fetch npx -y @modelcontextprotocol/server-fetch
```

#### 验证安装

```bash
claude mcp list
```

#### 可用工具

- `mcp__fetch__fetch` - 获取 URL 内容（支持 markdown 转换）

---

### MCP 配置汇总

将以上所有 MCP 添加后，你的 `settings.json` 中 `mcpServers` 部分应类似：

```json
{
  "mcpServers": {
    "exa": {
      "command": "npx",
      "args": ["-y", "exa-mcp-server"]
    },
    "metaso": {
      "url": "http://localhost:3000/sse"
    },
    "bocha": {
      "url": "http://localhost:3001/sse"
    },
    "zai": {
      "command": "npx",
      "args": ["-y", "@z_ai/mcp-server"]
    },
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

---

## 技能安装

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

### 方式一：手动调用（无需额外配置）

安装技能后，在 Claude Code 中直接使用：

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

## 快速开始检查清单

使用本技能前，请确认以下步骤：

- [ ] 已安装 Node.js / npm（用于 Exa、Zai MCP）
- [ ] 已安装 Python / uvx（用于 Fetch）
- [ ] 已添加 Exa MCP: `claude mcp add exa npx -y exa-mcp-server`
- [ ] 已配置 Metaso MCP（Docker 或本地运行）
- [ ] 已配置 Bocha MCP（Docker 或本地运行）
- [ ] 已登录 GitHub Copilot（GitHub MCP）
- [ ] 已添加 Zai MCP: `claude mcp add zai npx -y @z_ai/mcp-server`
- [ ] 已添加 Fetch MCP: `claude mcp add fetch uvx mcp-server-fetch`
- [ ] 已克隆本技能到 `~/.claude/skills/smart-search`
- [ ] （可选）已配置 hooks 实现自动触发

---

## 特性

- **智能路由**：根据查询内容自动选择最优 MCP
- **优先级驱动**：严格遵循 7 级优先级规则
- **成本优化**：优先使用免费 MCP
- **智能降级**：首选 MCP 失败时自动切换备选方案
- **自动触发**：配置 hooks 后无需手动调用

---

## 故障排查

### MCP 服务连接失败

```bash
# 检查 MCP 服务状态
claude mcp doctor

# 查看已配置的 MCP 列表
claude mcp list

# 查看 MCP 详细信息
claude mcp show <服务名>
```

### Docker 服务无法启动

```bash
# 检查 Docker 状态
docker ps

# 查看服务日志
docker-compose logs -f
```

### 技能无法加载

确认技能目录结构正确：
```
~/.claude/skills/smart-search/
└── SKILL.md
```

---

## 版本历史

- **v1.2.0** (2026-01-17): 添加详细的 MCP 服务配置说明
- **v1.1.0** (2026-01-17): 添加自动触发配置说明
- **v1.0.0** (2026-01-17): 初始版本，定义 6 个 MCP 的智能路由规则

---

## 许可证

MIT License
