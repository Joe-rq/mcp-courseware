# 微信文章 API 调研报告

## 1. API 概述

### 1.1 调研目标
寻找能够获取微信文章内容的API，包括官方微信API和第三方API方案。

### 1.2 调研方法
- 搜索微信官方开放平台文档
- 调研第三方API服务提供商
- 分析API功能和限制

## 2. 官方微信 API

### 2.1 微信公众号平台API

**认证要求**：
- 注册微信公众号
- 获取appid和AppSecret
- 设置IP白名单
- 通过access_token调用接口

**可用操作**：
- 新增素材
- 新建草稿
- 发布文章

**限制**：
- 主要面向公众号运营者
- 无法直接获取已发布的公众号文章内容
- 功能受限，不适合Agent读取场景

### 2.2 微信开放平台API

**可用功能**：
- 小程序基础API
- 第三方平台开发
- 用户授权登录

**限制**：
- 未直接提供公众号文章获取API
- 主要面向小程序开发

## 3. 第三方 API 方案

### 3.1 天聚数行TianAPI

**服务名称**：微信文章精选API接口

**功能特点**：
- 根据关键词搜索微信文章
- 支持数量、页码等参数
- 返回文章ID、标题、描述、图片、URL、公众号名称等

**返回数据示例**：
```json
{
  "code": 200,
  "msg": "success",
  "data": [
    {
      "id": "文章ID",
      "title": "文章标题",
      "description": "文章描述",
      "picurl": "封面图片URL",
      "url": "文章链接",
      "account": "公众号名称"
    }
  ]
}
```

**认证方式**：
- API Key认证
- 付费服务

**优势**：
- 接口简单易用
- 无需公众号认证
- 直接获取文章内容

### 3.2 EasyWeChat SDK

**功能**：
- PHP微信开发SDK
- 支持微信开放平台第三方平台
- 可用于获取access_token等

**限制**：
- 主要面向开发者
- 需要服务器部署
- 功能相对复杂

## 4. API 选择决策

### 4.1 选择第三方API的理由

**官方API限制**：
- 无法直接获取已发布文章内容
- 认证流程复杂
- 功能受限

**第三方API优势**：
- 接口简单直接
- 无需复杂认证
- 适合Agent使用场景
- 数据格式规范

### 4.2 推荐API：天聚数行TianAPI

**理由**：
1. 专门提供微信文章获取服务
2. 接口设计简洁
3. 返回数据完整
4. 文档清晰
5. 适合MCP集成

## 5. API 技术细节

### 5.1 认证方式

**API Key认证**：
- 在请求头中添加API Key
- 支持多个Key轮换
- 环境变量配置

### 5.2 请求格式

```http
GET /api/weixin/article?key=API_KEY&keyword=搜索关键词&num=10&page=1
```

**参数说明**：
- `key`：API密钥（必填）
- `keyword`：搜索关键词（必填）
- `num`：返回数量（可选，默认10）
- `page`：页码（可选，默认1）

### 5.3 响应格式

**成功响应**：
```json
{
  "code": 200,
  "msg": "success",
  "data": [
    {
      "id": "string",
      "title": "string",
      "description": "string",
      "picurl": "string",
      "url": "string",
      "account": "string"
    }
  ]
}
```

**错误响应**：
```json
{
  "code": 错误码,
  "msg": "错误信息"
}
```

## 6. 限制和约束

### 6.1 速率限制
- 需要了解具体API的速率限制
- 建议实现请求队列
- 支持自动重试

### 6.2 数据限制
- 每次请求最大返回数量
- 搜索关键词长度限制
- 分页支持

### 6.3 内容限制
- 可能无法获取所有公众号文章
- 内容更新延迟
- 部分文章可能无法访问

## 7. 集成考虑

### 7.1 错误处理
- API Key无效
- 请求频率超限
- 网络超时
- 数据格式错误

### 7.2 性能优化
- 请求缓存
- 连接池
- 超时设置
- 重试机制

### 7.3 安全性
- API Key安全存储
- 请求加密（如果支持）
- 数据隐私保护

## 8. 模拟浏览器爬虫方案

### 8.1 Playwright + Python 方案

**技术优势**：
- 完全模拟真实浏览器行为
- 支持JavaScript动态内容加载
- 自动处理Cookie和会话管理
- 支持多种浏览器引擎（Chromium、Firefox、WebKit）
- 现代且功能强大的API

**核心功能**：
```python
from playwright.sync_api import sync_playwright

def crawl_wechat_article(url):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        
        # 设置用户代理，模拟真实浏览器
        page.set_extra_http_headers({
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
        })
        
        # 访问微信公众号文章
        page.goto(url, wait_until='networkidle')
        
        # 等待内容加载
        page.wait_for_selector(".rich_media_content", timeout=10000)
        
        # 获取文章内容
        title = page.query_selector("#activity-name").inner_text().strip()
        content = page.query_selector(".rich_media_content").inner_text()
        account_name = page.query_selector("#js_name").inner_text().strip()
        
        browser.close()
        
        return {
            "title": title,
            "content": content,
            "account_name": account_name,
            "url": url
        }
```

**技术特点**：
- 无头浏览器模式，节省资源
- 自动等待机制，确保内容完全加载
- 支持代理服务器配置
- 错误处理和重试机制
- 支持截图和调试

### 8.2 其他模拟浏览器方案

**Selenium**：
- 最成熟的方案，社区支持完善
- 支持多种编程语言
- 配置相对复杂

**Puppeteer/Pyppeteer**：
- Google官方支持
- 专注于Chrome浏览器
- API简洁易用

## 9. API 选择决策

### 9.1 选择 Playwright + Python 的理由

**技术优势**：
1. 完全控制数据获取过程
2. 无需依赖第三方API服务
3. 能够获取最新发布的文章
4. 支持自定义数据提取逻辑
5. 避免API服务不稳定问题

**成本优势**：
- 无需支付API调用费用
- 开源免费使用
- 长期维护成本低

**法律合规性**：
- 遵守robots.txt规则
- 控制请求频率
- 仅用于个人学习和研究

### 9.2 推荐方案：Playwright + Python

**作为主要技术方案**：
- 技术成熟度：高
- 开发复杂度：中等
- 维护成本：低
- 数据可靠性：高

**实施要点**：
1. 合理控制请求频率
2. 实现错误重试机制
3. 添加请求延迟
4. 支持代理轮换
5. 遵守网站使用条款

## 10. 结论

**推荐使用 Playwright + Python**作为微信文章MCP的技术方案：
- 技术先进且功能强大
- 完全控制数据获取流程
- 避免第三方API依赖
- 适合长期维护和发展

**注意事项**：
- 需要合理控制爬取频率
- 遵守网站的使用条款
- 考虑网络稳定性
- 实现完善的错误处理