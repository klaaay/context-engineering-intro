## 功能：

- 具有另一个Pydantic AI代理作为工具的Pydantic AI代理。
- 主代理的研究代理，然后是子代理的邮件草稿代理。
- 与代理交互的CLI。
- 邮件草稿代理使用Gmail，研究代理使用Brave API。

## 示例：

在 `examples/` 文件夹中，有一个README供您阅读以了解示例的全部内容，以及当您创建上述功能的文档时如何构建自己的README。

- `examples/cli.py` - 使用此作为模板创建CLI
- `examples/agent/` - 阅读这里的所有文件，了解创建支持不同提供商和LLM的Pydantic AI代理、处理代理依赖项以及向代理添加工具的最佳实践。

不要直接复制任何这些示例，它完全是针对不同项目的。但请以此作为灵感和最佳实践。

## 文档：

Pydantic AI文档：https://ai.pydantic.dev/

## 其他考虑因素：

- 包括一个.env.example，README包含设置说明，包括如何配置Gmail和Brave。
- 在README中包含项目结构。
- 虚拟环境已经设置并安装了必要的依赖项。
- 使用python_dotenv和load_env()处理环境变量