# PydanticAI上下文工程 - AI代理开发的全局规则

此文件包含适用于所有PydanticAI代理开发工作的全局规则和原则。这些规则专门用于构建具有工具、内存和结构化输出的生产级AI代理。

## 🔄 PydanticAI核心原则

**重要：这些原则适用于所有PydanticAI代理开发：**

### 代理开发工作流
- **始终从INITIAL.md开始** - 在生成PRP之前定义代理要求
- **使用PRP模式**：INITIAL.md → `/generate-pydantic-ai-prp INITIAL.md` → `/execute-pydantic-ai-prp PRPs/filename.md`
- **遵循验证循环** - 每个PRP必须包含使用TestModel/FunctionModel的代理测试
- **上下文为王** - 包含所有必要的PydanticAI模式、示例和文档

### AI代理的研究方法论
- **广泛进行网络搜索** - 始终研究PydanticAI模式和最佳实践
- **研究官方文档** - ai.pydantic.dev是权威来源
- **模式提取** - 识别可重用的代理架构和工具模式
- **陷阱文档** - 记录异步模式、模型限制和上下文管理问题

## 📚 项目认知与上下文

- **使用虚拟环境** 运行所有代码和测试。如果需要时代码库中还没有，请创建一个
- **使用一致的PydanticAI命名约定** 和代理结构模式
- **遵循已建立的代理目录组织** 模式（agent.py、tools.py、models.py）
- **广泛利用PydanticAI示例** - 在创建新代理之前研究现有模式

## 🧱 代理结构与模块化

- **绝不创建超过500行的文件** - 接近限制时拆分为模块
- **将代理代码组织成清晰分离的模块**，按职责分组：
  - `agent.py` - 主要代理定义和执行逻辑
  - `tools.py` - 代理使用的工具函数
  - `models.py` - Pydantic输出模型和依赖类
  - `dependencies.py` - 上下文依赖和外部服务集成
- **使用清晰、一致的导入** - 适当从pydantic_ai包导入
- **使用python-dotenv和load_dotenv()** 处理环境变量 - 遵循examples/main_agent_reference/settings.py模式
- **绝不硬编码敏感信息** - 始终使用.env文件处理API密钥和配置

## 🤖 PydanticAI开发标准

### 代理创建模式
- **使用与模型无关的设计** - 支持多个提供商（OpenAI、Anthropic、Gemini）
- **实现依赖注入** - 对外部服务和上下文使用deps_type
- **定义结构化输出** - 使用Pydantic模型进行结果验证
- **包含全面的系统提示** - 静态和动态指令

### 工具集成标准
- **使用@agent.tool装饰器** 用于具有RunContext[DepsType]的上下文感知工具
- **使用@agent.tool_plain装饰器** 用于没有上下文依赖的简单工具
- **实现适当的参数验证** - 对工具参数使用Pydantic模型
- **优雅地处理工具错误** - 实现重试机制和错误恢复

### 使用python-dotenv的环境变量配置
```python
# 使用python-dotenv和pydantic-settings进行适当的配置管理
from pydantic_settings import BaseSettings
from pydantic import Field, ConfigDict
from dotenv import load_dotenv
from pydantic_ai.providers.openai import OpenAIProvider
from pydantic_ai.models.openai import OpenAIModel

class Settings(BaseSettings):
    """具有环境变量支持的应用设置。"""
    
    model_config = ConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
        extra="ignore"
    )
    
    # LLM配置
    llm_provider: str = Field(default="openai", description="LLM提供商")
    llm_api_key: str = Field(..., description="LLM提供商的API密钥")
    llm_model: str = Field(default="gpt-4", description="要使用的模型名称")
    llm_base_url: str = Field(
        default="https://api.openai.com/v1", 
        description="LLM API的基础URL"
    )

def load_settings() -> Settings:
    """使用适当的错误处理和环境加载加载设置。"""
    # 从.env文件加载环境变量
    load_dotenv()
    
    try:
        return Settings()
    except Exception as e:
        error_msg = f"加载设置失败：{e}"
        if "llm_api_key" in str(e).lower():
            error_msg += "\n确保在您的.env文件中设置LLM_API_KEY"
        raise ValueError(error_msg) from e

def get_llm_model():
    """使用适当的环境加载获取配置的LLM模型。"""
    settings = load_settings()
    provider = OpenAIProvider(
        base_url=settings.llm_base_url, 
        api_key=settings.llm_api_key
    )
    return OpenAIModel(settings.llm_model, provider=provider)
```

