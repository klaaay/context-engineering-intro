# 具有GitHub OAuth的MCP服务器 - 实施指南

本指南提供了使用Node.js、TypeScript和Cloudflare Workers构建具有GitHub OAuth身份验证的MCP（模型上下文协议）服务器的实施模式和标准。有关要构建什么，请参阅PRP（产品需求提示）文档。

## 核心原则

**重要：您必须在所有代码更改和PRP生成中遵循这些原则：**

### KISS（保持简单，愚蠢）

- 简单性应该是设计中的关键目标
- 只要可能，选择简单的解决方案而不是复杂的解决方案
- 简单的解决方案更容易理解、维护和调试

### YAGNI（你不会需要它）

- 避免基于推测构建功能
- 仅在需要时实施功能，而不是在您预期它们将来可能有用时

### 开放/封闭原则

- 软件实体应该对扩展开放但对修改封闭
- 设计系统，使新功能可以通过对现有代码的最小更改来添加

## 包管理与工具

**关键：此项目将npm用于Node.js包管理，将Wrangler CLI用于Cloudflare Workers开发。**

### 基本npm命令

```bash
# 从package.json安装依赖项
npm install

# 添加依赖项
npm install package-name

# 添加开发依赖项
npm install --save-dev package-name

# 删除包
npm uninstall package-name

# 更新依赖项
npm update

# 运行package.json中定义的脚本
npm run dev
npm run deploy
npm run type-check
```

### 基本Wrangler CLI命令

**关键：对所有Cloudflare Workers开发、测试和部署使用Wrangler CLI。**

```bash
# 身份验证
wrangler login          # 登录Cloudflare账户
wrangler logout         # 从Cloudflare注销
wrangler whoami         # 检查当前用户

# 开发与测试
wrangler dev           # 启动本地开发服务器（默认端口8787）

# 部署
wrangler deploy        # 将Worker部署到Cloudflare
wrangler deploy --dry-run  # 测试部署而不实际部署

# 配置与类型
wrangler types         # 从Worker配置生成TypeScript类型
```

## 项目架构

**重要：这是一个具有GitHub OAuth身份验证的Cloudflare Workers MCP服务器，用于安全数据库访问。**

### 当前项目结构

```
/
├── src/                          # TypeScript源代码
│   ├── index.ts                  # 主MCP服务器（标准）
│   ├── index_sentry.ts          # 启用Sentry的MCP服务器
│   ├── simple-math.ts           # 基本MCP示例（无身份验证）
│   ├── github-handler.ts        # GitHub OAuth流程实施
│   ├── database.ts              # PostgreSQL连接和实用程序
│   ├── utils.ts                 # OAuth辅助函数
│   ├── workers-oauth-utils.ts   # 基于cookie的批准系统
│   └── tools/                   # 工具注册系统
│       └── register-tools.ts    # 集中式工具注册
├── PRPs/                        # 产品需求提示
│   ├── README.md
│   └── templates/
│       └── prp_mcp_base.md
├── examples/                    # 示例工具创建+注册 - 永远不要编辑或从此文件夹导入
│   ├── database-tools.ts        # Postgres MCP服务器的示例工具，显示工具创建和注册的最佳实践
│   └── database-tools-sentry.ts # 具有用于生产监控的Sentry集成的Postgres MCP服务器的示例工具
├── wrangler.jsonc              # 主Cloudflare Workers配置
├── wrangler-simple.jsonc       # 简单示例配置
├── package.json                # npm依赖项和脚本
├── tsconfig.json               # TypeScript配置
├── worker-configuration.d.ts   # 生成的Cloudflare类型
└── CLAUDE.md                   # 本实施指南
```

### 关键文件用途（始终在此处添加新文件）

**主要实施文件：**

- `src/index.ts` - 具有GitHub OAuth + PostgreSQL的生产MCP服务器
- `src/index_sentry.ts` - 与上述相同，具有Sentry监控集成

**身份验证与安全：**

- `src/github-handler.ts` - 完整的GitHub OAuth 2.0流程
- `src/workers-oauth-utils.ts` - HMAC签名的cookie批准系统
- `src/utils.ts` - OAuth令牌交换和URL构建辅助函数

**数据库集成：**

- `src/database.ts` - PostgreSQL连接池、SQL验证、安全性

**工具注册：**

- `src/tools/register-tools.ts` - 集中式工具注册系统，导入和注册所有工具

**配置文件：**

- `wrangler.jsonc` - 具有Durable Objects、KV、AI绑定的主Worker配置
- `wrangler-simple.jsonc` - 简单示例配置
- `tsconfig.json` - Cloudflare Workers的TypeScript编译器设置

## 开发命令

### 核心工作流命令

```bash
# 设置与依赖项
npm install                  # 安装所有依赖项
npm install --save-dev @types/package  # 添加具有类型的开发依赖项

# 开发
wrangler dev                # 启动本地开发服务器
npm run dev                 # 通过npm脚本的替代方案

# 类型检查与验证
npm run type-check          # 运行TypeScript编译器检查
wrangler types              # 生成Cloudflare Worker类型
npx tsc --noEmit           # 不进行编译的类型检查

# 测试
npx vitest                  # 运行单元测试（如果已配置）

# 代码质量
npx prettier --write .      # 格式化代码
npx eslint src/            # 检查TypeScript代码
```

