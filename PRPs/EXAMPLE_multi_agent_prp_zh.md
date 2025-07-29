名称："多代理系统：带有邮件草稿子代理的研究代理"
描述：|

## 目的
构建一个Pydantic AI多代理系统，其中主要研究代理使用Brave搜索API，并具有使用Gmail API的邮件草稿代理作为工具。这演示了代理作为工具模式与外部API集成。

## 核心原则
1. **上下文为王**：包含所有必要的文档、示例和警告
2. **验证循环**：提供AI可以运行和修复的可执行测试/检查
3. **信息密集**：使用代码库中的关键词和模式
4. **渐进成功**：从简单开始，验证，然后增强

---

## 目标
创建一个生产就绪的多代理系统，用户可以通过CLI研究主题，研究代理可以将邮件起草任务委托给邮件草稿代理。系统应支持多个LLM提供商并安全处理API身份验证。

## 原因
- **商业价值**：自动化研究和邮件起草工作流
- **集成**：演示高级Pydantic AI多代理模式
- **解决的问题**：减少基于研究的邮件通信的手动工作

## 内容
基于CLI的应用程序，其中：
- 用户输入研究查询
- 研究代理使用Brave API搜索
- 研究代理可以调用邮件草稿代理创建Gmail草稿
- 结果实时流回给用户

### 成功标准
- [ ] 研究代理成功通过Brave API搜索
- [ ] 邮件代理使用适当的身份验证创建Gmail草稿
- [ ] 研究代理可以将邮件代理作为工具调用
- [ ] CLI提供具有工具可见性的流响应
- [ ] 所有测试通过且代码符合质量标准

## 所有需要的上下文

### 文档与参考资料
```yaml
# 必须阅读 - 在您的上下文窗口中包含这些内容
- url: https://ai.pydantic.dev/agents/
  原因：核心代理创建模式
  
- url: https://ai.pydantic.dev/multi-agent-applications/
  原因：多代理系统模式，特别是代理作为工具
  
- url: https://developers.google.com/gmail/api/guides/sending
  原因：Gmail API身份验证和草稿创建
  
- url: https://api-dashboard.search.brave.com/app/documentation
  原因：Brave搜索API REST端点
  
- file: examples/agent/agent.py
  原因：代理创建、工具注册、依赖项的模式
  
- file: examples/agent/providers.py
  原因：多提供商LLM配置模式
  
- file: examples/cli.py
  原因：具有流响应和工具可见性的CLI结构

- url: https://github.com/googleworkspace/python-samples/blob/main/gmail/snippet/send%20mail/create_draft.py
  原因：官方Gmail草稿创建示例
```

### 当前代码库树
```bash
.
├── examples/
│   ├── agent/
│   │   ├── agent.py
│   │   ├── providers.py
│   │   └── ...
│   └── cli.py
├── PRPs/
│   └── templates/
│       └── prp_base.md
├── INITIAL.md
├── CLAUDE.md
└── requirements.txt
```

### 期望的代码库树与要添加的文件
```bash
.
├── agents/
│   ├── __init__.py               # 包初始化
│   ├── research_agent.py         # 具有Brave搜索的主要代理
│   ├── email_agent.py           # 具有Gmail功能的子代理
│   ├── providers.py             # LLM提供商配置
│   └── models.py                # 用于数据验证的Pydantic模型
├── tools/
│   ├── __init__.py              # 包初始化
│   ├── brave_search.py          # Brave搜索API集成
│   └── gmail_tool.py            # Gmail API集成
├── config/
│   ├── __init__.py              # 包初始化
│   └── settings.py              # 环境和配置管理
├── tests/
│   ├── __init__.py              # 包初始化
│   ├── test_research_agent.py   # 研究代理测试
│   ├── test_email_agent.py      # 邮件代理测试
│   ├── test_brave_search.py     # Brave搜索工具测试
│   ├── test_gmail_tool.py       # Gmail工具测试
│   └── test_cli.py              # CLI测试
├── cli.py                       # CLI接口
├── .env.example                 # 环境变量模板
├── requirements.txt             # 更新的依赖项
├── README.md                    # 全面文档
└── credentials/.gitkeep         # Gmail凭据目录
```

### 已知陷阱和库怪癖
```python
# 关键：Pydantic AI要求全程异步 - 异步上下文中没有同步函数
# 关键：Gmail API首次运行需要OAuth2流程 - 需要credentials.json
# 关键：Brave API有速率限制 - 免费层每月2000次请求
# 关键：代理作为工具模式需要传递ctx.usage进行令牌跟踪
# 关键：Gmail草稿需要具有适当MIME格式的base64编码
# 关键：始终对更清晰的代码使用绝对导入
# 关键：在.env中存储敏感凭据，绝不提交它们
```

## 实施蓝图

### 数据模型和结构

```python
# models.py - 核心数据结构
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime

class ResearchQuery(BaseModel):
    query: str = Field(..., description="要调查的研究主题")
    max_results: int = Field(10, ge=1, le=50)
    include_summary: bool = Field(True)

class BraveSearchResult(BaseModel):
    title: str
    url: str
    description: str
    score: float = Field(0.0, ge=0.0, le=1.0)

class EmailDraft(BaseModel):
    to: List[str] = Field(..., min_items=1)
    subject: str = Field(..., min_length=1)
    body: str = Field(..., min_length=1)
    cc: Optional[List[str]] = None
    bcc: Optional[List[str]] = None

class ResearchEmailRequest(BaseModel):
    research_query: str
    email_context: str = Field(..., description="邮件生成的上下文")
    recipient_email: str
```

### 要完成的任务列表

```yaml
任务1：设置配置和环境
创建config/settings.py：
  - 模式：像examples使用os.getenv一样使用pydantic-settings
  - 加载具有默认值的环境变量
  - 验证必需的API密钥是否存在

创建.env.example：
  - 包括所有必需的环境变量及描述
  - 遵循examples/README.md中的模式

任务2：实施Brave搜索工具
创建tools/brave_search.py：
  - 模式：像examples/agent/tools.py中的异步函数
  使用httpx的简单REST客户端（已在要求中）
  - 优雅地处理速率限制和错误
  - 返回结构化的BraveSearchResult模型

任务3：实施Gmail工具
创建tools/gmail_tool.py：
  - 模式：遵循Gmail快速入门的OAuth2流程
  在credentials/目录中存储token.json
  - 使用适当的MIME编码创建草稿
  - 自动处理身份验证刷新

任务4：创建邮件草稿代理
创建agents/email_agent.py：
  - 模式：遵循examples/agent/agent.py结构
  使用具有deps_type模式的代理
  将gmail_tool注册为@agent.tool
  返回EmailDraft模型

任务5：创建研究代理
创建agents/research_agent.py：
  - 模式：来自Pydantic AI文档的多代理模式
  将brave_search注册为工具
  将email_agent.run()注册为工具
  对依赖注入使用RunContext

任务6：实施CLI接口
```