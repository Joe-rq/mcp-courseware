# 微信文章 MCP Server 规范

## 1. 执行摘要

**MCP Server 名称**: `wechat-articles`
**目标服务**: 微信文章内容获取
**主要用途**: 让AI Agent能够搜索、读取和分析微信文章内容
**传输协议**: STDIO

## 2. 目标服务

### 2.1 集成服务
- **主要服务**: 微信公众平台
- **技术方案**: Playwright + Python 爬虫
- **服务类型**: 内容聚合和数据提供

### 2.2 服务特点
- 提供微信文章搜索和获取功能
- 支持关键词搜索、公众号筛选
- 返回结构化文章数据
- 适合研究和分析用途

## 3. 主要用例

### 3.1 内容研究
- 搜索特定主题的微信文章
- 分析行业趋势和观点
- 收集参考资料

### 3.2 信息聚合
- 跟踪特定公众号内容
- 获取热门文章
- 内容摘要生成

### 3.3 数据分析
- 文章内容分析
- 趋势发现
- 观点提取

## 4. API 研究摘要

### 4.1 技术架构
- **核心组件**: Playwright + Python
- **浏览器模拟**: 无头浏览器自动化
- **数据提取**: HTML解析和内容提取

### 4.2 认证方式
- **类型**: 无API密钥要求
- **机制**: 模拟真实浏览器行为
- **优势**: 无需第三方服务依赖

### 4.3 核心功能
- 微信搜索页面自动化
- 公众号页面内容提取
- 文章详情页面解析
- 热门文章列表获取

### 4.4 数据模型
```typescript
interface WechatArticle {
  id: string;
  title: string;
  description: string;
  picurl: string;
  url: string;
  account: string;
}

interface ApiResponse {
  code: number;
  msg: string;
  data: WechatArticle[];
}
```

### 4.5 分页
- 支持分页参数：page, num
- 默认每页10条记录
- 最大每页100条记录

### 4.6 错误响应
```json
{
  "code": 错误码,
  "msg": "错误描述"
}
```

## 5. 工具设计

### 5.1 工具选择理由

基于Agent使用场景，设计以下工具：

1. **搜索工具** (`search_wechat_articles`): 核心功能，支持关键词搜索
2. **内容获取工具** (`get_wechat_article`): 获取单篇文章详细内容
3. **公众号工具** (`list_wechat_articles_by_account`): 按公众号筛选
4. **热门工具** (`get_trending_wechat_articles`): 发现热门内容

### 5.2 工具工作流设计

#### 工作流1：主题研究
```
Steps:
1. Call search_wechat_articles(keyword="人工智能") -> Get article list
2. Filter and select interesting articles
3. Call get_wechat_article(article_id="...") -> Get detailed content
4. Analyze and summarize findings
```

#### 工作流2：公众号跟踪
```
Steps:
1. Call list_wechat_articles_by_account(account_name="腾讯科技") -> Get latest articles
2. Review article summaries
3. Call get_wechat_article for selected articles -> Read full content
4. Track content trends
```

#### 工作流3：热门发现
```
Steps:
1. Call get_trending_wechat_articles(category="科技") -> Get trending articles
2. Analyze popular topics
3. Use search_wechat_articles for deeper research
```

## 6. 工具规范

### Tool 1: search_wechat_articles

#### 6.1.1 概述
**名称**: `search_wechat_articles`
**目的**: 根据关键词搜索微信文章
**类别**: 搜索

**详细描述**:
此工具允许Agent根据关键词搜索微信文章，返回相关的文章列表。支持分页和结果数量限制，适合快速查找特定主题的内容。工具设计考虑了Agent的上下文限制，提供简洁和详细两种响应格式。

#### 6.1.2 输入模式

```python
class SearchWechatArticlesInput(BaseModel):
    model_config = {"extra": "forbid"}

    keyword: str = Field(
        description="搜索关键词，如：人工智能、区块链、健康养生",
        min_length=1,
        max_length=50,
        examples=["人工智能", "区块链", "健康养生"]
    )

    limit: int = Field(
        default=10,
        ge=1,
        le=50,
        description="返回结果数量 (1-50)"
    )

    page: int = Field(
        default=1,
        ge=1,
        description="页码，用于分页"
    )

    format: Literal["json", "markdown"] = Field(
        default="json",
        description="响应格式：'json' 用于结构化数据，'markdown' 用于可读格式"
    )

    detail: Literal["concise", "detailed"] = Field(
        default="concise",
        description="详细程度：'concise' 用于摘要，'detailed' 用于完整信息"
    )
```

**参数表**:
| 参数 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| keyword | string | 是 | - | 搜索关键词 |
| limit | integer | 否 | 10 | 返回结果数量 (1-50) |
| page | integer | 否 | 1 | 页码 |
| format | enum | 否 | "json" | "json" 或 "markdown" |
| detail | enum | 否 | "concise" | "concise" 或 "detailed" |

