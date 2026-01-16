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

1. 克隆此仓库到你的技能目录：

```bash
git clone https://github.com/QLBQLB/smart-search-skill.git ~/.claude/skills/smart-search
```

2. 或者在 Windows 上：

```bash
git clone https://github.com/QLBQLB/smart-search-skill.git C:\Users\你的用户名\.claude\skills\smart-search
```

## 使用

在 Claude Code 中使用：

```
/smart-search 搜索内容
```

### 示例

```
/smart-search Python FastAPI 教程
/smart-search 深度学习最新论文
/smart-search 今天的科技新闻
/smart-search 搜索 Python 爬虫开源项目
```

## 智能路由规则

- **代码搜索** → Exa Code Context
- **学术搜索** → Metaso
- **中文资讯** → Bocha
- **GitHub 操作** → GitHub MCP
- **图片/视频分析** → Zai MCP
- **URL 直接获取** → Fetch

## 特性

- 智能路由：根据查询内容自动选择最优 MCP
- 优先级驱动：严格遵循 7 级优先级规则
- 成本优化：优先使用免费 MCP
- 智能降级：首选 MCP 失败时自动切换备选方案

## 版本历史

- **v1.0.0** (2026-01-17): 初始版本，定义 6 个 MCP 的智能路由规则

## 许可证

MIT License