### AI代理的测试标准
- **开发使用TestModel** - 无需API调用的快速验证
- **自定义行为使用FunctionModel** - 在测试中控制代理响应
- **测试使用Agent.override()** - 在测试上下文中替换模型
- **测试同步和异步模式** - 确保与不同执行模式的兼容性
- **测试工具验证** - 验证工具参数模式和错误处理

## ✅ AI开发的任务管理

- **将代理开发分解为清晰的步骤** 具有特定的完成标准
- **完成后立即标记任务完成** 完成代理实现后
- **实时更新任务状态** 随着代理开发的进展
- **在标记实现任务完成之前测试代理行为**

## 📎 PydanticAI编码标准

### 代理架构
```python
# 遵循main_agent_reference模式 - 除非需要结构化输出，否则不需要result_type
from pydantic_ai import Agent, RunContext
from dataclasses import dataclass
from .settings import load_settings

@dataclass
class AgentDependencies:
    """代理执行的依赖项"""
    api_key: str
    session_id: str = None

# 使用适当的dotenv处理加载设置
settings = load_settings()

# 具有字符串输出的简单代理（默认）
agent = Agent(
    get_llm_model(),  # 内部使用load_settings()
    deps_type=AgentDependencies,
    system_prompt="你是一个有用的助手..."
)

@agent.tool
async def example_tool(
    ctx: RunContext[AgentDependencies], 
    query: str
) -> str:
    """具有适当上下文访问的工具"""
    return await external_api_call(ctx.deps.api_key, query)
```

### 安全最佳实践
- **API密钥管理** - 使用python-dotenv和.env文件，绝不将密钥提交到版本控制
- **环境变量加载** - 始终使用load_dotenv()遵循examples/main_agent_reference/settings.py
- **输入验证** - 对所有工具参数使用Pydantic模型
- **速率限制** - 对外部API实施适当的请求限制
- **提示注入预防** - 验证和清理用户输入
- **错误处理** - 绝不在错误消息中暴露敏感信息

### 常见PydanticAI陷阱
- **异步/同步混合问题** - 在整个过程中保持异步/等待模式的一致性
- **模型令牌限制** - 不同模型具有不同的上下文限制，相应地进行规划
- **依赖注入复杂性** - 保持依赖关系图简单且类型良好
- **工具错误处理失败** - 始终实施适当的重试和回退机制
- **上下文状态管理** - 尽可能设计无状态工具以确保可靠性

## 🔍 AI代理的研究标准

- **使用Archon MCP服务器** - 通过RAG利用可用的PydanticAI文档
- **研究官方示例** - ai.pydantic.dev/examples具有工作实现
- **研究模型能力** - 了解提供商特定的功能和限制
- **记录集成模式** - 包括外部服务集成示例

## 🎯 AI代理的实现标准

- **严格遵循PRP工作流** - 不要跳过代理验证步骤
- **始终首先使用TestModel进行测试** - 在使用真实模型之前验证代理逻辑
- **使用现有代理模式** 而不是从头开始创建
- **为工具失败和模型错误包含全面的错误处理**
- **在实现实时代理交互时测试流模式**

## 🚫 始终避免的反模式

- ❌ 不要跳过代理测试 - 始终使用TestModel/FunctionModel进行验证
- ❌ 不要硬编码模型字符串 - 使用基于环境的配置，如main_agent_reference
- ❌ 除非特别需要结构化输出，否则不要使用result_type - 默认为字符串
- ❌ 不要忽略异步模式 - PydanticAI具有特定的异步/同步考虑因素
- ❌ 不要创建复杂的依赖关系图 - 保持依赖关系简单且可测试
- ❌ 不要忘记工具错误处理 - 实施适当的重试和优雅降级
- ❌ 不要跳过输入验证 - 对所有外部输入使用Pydantic模型

## 🔧 AI开发的工具使用标准

- **广泛使用网络搜索** 进行PydanticAI研究和文档
- **遵循PydanticAI命令模式** 用于斜杠命令和代理工作流
- **使用代理验证循环** 确保每个开发步骤的质量
- **使用多个模型提供商进行测试** 以确保代理兼容性

## 🧪 AI代理的测试与可靠性

- **始终创建全面的代理测试** 用于工具、输出和错误处理
- **使用TestModel测试代理行为** 在使用真实模型提供商之前
- **包含边界情况测试** 用于工具失败和模型提供商问题
- **测试结构化和非结构化输出** 以确保代理灵活性
- **验证依赖注入** 在测试环境中正确工作

这些全局规则专门适用于PydanticAI代理开发，并确保具有适当错误处理、测试和安全实践的生产就绪AI应用程序。