#### 6.1.3 输出格式

**简洁 JSON 响应**:
```json
[
  {
    "id": "article_123",
    "title": "文章标题",
    "description": "文章描述",
    "account": "公众号名称",
    "url": "https://mp.weixin.qq.com/..."
  }
]
```

**详细 JSON 响应**:
```json
[
  {
    "id": "article_123",
    "title": "文章标题",
    "description": "文章描述",
    "picurl": "https://...",
    "url": "https://mp.weixin.qq.com/...",
    "account": "公众号名称",
    "publish_time": "2024-01-01 10:00:00"
  }
]
```

**简洁 Markdown 响应**:
```markdown
**文章1**: 文章标题
- 描述: 文章描述
- 公众号: 公众号名称
- 链接: https://mp.weixin.qq.com/...

**文章2**: 文章标题
- 描述: 文章描述
- 公众号: 公众号名称
- 链接: https://mp.weixin.qq.com/...
```

**详细 Markdown 响应**:
```markdown
# 文章标题

**公众号**: 公众号名称
**发布时间**: 2024-01-01 10:00:00

## 描述
文章描述

![封面图片](https://...)

[阅读全文](https://mp.weixin.qq.com/...)
```

#### 6.1.4 工具注解

```python
annotations = {
    "readOnlyHint": True,     # 此工具只读取数据
    "destructiveHint": False, # 此工具不修改数据
    "idempotentHint": True,   # 重复调用具有相同效果
    "openWorldHint": True     # 与外部系统交互
}
```

#### 6.1.5 错误处理

**可能的错误**:
| 错误 | 原因 | Agent 操作 |
|------|------|------------|
| 网络超时 | 页面加载缓慢 | 等待后重试 |
| 反爬虫检测 | 被微信识别为爬虫 | 调整请求间隔和用户代理 |
| 页面结构变化 | 微信页面更新 | 更新解析逻辑 |
| 网络错误 | 连接超时 | 检查网络连接后重试 |

**错误消息示例**:
```
错误: 页面加载超时

建议:
1. 检查网络连接
2. 增加等待时间
3. 尝试使用不同的用户代理
4. 降低请求频率
```

#### 6.1.6 使用指南

**何时使用**:
- 搜索特定主题的微信文章
- 获取相关参考资料
- 研究行业趋势

**何时不使用**:
- 需要实时数据（使用其他工具）
- 需要深度分析（结合其他分析工具）

**示例工作流**:
```
工作流: 研究人工智能趋势
1. 调用 search_wechat_articles(keyword="人工智能", limit=20)
2. 从结果中选择相关文章
3. 使用 get_wechat_article 获取详细内容
4. 分析文章内容和观点
```

---

### Tool 2: get_wechat_article

#### 6.2.1 概述
**名称**: `get_wechat_article`
**目的**: 获取单篇微信文章的详细内容
**类别**: 内容

**详细描述**:
此工具允许Agent获取单篇微信文章的详细内容，包括文章正文、作者信息、发布时间等。支持不同的响应格式，适合深入阅读和分析单篇文章。

#### 6.2.2 输入模式

```python
class GetWechatArticleInput(BaseModel):
    model_config = {"extra": "forbid"}

    article_id: str = Field(
        description="文章ID，从搜索工具获取",
        min_length=1,
        examples=["article_123", "wx_456"]
    )

    include_content: bool = Field(
        default=True,
        description="是否包含文章正文内容"
    )

    format: Literal["json", "markdown"] = Field(
        default="markdown",
        description="响应格式：'json' 用于结构化数据，'markdown' 用于可读格式"
    )
```

#### 6.2.3 输出格式

**JSON 响应**:
```json
{
  "id": "article_123",
  "title": "文章标题",
  "author": "作者",
  "publish_time": "2024-01-01 10:00:00",
  "content": "文章正文内容...",
  "account": "公众号名称",
  "url": "https://mp.weixin.qq.com/..."
}
```

**Markdown 响应**:
```markdown
# 文章标题

**作者**: 作者名称
**发布时间**: 2024-01-01 10:00:00
**公众号**: 公众号名称

## 正文

文章正文内容...

[原文链接](https://mp.weixin.qq.com/...)
```

#### 6.2.4 工具注解

```python
annotations = {
    "readOnlyHint": True,
    "destructiveHint": False,
    "idempotentHint": True,
    "openWorldHint": True
}
```

---

### Tool 3: list_wechat_articles_by_account

#### 6.3.1 概述
**名称**: `list_wechat_articles_by_account`
**目的**: 按公众号获取文章列表
**类别**: 搜索

#### 6.3.2 输入模式

