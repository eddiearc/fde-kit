# FDE-Kit

FDE-Kit 是一份面向 Forward Deployed Engineer（FDE）的工具清单。

它不试图提供一键安装脚本，也不承诺搭出一整套企业 AI 系统。它只回答一个更实际的问题：

> 当你要把 AI 带进真实业务流程时，有哪些工具值得先知道？

FDE 的工作通常不是“装一个聊天机器人”，而是把业务问题拆成：上下文、工具、流程、权限、交付和复盘。下面这些工具可以按需组合。

English version: [README.en.md](README.en.md)

## 工具地图

```text
Agent Runtime      让 AI 能执行任务
Chat Bridge        让团队能在聊天工具里使用 Agent
Office CLI         让 Agent 读取和操作企业办公系统
Web Automation     让 Agent 操作网页和旧后台
Knowledge Layer    把文档变成可审查上下文
Workflow Tools     把具体业务流程产品化
Governance         控制权限、审查和风险
```

## 1. Agent Runtime

本地或云端 Agent 是 FDE 工具链的执行核心。

| 工具 | 适合场景 |
|---|---|
| [Claude Code](https://claude.ai/code) | 代码、文件、复杂多步任务，适合作为 FDE 默认基座之一。 |
| [Codex](https://openai.com/codex/) | 代码生成、工程任务、自动化开发流程。 |
| [Cursor](https://cursor.com/) | IDE 内的 AI 编程和代码库协作。 |
| [OpenCode](https://github.com/sst/opencode) | 终端里的开源 Agent / coding assistant。 |
| Gemini CLI / 其他 CLI Agent | 作为备选执行器，适合接入不同模型生态。 |

选择原则：先选一个你能稳定控制、能读写文件、能跑命令、能复盘输出的 Agent。

## 2. Chat Bridge

Bridge 让团队不用打开终端，也能在聊天工具里调用 Agent。

| 工具 | 适合场景 |
|---|---|
| [cc-connect](https://www.npmjs.com/package/cc-connect) | 多平台、多 Agent Runtime 的通用桥接层。 |
| [lark-channel-bridge](https://www.npmjs.com/package/lark-channel-bridge) | 飞书优先，把 Claude Code 接进飞书私聊、群聊、文档评论。 |

适合做：内部 AI 助手、项目群助手、文档评论助手、FDE 远程交付入口。

注意：生产环境必须配置用户/群聊白名单，不要让任何人都能触发本地 Agent。

## 3. Office CLI

企业 AI 落地经常需要读文档、表格、群聊、日历、通讯录和任务系统。

| 工具 | 适合场景 |
|---|---|
| [lark-cli](https://www.npmjs.com/package/@larksuite/cli) | 飞书文档、表格、群聊、日历、任务、审批等自动化。 |
| Microsoft Graph CLI / SDK | Microsoft 365、Outlook、Teams、SharePoint。 |
| Google Workspace CLI / SDK | Gmail、Docs、Sheets、Calendar。 |
| 企业内部 CLI / OpenAPI | ERP、CRM、工单、日志、监控等内部系统。 |

第一阶段建议只读：先让 Agent 能读取授权上下文，再逐步开放写入能力。

## 4. Web Automation

很多企业系统没有好用 API，真实流程仍然在网页后台里。

| 工具 | 适合场景 |
|---|---|
| [OpenCLI](https://github.com/jackwener/opencli) | 把网页、公开网站、SaaS 后台变成 Agent 可调用 CLI。 |
| Playwright | 自定义浏览器自动化、表单操作、端到端测试。 |
| Browserbase / Stagehand | 云端浏览器自动化和更高层的网页操作。 |

适合做：网页资料读取、竞品监控、线索收集、旧后台录入、无 API 系统的 PoC。

安全边界：不绕过登录、权限、风控或付费墙；付款、删除、合同提交、外发消息等高风险动作必须人工确认。

## 5. Knowledge Layer

FDE 需要把企业知识变成 Agent 可用、可审查、可追溯的上下文。

| 工具 | 适合场景 |
|---|---|
| [llm-wiki-compiler](https://www.npmjs.com/package/llm-wiki-compiler) | 把授权文档编译成可审查 LLM Wiki。 |
| Open WebUI / AnythingLLM / Dify Knowledge | 快速搭建知识库和 RAG 原型。 |
| RepoWiki / DeepWiki / RepoAgent 类工具 | 代码仓库知识层和工程文档理解。 |

建议：企业制度、SOP、事故复盘这类高风险知识，最好先人工 review，再交给 Agent 使用。

## 6. Workflow Tools

这些工具不一定是 FDE 基础设施，但可以作为具体业务工作流的参考或组件。

| 工具 | 适合场景 |
|---|---|
| [MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo) | 主题 → 文案 → 素材 → 配音 → 字幕 → 短视频的自动化流水线。适合内容草稿和选题测试。 |
| n8n | 低代码工作流编排，适合 API、Webhook、表格和消息系统串联。 |
| Dify / Coze / FastGPT | 快速搭建面向业务用户的 AI 应用和工作流。 |
| Apify | 网页数据采集和自动化 Actor。 |
| LangGraph / CrewAI | 需要自定义多 Agent 编排时使用。 |

MoneyPrinterTurbo 的定位：它不是个人 IP 内容的主引擎，但可以作为“内容生产 Skill”的样例，说明 AI 工作流如何把多个步骤串成一条流水线。

## 7. Governance

工具能跑起来不等于能交付。

FDE 项目至少需要这些边界：

- 专用账号 / 专用工作目录。
- 聊天白名单和数据源白名单。
- 只读优先，写入后置。
- 高风险动作人工确认。
- 日志、任务记录和复盘。
- 不把密钥、客户隐私、合同、工资等敏感数据放进公开示例。
- 未文档化问题要回答“不知道 / 未记录”，不要编造。

## 一个简单的 FDE 选型方式

```text
如果问题是“团队怎么用 Agent” → 看 Chat Bridge
如果问题是“Agent 怎么读企业资料” → 看 Office CLI + Knowledge Layer
如果问题是“没有 API 但网页能操作” → 看 OpenCLI / Playwright
如果问题是“把一个业务流程跑起来” → 看 Workflow Tools
如果问题是“能不能交付给客户” → 先看 Governance
```

## 贡献

欢迎补充工具，但建议按下面格式：

```text
工具名：
解决什么问题：
适合什么场景：
不适合什么场景：
FDE 使用提醒：
```

FDE-Kit 的目标不是收集最多工具，而是帮助 FDE 快速判断：这个工具在真实业务流程里能放在哪一层。
