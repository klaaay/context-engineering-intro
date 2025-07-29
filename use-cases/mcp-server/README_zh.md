# MCP服务器构建器 - 上下文工程用例

此用例演示如何使用**上下文工程**和**PRP（产品需求提示）流程**构建生产就绪的模型上下文协议（MCP）服务器。它提供了一个经过验证的模板和工作流，用于创建带有GitHub OAuth身份验证、数据库集成和Cloudflare Workers部署的MCP服务器。

> PRP是PRD + 策划的代码库智能 + 代理/运行手册 - AI在第一次尝试中合理交付生产就绪代码所需的最小可行数据包。

## 🚀 快速开始

### 先决条件

- 已安装Node.js和npm
- Cloudflare账户（免费层即可）
- 用于OAuth的GitHub账户
- PostgreSQL数据库（本地或托管）

### 步骤1：设置您的项目

```bash
# 克隆上下文工程仓库
git clone https://github.com/coleam00/Context-Engineering-Intro.git
cd Context-Engineering-Intro/use-cases/mcp-server

# 将模板复制到您的新项目目录
python copy_template.py my-mcp-server-project

# 导航到您的新项目
cd my-mcp-server-project

# 安装依赖项
npm install

# 全局安装Wrangler CLI
npm install -g wrangler

# 使用Cloudflare进行身份验证
wrangler login
```

**copy_template.py的作用：**
- 复制所有模板文件，构建工件除外（尊重.gitignore）
- 将README.md重命名为README_TEMPLATE.md（这样您可以创建自己的README）
- 包括所有源代码、示例、测试和配置文件
- 保留完整的上下文工程设置

## 🎯 您将学到什么

此用例教您如何：

- **使用PRP流程**系统地构建复杂的MCP服务器
- **利用专门的上下文工程**进行MCP开发
- **遵循经过验证的模式**来自生产就绪的MCP服务器模板
- **实施安全身份验证**与GitHub OAuth和基于角色的访问
- **部署到Cloudflare Workers**并进行监控和错误处理

## 📋 如何工作 - MCP服务器的PRP流程

> **步骤1是上面的快速开始设置** - 克隆仓库、复制模板、安装依赖项、设置Wrangler

### 步骤2：定义您的MCP服务器

编辑`PRPs/INITIAL.md`以描述您的特定MCP服务器要求：

```markdown
## 功能：
我们想要创建一个天气MCP服务器，提供实时天气数据
并具有缓存和速率限制。

## 附加功能：
- 与OpenWeatherMap API集成
- 用于性能的Redis缓存
- 按用户的速率限制
- 历史天气数据访问
- 位置搜索和自动完成

## 其他考虑因素：
- 外部服务的API密钥管理
- API失败的正确错误处理
- 位置查询的坐标验证
```

### 步骤3：生成您的PRP

使用专门的MCP PRP命令创建全面的实施计划：

```bash
/prp-mcp-create INITIAL.md
```

**此操作的作用：**
- 读取您的功能请求
- 研究现有MCP代码库模式
- 研究身份验证和数据库集成模式
- 在`PRPs/your-server-name.md`中创建全面的PRP
- 包括所有上下文、验证循环和逐步任务

> PRP生成后验证一切很重要！使用PRP框架，您应该成为过程的一部分，以确保所有上下文的质量！执行的好坏取决于您的PRP。使用/prp-mcp-create作为坚实的起点。

### 步骤4：执行您的PRP

使用专门的MCP执行命令构建您的服务器：

```bash
/prp-mcp-execute PRPs/your-server-name.md
```

**此操作的作用：**
- 加载完整的PRP及所有上下文
- 使用TodoWrite创建详细的实施计划
- 遵循经过验证的模式实施每个组件
- 运行全面验证（TypeScript、测试、部署）
- 确保您的MCP服务器端到端工作

### 步骤5：配置环境

```bash
# 创建环境文件
cp .dev.vars.example .dev.vars

# 使用您的凭据编辑.dev.vars
# - GitHub OAuth应用凭据
# - 数据库连接字符串
# - Cookie加密密钥
```

### 步骤6：测试和部署

```bash
# 本地测试
wrangler dev --config <your wrangler config (.jsonc)>

# 使用MCP Inspector测试
npx @modelcontextprotocol/inspector@latest
# 连接到：http://localhost:8792/mcp

# 部署到生产环境
wrangler deploy
```

