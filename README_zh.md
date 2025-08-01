# 上下文工程模板

开始使用上下文工程的全面模板 - 上下文工程是为AI编码助手提供必要信息以端到端完成工作的学科。

> **上下文工程比提示工程好10倍，比氛围编码好100倍。**

## 🚀 快速开始

```bash
# 1. 克隆此模板
git clone https://github.com/coleam00/Context-Engineering-Intro.git
cd Context-Engineering-Intro

# 2. 设置项目规则（可选 - 提供模板）
# 编辑 CLAUDE.md 添加您的项目特定指南

# 3. 添加示例（强烈推荐）
# 将相关代码示例放在 examples/ 文件夹中

# 4. 创建您的初始功能请求
# 使用您的功能要求编辑 INITIAL.md

# 5. 生成全面的PRP（产品需求提示）
# 在Claude Code中运行：
/generate-prp INITIAL.md

# 6. 执行PRP以实施您的功能
# 在Claude Code中运行：
/execute-prp PRPs/your-feature-name.md
```

## 📚 目录

- [什么是上下文工程？](#什么是上下文工程)
- [模板结构](#模板结构)
- [逐步指南](#逐步指南)
- [编写有效的INITIAL.md文件](#编写有效的initialmd文件)
- [PRP工作流](#prp工作流)
- [有效使用示例](#有效使用示例)
- [最佳实践](#最佳实践)

## 什么是上下文工程？

上下文工程代表了从传统提示工程的范式转变：

### 提示工程 vs 上下文工程

**提示工程：**
- 专注于巧妙的措辞和特定短语
- 仅限于您如何表达任务
- 就像给别人一张便利贴

**上下文工程：**
- 提供全面上下文的完整系统
- 包括文档、示例、规则、模式和验证
- 就像编写包含所有细节的完整剧本

### 为什么上下文工程很重要

1. **减少AI失败**：大多数代理失败不是模型失败 - 它们是上下文失败
2. **确保一致性**：AI遵循您的项目模式和约定
3. **启用复杂功能**：AI可以通过适当的上下文处理多步骤实施
4. **自我纠正**：验证循环允许AI修复自己的错误

## 模板结构

```
context-engineering-intro/
├── .claude/
│   ├── commands/
│   │   ├── generate-prp.md    # 生成全面的PRP
│   │   └── execute-prp.md   # 执行PRP以实施功能
│   └── settings.local.json    # Claude Code权限
├── PRPs/
│   ├── templates/
│   │   └── prp_base.md       # PRP的基础模板
│   └── EXAMPLE_multi_agent_prp.md  # 完整PRP的示例
├── examples/                  # 您的代码示例（关键！）
├── CLAUDE.md                 # AI助手的全局规则
├── INITIAL.md               # 功能请求的模板
├── INITIAL_EXAMPLE.md       # 功能请求的示例
└── README.md                # 此文件
```

此模板不专注于上下文工程的RAG和工具，因为我很快就会有很多更多内容。;)

## 逐步指南

### 1. 设置全局规则（CLAUDE.md）

`CLAUDE.md`文件包含AI助手将在每次对话中遵循的项目范围规则。模板包括：

- **项目认知**：阅读规划文档，检查任务
- **代码结构**：文件大小限制，模块组织
- **测试要求**：单元测试模式，覆盖期望
- **样式约定**：语言偏好，格式化规则
- **文档标准**：文档字符串格式，注释实践

**您可以按原样使用提供的模板，或为您的项目自定义它。**

### 2. 创建您的初始功能请求

编辑`INITIAL.md`以描述您想要构建的内容：

```markdown
## 功能：
[描述您想要构建的内容 - 具体说明功能和要求]

## 示例：
[列出examples/文件夹中的任何示例文件并解释应如何使用它们]

## 文档：
[包括相关文档、API或MCP服务器资源的链接]

## 其他考虑因素：
[提及任何陷阱、特定要求或AI助手常遗漏的内容]
```

**查看`INITIAL_EXAMPLE.md`获取完整示例。**

### 3. 生成PRP

PRP（产品需求提示）是包括以下内容的全面实施蓝图：

- 完整的上下文和文档
- 带有验证的实施步骤
- 错误处理模式
- 测试要求

它们类似于PRD（产品需求文档），但专门为指导AI编码助手而制作。

在Claude Code中运行：
```bash
/generate-prp INITIAL.md
```

**注意：**斜杠命令是在`.claude/commands/`中定义的自定义命令。您可以查看其实现：
- `.claude/commands/generate-prp.md` - 查看它如何研究和创建PRP
- `.claude/commands/execute-prp.md` - 查看它如何从PRP实施功能

这些命令中的`$ARGUMENTS`变量接收您在命令名称后传递的任何内容（例如`INITIAL.md`或`PRPs/your-feature.md`）。

此命令将：
1. 阅读您的功能请求
2. 分析代码库中的模式
3. 搜索相关文档
4. 在`PRPs/your-feature-name.md`中创建全面的PRP

### 4. 执行PRP

生成后，执行PRP以实施您的功能：

```bash
/execute-prp PRPs/your-feature-name.md
```

AI编码助手将：
1. 从PRP读取所有上下文
2. 创建详细的实施计划
3. 执行每个步骤并进行验证
4. 运行测试并修复任何问题
5. 确保满足所有要求

## 编写有效的INITIAL.md文件

### 关键部分解释

**功能**：具体且全面
- ❌ "构建网络爬虫"
- ✅ "使用BeautifulSoup构建异步网络爬虫，从电子商务网站提取产品数据，处理速率限制，并将结果存储在PostgreSQL中"

**示例**：利用examples/文件夹
- 将相关代码模式放在`examples/`中
- 引用要遵循的特定文件和模式
- 解释应模仿的方面

**文档**：包括所有相关资源
- API文档URL
- 库指南
- MCP服务器文档
- 数据库模式

**其他考虑因素**：捕获重要细节
- 身份验证要求
- 速率限制或配额
- 常见陷阱
- 性能要求

## PRP工作流

### /generate-prp如何工作

该命令遵循以下过程：

1. **研究阶段**
   - 分析代码库中的模式
   - 搜索类似实施
   - 识别要遵循的约定

2. **文档收集**
   - 获取相关API文档
   - 包括库文档
   - 添加陷阱和怪癖

3. **蓝图创建**
   - 创建逐步实施计划
   - 包括验证门
   - 添加测试要求

4. **质量检查**
   - 评分置信度（1-10）
   - 确保包含所有上下文

### /execute-prp如何工作

1. **加载上下文**：阅读整个PRP
2. **计划**：使用TodoWrite创建详细任务列表
3. **执行**：实施每个组件
4. **验证**：运行测试和linting
5. **迭代**：修复发现的任何问题
6. **完成**：确保满足所有要求

查看`PRPs/EXAMPLE_multi_agent_prp.md`了解生成的完整示例。

## 有效使用示例

`examples/`文件夹对成功**至关重要**。当AI编码助手能够看到要遵循的模式时，它们的表现会更好。

### 在示例中包含什么

1. **代码结构模式**
   - 您如何组织模块
   - 导入约定
   - 类/函数模式

2. **测试模式**
   - 测试文件结构
   - 模拟方法
   - 断言样式

3. **集成模式**
   - API客户端实施
   - 数据库连接
   - 身份验证流程

4. **CLI模式**
   - 参数解析
   - 输出格式化
   - 错误处理

### 示例结构

```
examples/
├── README.md           # 解释每个示例演示的内容
├── cli.py             # CLI实施模式
├── agent/             # 代理架构模式
│   ├── agent.py      # 代理创建模式
│   ├── tools.py      # 工具实施模式
│   └── providers.py  # 多提供商模式
└── tests/            # 测试模式
    ├── test_agent.py # 单元测试模式
    └── conftest.py   # Pytest配置
```

## 最佳实践

### 1. 在INITIAL.md中明确
- 不要假设AI知道您的偏好
- 包括具体要求和约束
- 自由引用示例

### 2. 提供全面示例
- 更多示例 = 更好的实施
- 显示做什么和不要做什么
- 包括错误处理模式

### 3. 使用验证门
- PRP包括必须通过测试的命令
- AI将迭代直到所有验证成功
- 这确保第一次就获得工作代码

### 4. 利用文档
- 包括官方API文档
- 添加MCP服务器资源
- 引用特定文档部分

### 5. 自定义CLAUDE.md
- 添加您的约定
- 包括项目特定规则
- 定义编码标准

## 资源

- [Claude Code文档](https://docs.anthropic.com/en/docs/claude-code)
- [上下文工程最佳实践](https://www.philschmid.de/context-engineering)