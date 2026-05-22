# FDE-Kit

面向 Forward Deployed Engineer（FDE）的企业 AI 工作台部署指南。

FDE-Kit 记录了一条分阶段的部署路线：把本地 AI Agent 变成团队可访问、可审查、可治理的企业 AI 助手。它把本地 Agent Runtime、`cc-connect`、办公平台 CLI、`llm-wiki-compiler` 和企业定制 Skills 串成一套可交付流程。

English version: [README.en.md](README.en.md)

## 系统架构

```mermaid
flowchart LR
  User[员工] --> Chat[飞书 / 企微 / 钉钉]
  Chat --> Bridge{Chat Bridge}
  Bridge -->|多平台| cc[cc-connect]
  Bridge -->|飞书专精| lark[lark-channel-bridge]
  cc --> Agent[本地 AI Agent]
  lark --> Agent
  Agent --> Platform[办公平台 CLI]
  Agent --> Wiki[已审查 LLM Wiki]
  Agent --> Skills[企业 Skills]
  Platform --> Data[文档 / 表格 / 群聊 / 通讯录]
  Data --> Compiler[llm-wiki-compiler]
  Compiler --> Review[人工 Review]
  Review --> Wiki
  Skills --> Workflow[日志 / 指标 / 报告 / SOP]
  Agent --> Chat
```

目标结果：

> 搭建一个团队可访问的企业 AI 工作台：它能读取授权的企业上下文，基于来源回答问题，并在明确的安全边界内执行工作流。

## 部署路线图

```text
Step 0  本地 Agent Runtime
Step 1  Chat Bridge
Step 2  企业数据访问
Step 3  LLM Wiki
Step 4  企业 Skills
Step 5  Governance
```

Step 0-3 建立基础设施。  
Step 4 产生业务价值。  
Step 5 让系统具备企业级运行边界。

## 环境要求

- macOS 或 Windows
- Node.js >= 24
- npm
- 一个本地 Agent Runtime：Claude Code、Codex、Cursor、Gemini CLI、OpenCode 或同类工具
- 一个企业聊天平台：飞书、企微、钉钉、Slack、Telegram、Discord、QQ 或同类平台
- 一批已授权的测试文档
- 一个专用工作目录

检查 Node.js：

```bash
node -v
npm -v
```

安装 Node.js：

```bash
# macOS
brew install node
```

```powershell
# Windows
winget install OpenJS.NodeJS
```

## Step 0：本地 Agent Runtime

先安装并验证一个本地 Agent Runtime，再把它接入任何企业系统。

推荐选项：

- Claude Code
- Codex
- Claude Code + 低成本模型供应商，例如 DeepSeek、GLM、Kimi

安装 Claude Code：

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

验证：

```text
Agent 能在一个专用测试目录内读写文件、执行任务。
```

## Step 1：Chat Bridge

Chat Bridge 把本地 Agent 暴露给团队聊天平台。根据目标平台选择：

### 方案 A：cc-connect（多平台通用）

`cc-connect` 支持飞书、企微、钉钉、Slack、Telegram、Discord 等多平台。

安装：

```bash
npm install -g cc-connect
cc-connect --version
```

生成与当前版本匹配的配置：

```bash
cc-connect config example > cc-connect.toml
```

先配置一个平台。以飞书为例：

```bash
cc-connect feishu setup
```

绑定已有飞书应用：

```bash
cc-connect feishu setup --app <app_id:app_secret>
```

或：

```bash
cc-connect feishu bind --app <app_id:app_secret>
```

在 `cc-connect.toml` 里重点确认：

```text
project.name                              # 项目名称
projects.agent.type                       # Agent 类型
projects.agent.options.work_dir           # Agent 工作目录
projects.platforms.type                   # 平台类型
projects.platforms.options.app_id         # 平台应用 ID
projects.platforms.options.app_secret     # 平台应用密钥
projects.platforms.options.allow_from     # 授权用户白名单
```

进入团队试点前，`allow_from` 应该明确限制授权用户。

启动：

```bash
cc-connect --config ./cc-connect.toml
```

### 方案 B：lark-channel-bridge（飞书专精）

如果只用飞书，`lark-channel-bridge` 提供更好的飞书原生体验。

优势（相比 cc-connect）：