```python
class ListWechatArticlesByAccountInput(BaseModel):
    model_config = {"extra": "forbid"}

    account_name: str = Field(
        description="公众号名称",
        min_length=1,
        examples=["腾讯科技", "人民日报", "央视新闻"]
    )

    limit: int = Field(
        default=10,
        ge=1,
        le=50,
        description="返回结果数量 (1-50)"
    )

    time_range: Optional[str] = Field(
        default=None,
        description="时间范围筛选，如：7d, 30d, 1y"
    )
```

---

### Tool 4: get_trending_wechat_articles

#### 6.4.1 概述
**名称**: `get_trending_wechat_articles`
**目的**: 获取热门微信文章
**类别**: 分析

#### 6.4.2 输入模式

```python
class GetTrendingWechatArticlesInput(BaseModel):
    model_config = {"extra": "forbid"}

    category: Optional[str] = Field(
        default=None,
        description="文章分类，如：科技、财经、健康、娱乐"
    )

    limit: int = Field(
        default=10,
        ge=1,
        le=50,
        description="返回结果数量 (1-50)"
    )
```

## 7. 共享基础设施设计

### 7.1 API 客户端

```python
class WechatCrawler:
    """微信文章爬虫客户端"""

    def __init__(self, headless: bool = True, timeout: int = 30000):
        self.headless = headless
        self.timeout = timeout
        self.browser = None
        self.context = None

    async def search_articles(
        self,
        keyword: str,
        limit: int = 10,
        page: int = 1
    ) -> List[WechatArticle]:
        """搜索微信文章"""
        pass

    async def get_article_details(self, article_url: str) -> WechatArticle:
        """获取文章详情"""
        pass
```

### 7.2 响应格式化器

```python
class ResponseFormatter:
    """格式化响应"""

    @staticmethod
    def format_articles_json(
        articles: List[WechatArticle],
        detail: Literal["concise", "detailed"]
    ) -> str:
        """格式化文章列表为JSON"""
        pass

    @staticmethod
    def format_articles_markdown(
        articles: List[WechatArticle],
        detail: Literal["concise", "detailed"]
    ) -> str:
        """格式化文章列表为Markdown"""
        pass
```

### 7.3 错误处理器

```python
class MCPError(Exception):
    """MCP错误基类"""

    def __init__(self, message: str, suggestion: str = None):
        self.message = message
        self.suggestion = suggestion

def handle_api_error(error: Exception) -> MCPError:
    """转换API错误为可操作的MCP错误"""
    pass
```

## 8. 非功能需求

### 8.1 性能

**响应时间目标**:
- 简单搜索: < 3秒
- 文章详情获取: < 5秒
- 批量操作: < 10秒

**字符限制**:
- 最大响应大小: 25,000 tokens
- 截断策略: 保留重要数据，添加截断通知

### 8.2 可靠性

**错误恢复**:
- 瞬时错误自动重试（3次尝试）
- 指数退避: 1s, 2s, 4s
- 尽可能优雅降级

**超时处理**:
- 请求超时: 30秒
- 整体操作超时: 2分钟

### 8.3 可扩展性

**并发请求**:
- 支持多个同时工具调用
- 速率限制时请求排队
- 连接池提高效率

## 9. 安全考虑

### 9.1 认证安全

**令牌存储**:
- 永不硬编码令牌
- 使用环境变量
- 支持安全令牌提供者

**令牌验证**:
- 启动时验证令牌格式
- 检查令牌权限
- 为无效令牌提供清晰的错误消息

### 9.2 输入验证

**所有输入必须验证**:
- 使用 Pydantic/Zod 模式
- 强制执行最小/最大长度
- 验证格式（URL、邮箱等）
- 防止注入攻击

### 9.3 数据隐私

**敏感数据处理**:
- 不记录令牌或凭据
- 清理错误消息
- 遵循服务的数据保留策略

## 10. 测试策略

### 10.1 单元测试

**测试覆盖**:
- 所有API客户端函数
- 所有格式化器
- 所有错误处理器
- 每个工具的输入验证

**模拟API响应**:
- 成功场景
- 错误场景
- 边界情况

### 10.2 集成测试

**测试场景**:
- 端到端工具执行
- 多工具工作流
- 错误处理流程
- 速率限制处理

## 11. 评估场景

### 11.1 评估要求

每个评估问题必须：
- **独立**: 不依赖其他问题
- **只读**: 仅非破坏性操作
- **复杂**: 需要多次工具调用（3-5+）
- **真实**: 基于真实用例
- **可验证**: 单一明确答案
- **稳定**: 答案不会随时间变化

### 11.2 示例评估

#### 评估 1
**问题**: 研究人工智能在医疗领域的应用，找到3篇相关微信文章并总结主要观点
**预期答案**: 包含3篇文章标题、主要观点和总结的分析报告
**工具序列**:
1. 调用 `search_wechat_articles(keyword="人工智能 医疗", limit=20)` 找到相关文章
2. 从结果中选择3篇最相关的文章
3. 调用 `get_wechat_article(article_id="...")` 获取每篇文章的详细内容
4. 分析内容并总结主要观点