## 🏗️ MCP特定的上下文工程

此用例包括专门为MCP服务器开发设计的专门上下文工程组件：

### 专门的斜杠命令

位于`.claude/commands/`中：

- **`/prp-mcp-create`** - 专门为MCP服务器生成PRP
- **`/prp-mcp-execute`** - 使用全面验证执行MCP PRP

这些是根`.claude/commands/`中通用命令的专门版本，但针对MCP开发模式进行了定制。

### 专门的PRP模板

模板`PRPs/templates/prp_mcp_base.md`包括：

- **MCP特定模式**用于工具注册和身份验证
- **Cloudflare Workers配置**用于部署
- **GitHub OAuth集成**模式
- **数据库安全**和SQL注入保护
- **全面验证循环**从TypeScript到生产

### AI文档

`PRPs/ai_docs/`文件夹包含：

- **`mcp_patterns.md`** - 核心MCP开发模式和安全实践
- **`claude_api_usage.md`** - 如何与Anthropic的API集成以获得LLM驱动的功能

## 🔧 模板架构

此模板提供了一个完整的生产就绪MCP服务器，包括：

### 核心组件

```
src/
├── index.ts                 # 主要经过身份验证的MCP服务器
├── index_sentry.ts         # 带有Sentry监控的版本
├── simple-math.ts          # 基本MCP示例（无身份验证）
├── github-handler.ts       # 完整的GitHub OAuth实施
├── database.ts             # 具有安全模式的PostgreSQL
├── utils.ts                # OAuth帮助程序和实用程序
├── workers-oauth-utils.ts  # HMAC签名的cookie系统
└── tools/                  # 模块化工具注册系统
    └── register-tools.ts   # 中央工具注册表
```

### 示例工具

`examples/`文件夹显示如何创建MCP工具：

- **`database-tools.ts`** - 具有正确模式的数据库工具示例
- **`database-tools-sentry.ts`** - 带有Sentry监控的相同工具

### 关键功能

- **🔐 GitHub OAuth** - 具有基于角色的访问的完整身份验证流程
- **🗄️ 数据库集成** - 具有连接池和安全的PostgreSQL
- **🛠️ 模块化工具** - 具有中央注册的清晰关注点分离
- **☁️ Cloudflare Workers** - 具有Durable Objects的全球边缘部署
- **📊 监控** - 用于生产的可选Sentry集成
- **🧪 测试** - 从TypeScript到部署的全面验证

## 🔍 需要理解的关键文件

要完全理解此用例，请检查这些文件：

### 上下文工程组件

- **`PRPs/templates/prp_mcp_base.md`** - 专门的MCP PRP模板
- **`.claude/commands/prp-mcp-create.md`** - MCP特定的PRP生成
- **`.claude/commands/prp-mcp-execute.md`** - MCP特定的执行

### 实施模式

- **`src/index.ts`** - 具有身份验证的完整MCP服务器
- **`examples/database-tools.ts`** - 工具创建和注册模式
- **`src/tools/register-tools.ts`** - 模块化工具注册系统

### 配置和部署

- **`wrangler.jsonc`** - Cloudflare Workers配置
- **`.dev.vars.example`** - 环境变量模板
- **`CLAUDE.md`** - 实施指南和模式

## 📈 成功指标

当您成功使用此流程时，您将实现：

- **快速实施** - 以最少的迭代快速拥有MCP服务器
- **生产就绪** - 安全身份验证、监控和错误处理
- **可扩展架构** - 清晰的关注点分离和模块化设计
- **全面测试** - 从TypeScript到生产部署的验证

## 🤝 贡献

此用例展示了复杂软件开发中上下文工程的力量。为了改进它：

1. **添加新的MCP服务器示例**以显示不同模式
2. **增强PRP模板**以更全面的上下文
3. **改进验证循环**以获得更好的错误检测
4. **记录边缘情况**和常见陷阱

目标是使MCP服务器开发通过全面的上下文工程变得可预测和成功。

---

**准备构建您的MCP服务器？** 遵循上面的完整流程：使用复制模板设置您的项目，配置您的环境，在`PRPs/INITIAL.md`中定义您的要求，然后生成并执行您的PRP以构建您的生产就绪MCP服务器。