- QR 码一键创建飞书 PersonalAgent 应用，无需手动去开放平台配置
- 流式卡片：Claude 回复实时更新在一张飞书卡片上，不用等整段回复
- 多 workspace：`/ws` 切换项目目录，每个聊天独立 session
- 内置访问控制：`allowedUsers` / `allowedChats` / `admins` 三层白名单
- 消息中断 + 合并：新消息能打断正在跑的任务，连续发消息合并成一次请求

安装：

```bash
npm install -g lark-channel-bridge
```

启动：

```bash
lark-channel-bridge start
```

首次运行会显示 QR 码，扫码即可绑定飞书 PersonalAgent 应用。

在飞书 Developer Console 确认权限和事件订阅：

权限：`im:message`、`im:message:send_as_bot`、`im:resource`

事件：`im.message.receive_v1`、`card.action.trigger`

访问控制（通过飞书内 `/config` 设置）：

```text
allowedUsers    # 允许对话的用户 open_id 列表
allowedChats    # 允许触发回复的群聊 chat_id 列表（DM 始终放行）
admins          # 可执行 /config、/cd、/ws 等管理命令的用户
```

### 方案选择

| 维度 | cc-connect | lark-channel-bridge |
|---|---|---|
| 支持平台 | 飞书、企微、钉钉、Slack 等 | 仅飞书 |
| 首次配置 | 手动建应用 + 配事件 | QR 码扫码即用 |
| 回复体验 | 等整段回复完 | 流式卡片实时更新 |
| 访问控制 | allow_from 白名单 | 三层白名单（用户/群聊/管理员） |
| Session 管理 | 无内置 | 每聊独立 session + workspace |
| 适用场景 | 多平台团队 | 飞书为主的个人或小团队 |

验证：

```text
用户能从聊天平台发送消息，并收到本地 Agent 的回复。
```

注意：两个方案都只支持 IM 聊天消息（群聊/私聊），不支持飞书文档内的 @ 机器人。

## Step 2：企业数据访问

Chat Bridge 让 Agent 能被访问。企业数据访问让 Agent 真正有用。

为选定的办公平台安装 CLI、MCP Server 或内部工具适配器。第一阶段建议只读，并限制在已批准的测试空间内。

飞书 / Lark：

```bash
npm install -g @larksuite/cli
lark-cli --help
lark-cli config init
lark-cli auth login
lark-cli doctor
```

典型能力：

- 搜索文档
- 读取文档
- 读取表格
- 查询通讯录
- 向授权群聊发送进度汇报
- 上传生成的文件或图片

验证：

```text
Agent 能读取一份授权测试文档，并向一个授权群聊发送一条测试汇报。
```

## Step 3：LLM Wiki

`llm-wiki-compiler` 用来把授权源文档编译成可审查的 LLM Wiki。

安装：

```bash
npm install -g llm-wiki-compiler
llmwiki --help
```

创建测试工作区：

```bash
mkdir -p ~/fde-demo/sources
cd ~/fde-demo
```

Windows PowerShell：

```powershell
mkdir $env:USERPROFILE\fde-demo\sources
cd $env:USERPROFILE\fde-demo
```

创建 `sources/example.md`：

```markdown
# 示例公司 SOP

## P0 事故响应流程

发生 P0 事故时，创建事故群，指定唯一负责人，检查监控面板，并每 15 分钟同步一次进展。

## 外部 SaaS 订阅

员工订阅外部 SaaS 工具前，应先向财务确认报销和续费规则。
```

编译：

```bash
llmwiki schema init
llmwiki compile --review --lang zh-CN
llmwiki review list
llmwiki review show <candidate-id>
llmwiki review approve <candidate-id>
llmwiki lint
llmwiki view
```

查询：

```bash
llmwiki query "P0 事故应该怎么处理？"
```

验证：

```text
已记录的问题能从已审查 Wiki 内容中回答。
未记录的制度问题会被标记为“未文档化”，而不是被编造。
```

## Step 4：企业 Skills

Skills 把基础设施转化成业务工作流。

一个 Skill 是一套可重复执行的流程，包含：

- 触发条件
- 输入
- 授权工具
- 执行步骤
- 输出格式
- 权限边界
- 失败处理

### 日志诊断 Skill

触发：

```text
排查这个失败请求：trace_id=xxx
```