### 环境配置

**环境变量设置：**

```bash
# 基于.dev.vars.example为本地开发创建.dev.vars文件
cp .dev.vars.example .dev.vars

# 生产密钥（通过Wrangler）
wrangler secret put GITHUB_CLIENT_ID
wrangler secret put GITHUB_CLIENT_SECRET
wrangler secret put COOKIE_ENCRYPTION_KEY
wrangler secret put DATABASE_URL
wrangler secret put SENTRY_DSN
```

## MCP开发上下文

**重要：此项目使用Node.js/TypeScript在具有GitHub OAuth身份验证的Cloudflare Workers上构建生产就绪的MCP服务器。**

### MCP技术栈

**核心技术：**

- **@modelcontextprotocol/sdk** - 官方MCP TypeScript SDK
- **agents/mcp** - Cloudflare Workers MCP代理框架
- **workers-mcp** - Workers的MCP传输层
- **@cloudflare/workers-oauth-provider** - OAuth 2.1服务器实施

**Cloudflare平台：**

- **Cloudflare Workers** - 无服务器运行时（V8隔离）
- **Durable Objects** - 用于MCP代理持久性的有状态对象
- **KV存储** - OAuth状态和会话管理

### MCP服务器架构

此项目将MCP服务器实施为具有三种主要模式的Cloudflare Workers：

**1. 经过身份验证的数据库MCP服务器（`src/index.ts`）：**

**2. 简单数学MCP服务器（`src/simple-math.ts`）：**

**3. 启用Sentry的MCP服务器（`src/index_sentry.ts`）：**

### 实施模式

**关键实施模式：**

1. **工具注册模式**：
   - 使用集中式工具注册（`src/tools/register-tools.ts`）
   - 每个工具实现为具有适当参数验证的TypeScript函数
   - 工具在服务器初始化期间注册

2. **OAuth身份验证模式**：
   - GitHub OAuth 2.0流程用于用户身份验证
   - 具有HMAC签名的基于cookie的会话管理
   - 安全的令牌存储和刷新

3. **数据库集成模式**：
   - 具有连接池的PostgreSQL
   - SQL注入预防
   - 查询参数化

4. **错误处理模式**：
   - 具有用户友好消息的集中式错误处理
   - 生产环境的Sentry集成
   - 适当的HTTP状态代码

### 常见陷阱和最佳实践

**关键陷阱：**

1. **Cloudflare Workers限制**：
   - 50ms CPU时间限制（付费层300ms）
   - 128MB内存限制
   - 无文件系统访问

2. **TypeScript配置**：
   - 使用`tsconfig.json`中的`"target": "es2022"`
   - 为Cloudflare Workers启用`"moduleResolution": "bundler"`

3. **环境变量**：
   - 对机密使用`wrangler secret put`
   - 对非机密配置使用`.dev.vars`
   - 绝不在代码中硬编码API密钥

4. **OAuth流程**：
   - 始终使用HTTPS重定向URL
   - 正确实施状态参数以防止CSRF
   - 安全地存储刷新令牌

### 开发工作流

**标准开发工作流：**

1. **初始设置**：
   ```bash
   npm install
   cp .dev.vars.example .dev.vars
   # 编辑.dev.vars与您的配置
   ```

2. **开发**：
   ```bash
   wrangler dev  # 或 npm run dev
   ```

3. **测试**：
   ```bash
   npm run type-check
   npm test      # 如果已配置
   ```

4. **部署**：
   ```bash
   wrangler deploy
   ```

### 安全最佳实践

**关键安全原则：**

1. **身份验证**：
   - 使用GitHub OAuth进行用户身份验证
   - 实施适当的会话管理
   - 使用安全cookie设置

2. **授权**：
   - 基于用户身份验证实施权限
   - 使用适当的范围进行API访问
   - 验证所有用户输入

3. **数据保护**：
   - 传输中的加密（HTTPS）
   - 静态加密敏感数据
   - 适当的密钥管理

### 监控和可观察性

**生产监控：**

1. **Sentry集成**：
   - 错误跟踪和报告
   - 性能监控
   - 用户会话跟踪

2. **Cloudflare分析**：
   - 请求指标
   - 错误率
   - 性能数据

### 文档和测试

**文档标准：**

1. **代码文档**：
   - 所有公共函数的JSDoc注释
   - README.md中的清晰示例
   - API使用示例

2. **测试策略**：
   - 单元测试用于工具功能
   - OAuth流程的集成测试
   - 端到端测试用于关键用户流程

### 部署清单

**部署前：**

- [ ] 所有测试通过
- [ ] TypeScript编译无错误
- [ ] 环境变量已配置
- [ ] 机密已设置（wrangler secret put）
- [ ] 文档已更新
- [ ] 安全审查已完成

**部署后：**

- [ ] 验证生产部署
- [ ] 测试OAuth流程
- [ ] 验证工具功能
- [ ] 检查错误监控
- [ ] 更新文档中的URL

此实施指南确保使用现代云原生技术构建安全、可扩展、生产就绪的MCP服务器。