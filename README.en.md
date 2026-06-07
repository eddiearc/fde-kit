# FDE-Kit

FDE-Kit is a curated tool map for Forward Deployed Engineers.

It is not a one-command installer and does not try to assemble a full enterprise AI system for you. It answers a more practical question:

> When you bring AI into real business workflows, which tools are worth knowing first?

FDE work is rarely “install a chatbot.” It is usually about decomposing a business problem into context, tools, workflows, permissions, delivery, and review. The tools below can be combined as needed.

中文版本：[README.md](README.md)

## Tool Map

```text
Agent Runtime      Make AI execute tasks
Chat Bridge        Let teams use Agents from chat tools
Office CLI         Let Agents read and operate office systems
Web Automation     Let Agents operate websites and legacy web apps
Knowledge Layer    Turn documents into reviewable context
Workflow Tools     Productize specific business workflows
Governance         Control permissions, review, and risk
```

## 1. Agent Runtime

A local or cloud Agent is the execution core of an FDE toolchain.

| Tool | Best for |
|---|---|
| [Claude Code](https://claude.ai/code) | Code, files, and complex multi-step work. A strong default FDE runtime. |
| [Codex](https://openai.com/codex/) | Coding, engineering tasks, and automated development workflows. |
| [Cursor](https://cursor.com/) | AI programming inside the IDE and codebase collaboration. |
| [OpenCode](https://github.com/sst/opencode) | Open-source terminal Agent / coding assistant. |
| Gemini CLI / other CLI Agents | Alternative execution layers for different model ecosystems. |

Selection rule: start with one Agent you can reliably control, one that can read/write files, run commands, and produce reviewable outputs.

## 2. Chat Bridge

A bridge lets teammates call the Agent from chat instead of opening a terminal.

| Tool | Best for |
|---|---|
| [cc-connect](https://www.npmjs.com/package/cc-connect) | General multi-platform bridge for multiple chat platforms and Agent runtimes. |
| [lark-channel-bridge](https://www.npmjs.com/package/lark-channel-bridge) | Feishu/Lark-first bridge for Claude Code in DMs, groups, and cloud-doc comments. |

Good for: internal AI assistants, project-room assistants, doc-comment assistants, and remote FDE delivery entry points.

Warning: production environments need user/chat allowlists. Do not let anyone trigger a local Agent by default.

## 3. Office CLI

Enterprise AI often needs to read documents, sheets, chats, calendars, contacts, and task systems.

| Tool | Best for |
|---|---|
| [lark-cli](https://www.npmjs.com/package/@larksuite/cli) | Feishu/Lark docs, sheets, chats, calendar, tasks, approvals, and more. |
| Microsoft Graph CLI / SDK | Microsoft 365, Outlook, Teams, SharePoint. |
| Google Workspace CLI / SDK | Gmail, Docs, Sheets, Calendar. |
| Internal CLI / OpenAPI | ERP, CRM, tickets, logs, metrics, and internal systems. |

For the first phase, prefer read-only access. Let the Agent read approved context before granting write actions.

## 4. Web Automation

Many enterprise systems do not have convenient APIs. Real workflows often still live inside web admin consoles.

| Tool | Best for |
|---|---|
| [OpenCLI](https://github.com/jackwener/opencli) | Turning websites, public pages, and SaaS/admin UIs into Agent-callable CLIs. |
| Playwright | Custom browser automation, form operations, and end-to-end testing. |
| Browserbase / Stagehand | Cloud browser automation and higher-level web operations. |

Good for: web research, competitor monitoring, lead collection, legacy admin data entry, and PoCs for systems without APIs.

Safety boundary: do not bypass logins, permissions, anti-abuse systems, or paywalls. Payments, deletions, contract submissions, and outbound messages require human confirmation.

## 5. Knowledge Layer

FDEs need to turn enterprise knowledge into Agent-usable, reviewable, traceable context.

| Tool | Best for |
|---|---|
| [llm-wiki-compiler](https://www.npmjs.com/package/llm-wiki-compiler) | Compiling approved documents into a reviewable LLM Wiki. |
| Open WebUI / AnythingLLM / Dify Knowledge | Quick knowledge-base and RAG prototypes. |
| RepoWiki / DeepWiki / RepoAgent-style tools | Codebase knowledge and engineering documentation. |

Recommendation: high-risk knowledge such as policies, SOPs, and incident reviews should be human-reviewed before Agent use.

## 6. Workflow Tools

These tools are not always core FDE infrastructure, but they are useful references or components for specific business workflows.

| Tool | Best for |
|---|---|
| [MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo) | Topic → script → footage → voiceover → subtitles → short video. Useful for content drafts and topic testing. |
| n8n | Low-code workflow automation across APIs, webhooks, sheets, and messaging systems. |
| Dify / Coze / FastGPT | Fast AI apps and workflows for business users. |
| Apify | Web data extraction and automation Actors. |
| LangGraph / CrewAI | Custom multi-Agent orchestration. |

MoneyPrinterTurbo’s role: not the main engine for personal-brand content, but a good example of a content-production Skill that chains multiple steps into one workflow.

## 7. Governance

A tool that runs is not necessarily a tool that can be delivered.

At minimum, an FDE project needs:

- Dedicated account / workspace directory.
- Chat allowlist and data-source allowlist.
- Read-only first, writes later.
- Human confirmation for high-risk actions.
- Logs, task records, and review notes.
- No secrets, customer privacy, contracts, salaries, or sensitive data in public examples.
- Unknown or undocumented questions should be answered as “unknown / not documented,” not invented.

## A Simple Selection Rule

```text
If the problem is “how can the team use the Agent?” → Chat Bridge
If the problem is “how can the Agent read company context?” → Office CLI + Knowledge Layer
If the problem is “there is no API but the website works” → OpenCLI / Playwright
If the problem is “how do we run one business process?” → Workflow Tools
If the problem is “can this be delivered to a customer?” → Governance first
```

## Contributing

When adding a tool, prefer this format:

```text
Tool name:
Problem it solves:
Best for:
Not for:
FDE usage note:
```

FDE-Kit’s goal is not to collect the most tools. Its goal is to help FDEs quickly decide where a tool belongs in a real business workflow.
