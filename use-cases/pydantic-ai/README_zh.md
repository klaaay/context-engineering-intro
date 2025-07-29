# Pydantic AI 上下文工程模板

使用上下文工程最佳实践、工具集成、结构化输出和全面测试模式构建生产级AI代理的综合模板。

## 🚀 快速开始 - 复制模板

**2分钟内开始：**

```bash
# 克隆上下文工程仓库
git clone https://github.com/coleam00/Context-Engineering-Intro.git
cd Context-Engineering-Intro/use-cases/pydantic-ai

# 1. 将此模板复制到您的新项目
python copy_template.py /path/to/my-agent-project

# 2. 导航到您的项目
cd /path/to/my-agent-project

# 3. 使用PRP工作流开始构建
# 使用您想要创建的代理填写PRPs/INITIAL.md

# 4. 根据您的详细要求生成PRP（生成后验证PRP！）
/generate-pydantic-ai-prp PRPs/INITIAL.md

# 5. 执行PRP以创建您的Pydantic AI代理
/execute-pydantic-ai-prp PRPs/generated_prp.md
```

如果您不使用Claude Code，您可以简单地告诉您的AI编码助手在.claude/commands中使用generate-pydantic-ai-prp和execute-pydantic-ai-prp斜杠命令作为提示。

## 📖 这是什么模板？

此模板提供了使用经过验证的上下文工程工作流构建复杂Pydantic AI代理所需的一切。它结合了：

- **Pydantic AI最佳实践**：具有工具、结构化输出和依赖注入的类型安全代理
- **上下文工程工作流**：经过验证的PRP（产品需求提示）方法论
- **工作示例**：您可以学习和扩展的完整代理实施

## 🎯 PRP框架工作流

此模板使用3步上下文工程工作流来构建AI代理：

### 1. **定义需求** (`PRPs/INITIAL.md`)
首先明确定义您的代理需要做什么：
```markdown
# 客户支持代理 - 初始需求

## 概述
构建一个智能客户支持代理，可以处理查询、
访问客户数据并适当升级问题。

## 核心需求
- 具有上下文和记忆的多轮对话
- 客户认证和账户访问
- 账户余额和交易查询
- 付款处理和退款处理
...
```

### 2. **生成实施计划** 
```bash
/generate-pydantic-ai-prp PRPs/INITIAL.md
```
这会创建一个全面的"产品需求提示"文档，包括：
- Pydantic AI技术研究和最佳实践
- 具有工具和依赖项的代理架构设计
- 带有验证循环的实施路线图
- 安全模式和生产考虑

### 3. **执行实施**
```bash
/execute-pydantic-ai-prp PRPs/your_agent.md
```
这基于PRP实施完整的代理，包括：
- 具有正确模型提供商配置的代理创建
- 具有错误处理和验证的工具集成
- 具有Pydantic验证的结构化输出模型
- 使用TestModel和FunctionModel的全面测试

## 📂 模板结构

```
pydantic-ai/
├── CLAUDE.md                           # Pydantic AI全局开发规则
├── copy_template.py                    # 模板部署脚本
├── .claude/commands/
│   ├── generate-pydantic-ai-prp.md     # 代理的PRP生成
│   └── execute-pydantic-ai-prp.md      # 代理的PRP执行
├── PRPs/
│   ├── templates/
│   │   └── prp_pydantic_ai_base.md     # 代理的基础PRP模板
│   └── INITIAL.md                      # 代理需求的示例
├── examples/
│   ├── basic_chat_agent/               # 简单的对话代理
│   │   ├── agent.py                    # 具有记忆和上下文的代理
│   │   └── README.md                   # 使用指南
│   ├── tool_enabled_agent/             # 具有外部工具的代理
│   │   ├── agent.py                    # 网络搜索+计算器工具
│   │   └── requirements.txt            # 依赖项
│   └── testing_examples/               # 全面测试模式
│       ├── test_agent_patterns.py      # TestModel、FunctionModel示例
│       └── pytest.ini                  # 测试配置
└── README.md                           # 此文件
```

## 🤖 包含的代理示例

### 1. 主代理参考 (`examples/main_agent_reference/`)
**规范参考实施**显示正确的Pydantic AI模式：
- 基于环境的配置，带有`settings.py`和`providers.py`
- 邮件和研究代理之间的关注点清晰分离
- 适当的文件结构以分离提示、工具、代理和Pydantic模型
- 与外部API（Gmail、Brave搜索）的工具集成

**关键文件：**
- `settings.py`：使用pydantic-settings的环境配置
- `providers.py`：带有`get_llm_model()`的模型提供商抽象
- `research_agent.py`：具有网络搜索和电子邮件集成的多工具代理
- `email_agent.py`：用于Gmail草稿创建的专用代理

### 2. 基本聊天代理 (`examples/basic_chat_agent/`)
显示核心模式的简单对话代理：
- **基于环境的模型配置**（遵循main_agent_reference）
- **默认字符串输出**（除非需要，没有`result_type`）
- 系统提示（静态和动态）
- 具有依赖注入的对话记忆

**关键功能：**
- 简单字符串响应（不是结构化输出）
- 基于设置的配置模式
- 对话上下文跟踪
- 简洁、最小的实施

### 3. 工具启用代理 (`examples/tool_enabled_agent/`)
具有工具集成功能的代理：
- **基于环境的配置**（遵循main_agent_reference）
- **默认字符串输出**（不需要不必要的结构）
- 网络搜索和计算工具
- 错误处理和重试机制

**关键功能：**
- `@agent.tool`装饰器模式
- 依赖注入的RunContext
- 工具错误处理和恢复
- 来自工具的简单字符串响应

### 4. 结构化输出代理 (`examples/structured_output_agent/`)
**新**：显示何时使用`result_type`进行数据验证：
- **基于环境的配置**（遵循main_agent_reference）
- **具有Pydantic验证的结构化输出**（在特别需要时）
- 具有统计工具的数据分析
- 专业报告生成

**关键功能：**
- 演示`result_type`的正确使用
- Pydantic验证业务报告
- 具有数值统计的数据分析工具
- 关于何时使用结构化与字符串输出的清晰文档

### 5. 测试示例 (`examples/testing_examples/`)
Pydantic AI代理的全面测试模式：
- 用于快速开发验证的TestModel
- 用于自定义行为测试的FunctionModel
- Agent.override()用于测试隔离
- Pytest夹具和异步测试

**关键功能：**
- 无需API成本的单元测试
- 模拟依赖注入
- 工具验证和错误场景测试
- 集成测试模式

## 📚 额外资源

- **官方Pydantic AI文档**：https://ai.pydantic.dev/
- **上下文工程方法论**：查看主仓库README

## 🆘 支持和贡献

- **问题**：报告模板或示例的问题
- **改进**：贡献额外的示例或模式
- **问题**：询问Pydantic AI集成或上下文工程

此模板是更大的上下文工程框架的一部分。查看主仓库以获取更多上下文工程模板和方法论。

---

**准备构建生产级AI代理？** 从`python copy_template.py my-agent-project`开始，遵循PRP工作流！🚀