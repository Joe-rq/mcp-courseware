# 微信文章 MCP 服务器

基于 FastMCP 框架的微信文章搜索和分析服务器，使用 Playwright + Python 爬虫方案。

## 功能特点

- 搜索微信文章
- 获取文章详情
- 按公众号获取文章列表
- 获取热门文章
- 支持 JSON 和 Markdown 格式响应
- 完整的错误处理和可操作建议

## 安装

### 前置条件

1. Python 3.10+
2. Playwright 浏览器

### 安装步骤

```bash
# 克隆或进入项目目录
cd mcp-server-wechat

# 创建虚拟环境（推荐）
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# 或 .venv\Scripts\activate  # Windows

# 安装依赖
pip install -e .

# 安装 Playwright 浏览器
playwright install chromium
```

## 使用方法

### 方式 1: 使用命令行脚本（推荐）

安装后会自动创建 `mcp-server-wechat` 命令：

```bash
# 直接运行
mcp-server-wechat

# 或使用 Python 模块方式
python -m mcp_server_wechat.server
```

### 方式 2: 使用 fastmcp dev 调试

```bash
# 使用 fastmcp dev 启动开发服务器（带 MCP Inspector）
fastmcp dev src/mcp_server_wechat/server.py:mcp

# 注意：由于包结构原因，需要设置 PYTHONPATH
PYTHONPATH=src fastmcp dev src/mcp_server_wechat/server.py:mcp
```

### 方式 3: 安装到 Claude Desktop

在 Claude Desktop 配置文件中添加：

```json
{
  "mcpServers": {
    "wechat-articles": {
      "command": "python",
      "args": [
        "-m",
        "mcp_server_wechat.server"
      ],
      "env": {
        "PYTHONPATH": "/path/to/mcp-server-wechat/src"
      }
    }
  }
}
```

或使用 fastmcp install：

```bash
fastmcp install src/mcp_server_wechat/server.py:mcp --name wechat-articles
```

### 传输协议配置

默认使用 STDIO 传输协议，适用于本地/Claude Desktop。

如需使用 HTTP 传输协议（用于远程访问/多客户端），修改 `server.py` 中的运行配置：

```python
# 使用 HTTP 传输协议
mcp.run(transport="http", host="127.0.0.1", port=8000)
```

## 环境变量

- `WECHAT_SCRAPER_TIMEOUT`: 爬虫超时时间（秒），默认 30
- `WECHAT_SCRAPER_RETRIES`: 重试次数，默认 3
- `WECHAT_SCRAPER_HEADLESS`: 无头模式（true/false），默认 true

## 已实现工具

1. `search_wechat_articles`: 搜索微信文章
2. `get_wechat_article`: 获取文章详情
3. `list_wechat_articles_by_account`: 按公众号获取文章列表
4. `get_trending_wechat_articles`: 获取热门文章

## 调试技巧

使用 `fastmcp dev` 进行调试时，可以：

1. 在 MCP Inspector UI 中测试工具调用
2. 查看请求和响应日志
3. 检查错误和异常
4. 实时修改代码并自动重新加载

## 注意事项

- 爬虫可能受到反爬虫机制影响，请合理设置重试和延迟
- 遵守相关法律法规和网站使用条款
- 大量请求可能导致 IP 被临时限制