#### 评估 2
**问题**: 跟踪"人民日报"公众号最近发布的5篇文章，分析其主要内容主题
**预期答案**: 5篇文章的标题列表和主题分析
**工具序列**:
1. 调用 `list_wechat_articles_by_account(account_name="人民日报", limit=5)`
2. 获取文章基本信息
3. 调用 `get_wechat_article` 获取部分文章的详细内容
4. 分析文章主题和内容倾向

#### 评估 3
**问题**: 查找当前热门的科技类微信文章，分析热门话题趋势
**预期答案**: 热门科技文章列表和趋势分析
**工具序列**:
1. 调用 `get_trending_wechat_articles(category="科技", limit=15)`
2. 分析文章标题和描述中的关键词
3. 使用 `search_wechat_articles` 对热门话题进行深入搜索
4. 总结当前科技热点趋势

#### 评估 4
**问题**: 比较"腾讯科技"和"阿里技术"两个公众号在人工智能主题上的内容差异
**预期答案**: 两个公众号在AI主题上的内容对比分析
**工具序列**:
1. 调用 `list_wechat_articles_by_account(account_name="腾讯科技", limit=10)`
2. 调用 `list_wechat_articles_by_account(account_name="阿里技术", limit=10)`
3. 使用 `search_wechat_articles(keyword="人工智能")` 在两个公众号中搜索相关文章
4. 分析文章内容和观点差异

#### 评估 5
**问题**: 研究区块链技术在金融领域的应用案例
**预期答案**: 区块链金融应用案例的详细分析
**工具序列**:
1. 调用 `search_wechat_articles(keyword="区块链 金融", limit=20)`
2. 筛选出案例相关的文章
3. 调用 `get_wechat_article` 获取详细案例内容
4. 总结应用场景和效果

## 12. 部署配置

### 12.1 传输协议

**选择**: STDIO

**配置示例** (Claude Desktop):
```json
{
  "mcpServers": {
    "wechat-articles": {
      "command": "python",
      "args": ["main.py"],
      "env": {
        "PLAYWRIGHT_BROWSERS_PATH": "0"
      }
    }
  }
}
```

### 12.2 环境变量

- `PLAYWRIGHT_BROWSERS_PATH`: Playwright 浏览器路径（可选）
- `WEIXIN_CRAWLER_TIMEOUT`: 爬虫超时时间（可选，默认：30000毫秒）
- `WEIXIN_CRAWLER_RETRY_COUNT`: 重试次数（可选，默认：3）
- `WEIXIN_CRAWLER_HEADLESS`: 是否无头模式（可选，默认：true）

### 12.3 依赖列表

- Python 3.8+
- Playwright
- BeautifulSoup4
- requests
- asyncio

## 13. 实施计划

### 13.1 阶段1: 基础架构（第1周）
- [ ] 设置项目结构
- [ ] 实现API客户端
- [ ] 实现错误处理
- [ ] 实现响应格式化器
- [ ] 编写基础设施单元测试

### 13.2 阶段2: 核心工具（第2-3周）
- [ ] 实现 Tool 1: search_wechat_articles
- [ ] 实现 Tool 2: get_wechat_article
- [ ] 实现 Tool 3: list_wechat_articles_by_account
- [ ] 编写每个工具的测试
- [ ] 创建工具文档

### 13.3 阶段3: 高级工具（第4周）
- [ ] 实现 Tool 4: get_trending_wechat_articles
- [ ] 优化性能
- [ ] 添加缓存

### 13.4 阶段4: 测试和文档（第5周）
- [ ] 创建评估场景
- [ ] 运行评估并迭代
- [ ] 编写README和使用指南
- [ ] 创建示例工作流
- [ ] 最终审查和优化

## 14. 参考资料

### 14.1 技术文档
- Playwright 官方文档: https://playwright.dev/python/
- BeautifulSoup 文档: https://www.crummy.com/software/BeautifulSoup/bs4/doc/
- 微信公众平台: https://mp.weixin.qq.com/

### 14.2 MCP 文档
- MCP 协议: https://modelcontextprotocol.io/llms-full.txt
- Python SDK: https://github.com/modelcontextprotocol/python-sdk

### 14.3 相关资源
- 网页爬虫最佳实践
- 反爬虫规避策略
- Python异步编程指南

## 15. 修订历史

| 版本 | 日期 | 作者 | 变更 |
|------|------|------|------|
| 0.1 | 2024-12-19 | MCP架构师 | 初始草案 |
| 1.0 | 2024-12-19 | MCP架构师 | 完成规范 |

---

**注意**: 此规范是动态文档，应在实施过程中根据新发现的需求进行更新。