流程：

```text
1. 校验 trace_id。
2. 查询授权日志。
3. 识别请求路径、用户范围和错误类型。
4. 必要时检查相关指标。
5. 输出诊断结论和下一步动作。
6. 如有需要，汇报到授权群聊。
```

### API 交付验收 Skill

触发：

```text
这个 API 改动准备交付，帮我做一次验收。
```

流程：

```text
1. 读取需求和 API 路径。
2. 运行测试。
3. 发送冒烟请求。
4. 检查日志和指标。
5. 输出通过 / 失败 / 风险报告。
```

### 内容审核 Skill

触发：

```text
审核这一批用户内容。
```

流程：

```text
1. 读取授权数据。
2. 应用审核标准。
3. 输出结构化审核决策。
4. 写入授权目标位置。
5. 标记需要人工复核的样本。
```

### 竞品监控 Skill

触发：

```text
总结最近的竞品动态。
```

流程：

```text
1. 读取授权关键词和竞品列表。
2. 搜索公开来源。
3. 提取标题、互动数据和核心主张。
4. 聚类选题方向。
5. 向授权群聊输出报告。
```

### SOP 执行 Skill

触发：

```text
执行发布检查清单。
```

流程：

```text
1. 读取已批准的 SOP。
2. 检查必要项目。
3. 确认负责人。
4. 确认回滚和监控方案。
5. 输出发布 / 暂停发布摘要。
```

### Wiki 更新 Skill

触发：

```text
把这次事故复盘整理成 Wiki 更新。
```

流程：

```text
1. 提取时间线、影响范围、原因和修复动作。
2. 起草复盘文档。
3. 起草可复用的事故模式。
4. 提交人工 review。
5. 审批后发布。
```

验证：

```text
至少一个 Skill 能端到端完成真实工作流，并产出可审计报告。
```

## Step 5：Governance

最低运行控制：

- 专用 OS 用户或隔离服务器
- 专用工作目录
- 聊天白名单
- 源文档白名单
- 第一次集成优先只读
- Wiki 发布前必须人工 review
- 源文档中不放密钥、凭证、合同、工资、客户隐私数据
- 未知问题捕获流程
- 定期 wiki lint 和 review

验证：

```text
Agent 能回答已知问题，拒绝未文档化问题，并只在预期数据边界内运行。
```

## LLM Wiki vs 直接 RAG

| 维度 | 直接 RAG | LLM Wiki |
|---|---|---|
| 知识形态 | 查询时临时检索 chunks | 预先编译成 Markdown 页面 |
| 可审查性 | 通常只能审查最终回答 | Wiki 页面可在使用前审查 |
| 来源追溯 | 依赖检索结果 | 页面和引用中内置来源 |
| 复用性 | 每次查询重新发现 | 持续沉淀为稳定知识资产 |
| 适用场景 | 大规模临时搜索 | SOP、制度、复盘、手册 |
| 主要风险 | 回答看似合理但未被批准 | 前期设置较慢，但控制更强 |

FDE-Kit 不替代 RAG。它定义的是：什么时候企业知识应该先变成可审查资产，再交给 Agent 使用。

## 交付检查清单

```text
[ ] Step 0: 本地 Agent 能在测试目录内运行
[ ] Step 1: cc-connect 能把聊天平台桥接到 Agent
[ ] Step 2: Agent 能访问一份授权测试文档
[ ] Step 3: LLM Wiki 能编译，并基于已审查内容回答
[ ] Step 4: 至少一个企业 Skill 能完成真实工作流
[ ] Step 5: 权限、审查和运营流程已文档化
```

## 公开安全基线

公开示例只使用通用数据。

避免发布：

- 真实项目名
- 内部域名
- 生产路径
- 员工信息
- 客户信息
- 密钥或凭证
- 原始事故记录
- 未发布商业计划

## 范围

FDE-Kit 是 FDE 风格企业 AI 工作台项目的部署指南和运行模型。

当前范围：

- 分阶段部署指南
- 运行边界
- 可审查知识工作流
- Skill 设计示例

暂不包含：

- SaaS 界面
- 一键安装脚本
- 自动导入未经审查的公司文档
- 对未批准来源的回答正确性保证

预期结果是一条从本地 Agent 到企业 AI 工作台的可控路径。
