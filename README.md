# Hax Agent

> 轻量级、Claude-like 的本地 Agent 工具，CLI 仍是一等入口，同时提供 Electron + Vue 桌面端 · 由 [IdiotTIQS](https://github.com/IdiotTIQS) 开发

[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D18-brightgreen)](https://nodejs.org)
[![License](https://img.shields.io/badge/License-MIT-blue)](#license)
[![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen)](#开发与贡献)
[![npm](https://img.shields.io/npm/v/hax-agent-cli)](https://www.npmjs.com/package/hax-agent-cli)

Hax Agent 是一个面向开发者的 AI 编码助手。终端默认运行 Ink TUI（三段式布局、流式输出、菜单式审批），也提供 Electron + Vue 桌面端。支持 12+ AI 提供商（Anthropic、OpenAI、DeepSeek、Groq、Mistral、Google、Moonshot、智谱、DashScope、OpenRouter、Ollama、vLLM），内置文件工具、shell 执行、Web 搜索（DuckDuckGo，无需 API Key）、Skills/Plugins/Hooks 扩展、MCP 集成、Docker 沙箱、多 Agent 协作（swarm）以及持久化记忆。代码库为 TypeScript + ESM。

---

## 目录

- [功能特性](#功能特性)
- [环境要求](#环境要求)
- [快速开始](#快速开始)
- [命令行用法](#命令行用法)
- [交互式命令](#交互式命令)
- [Skills 技能系统](#skills-技能系统)
- [插件与 Hooks](#插件与-hooks)
- [配置说明](#配置说明)
- [本地工具](#本地工具)
- [架构概览](#架构概览)
- [会话与记忆](#会话与记忆)
- [Provider 档案](#provider-档案)
- [桌面端](#桌面端)
- [开发与测试](#开发与测试)
- [License](#license)

---

## 功能特性

- **Ink TUI 交互界面** — 默认全屏终端 UI：固定三段式布局（历史区 + 活动区 + 输入区）、`<Static>` 已完成轮次渲染、斜杠命令面板、菜单式工具审批、富状态栏（模型 / 上下文用量 / 费用），原生 ink 元素渲染，CJK 对齐正确。
- **经典 readline 备选** — `--legacy` 或 `HAXAGENT_LEGACY=1` 回退到经典 readline 界面，功能等价。
- **桌面 GUI** — Electron + Vue 界面，保留 CLI 工作流，同时提供会话列表、文件树、右侧状态面板和会话恢复。
- **多 Provider 支持** — 内置 12+ Provider：Anthropic（Claude）、OpenAI（GPT）、Google（Gemini）、DeepSeek、Groq、Mistral、Moonshot、智谱、DashScope、OpenRouter、Ollama、vLLM。
- **Provider 档案** — 22 个预置档案（claude、sonnet、haiku、opus、gpt、gpt-mini、o3、deepseek、gemini、local 等），支持自定义，运行时一键切换。
- **本地工具集** — 文件读写/编辑/搜索/Glob、带权限确认的 shell 执行、网页抓取、免 Key 的 DuckDuckGo 网络搜索。
- **多 Agent 协作** — `agent` 子任务工具、task/team 生命周期管理、git worktree 隔离、swarm 子进程/进程内两种后端。
- **定时任务** — `cron.create` 等工具与 `/dream` 目标续跑，支持后台任务。
- **Skills 技能系统** — 将重复性工作流封装为 SKILL.md 文件，用户级与项目级自动发现。
- **插件与 Hook 生命周期** — 10 种生命周期钩子（session.start/end、pre/post.compact、pre/post.tool_use 等），支持插件与脚本扩展。
- **MCP 集成** — stdio/http/ws 传输的 MCP 客户端管理，工具与资源自动发现。
- **沙箱隔离** — Docker / bubblewrap / macOS / Windows 四种后端，`--sandbox` 一键启用 shell 隔离执行。
- **LSP 代码导航** — `/lsp` 命令支持 go-to-definition、find-references 和 workspace 符号搜索。
- **会话与记忆** — 会话快照持久化、`/continue` 恢复上下文、持久化记忆（Markdown 文件 + MEMORY.md 索引）、自动记忆提取、上下文压缩。
- **权限管理** — 四种模式：normal（询问）、yolo（全部批准）、plan（阻止写操作）、full_auto（静默批准）。
- **Cost 追踪** — token 用量与费用统计，支持缓存命中分析。

---

## 环境要求

- **Node.js** >= 18
- **npm**（或 pnpm / yarn）
- **API Key**（至少一个云 Provider；仅使用 Ollama / vLLM 本地推理时无需）

---

## 快速开始

### 1. 安装

```bash
# 从 npm 安装
npm install -g hax-agent-cli

# 或从源码安装
git clone https://github.com/IdiotTIQS/hax-agent-cli.git
cd hax-agent-cli
npm install
npm run build
```

### 2. 启动交互式会话

```bash
hax-agent          # 进入 Ink TUI（默认）
npm run dev        # 源码开发模式（tsx 直接运行 src/cli.ts）
```

首次运行时如果还没有配置 API Key，可以在会话中使用 `/api-key` 命令设置：

```text
/api-key anthropic sk-ant-xxxxxxxxxxxx
/api-key openai sk-xxxxxxxxxxxx
```

也可以直接使用环境变量（见[配置说明](#配置说明)）。

### 3. 切换 Provider 档案

```text
/provider claude        # 切换到 Anthropic Claude
/provider gpt           # 切换到 OpenAI GPT
/provider local         # 切换到本地 Ollama
/provider list          # 查看所有可用档案
```

运行时还可以用 `/models` 查看当前 Provider 的可用模型，`/model <id>` 动态切换。

---

## 命令行用法

```bash
hax-agent                              # 交互式会话（Ink TUI）
hax-agent "解释这段代码"                # 带初始 prompt 启动
hax-agent --provider deepseek          # 指定 Provider
hax-agent --profile opus               # 使用档案快捷方式
hax-agent -y "修复 main.js 的 bug"     # YOLO 模式自动批准所有工具
hax-agent --sandbox                    # 启用 Docker 沙箱隔离 shell 命令
hax-agent --legacy                     # 经典 readline 界面
hax-agent --batch "解释这个项目"        # 单轮批处理，结束即退出
hax-agent --batch --input task.txt --output result.md
hax-agent -v                           # 查看版本
hax-agent --help                       # 查看完整帮助
```

### 完整参数

| 参数 | 说明 |
|------|------|
| `-h`, `--help` | 显示帮助 |
| `-v`, `--version` | 显示版本 |
| `--provider <name>` | 指定 Provider（anthropic、openai、deepseek、groq、mistral、google、moonshot、zhipu、dashscope、openrouter、ollama、vllm） |
| `--model <name>` | 指定模型 ID |
| `--profile <name>` | 使用预置档案（见 [Provider 档案](#provider-档案)） |
| `--permission-mode <mode>` | 权限模式：normal、yolo、plan、full_auto |
| `-y` | `--permission-mode yolo` 的简写 |
| `--api-key <key>` | API Key（覆盖环境变量与已保存的 Key） |
| `--max-turns <n>` | 最大工具执行轮数（默认 200） |
| `--sandbox` | 为 shell 命令启用 Docker 沙箱隔离 |
| `--no-color` | 禁用 ANSI 颜色 |
| `--legacy`, `--no-ink` | 使用经典 readline 界面 |
| `--ink` | 无操作（Ink 已是默认，保留仅为兼容） |
| `--batch <prompt>` | 单轮非交互模式 |
| `--input <file>` | 从文件读取 prompt（配合 `--batch`） |
| `--output <file>` | 将回复写入文件（配合 `--batch`） |

---

## 交互式命令

会话中输入以下斜杠命令（Ink TUI 中输入 `/` 会弹出命令面板）。

### 会话管理

| 命令 | 说明 |
|------|------|
| `/help` | 查看所有可用命令 |
| `/exit` `/quit` `/q` | 退出会话 |
| `/clear` | 清空当前上下文并新建会话 |
| `/continue` | 恢复最近的会话快照继续对话 |
| `/session` | 管理会话快照 |
| `/summary` | 生成当前会话摘要 |
| `/rewind [n]` | 回退最近 n 轮对话（默认 2） |
| `/export` | 导出会话到 JSON 文件 |
| `/copy` | 复制最后一条 AI 回复到剪贴板 |
| `/turns` | 查看轮次统计 |
| `/files` | 列出本次会话修改过的文件 |
| `/tag <name>` / `/tags` | 为会话打标签 / 查看标签 |
| `/version` | 查看版本信息 |

### 模型与 Provider

| 命令 | 说明 |
|------|------|
| `/models` | 查看当前 Provider 可用模型 |
| `/model <id>` | 切换模型 |
| `/provider <name>` | 切换 Provider 档案（`list` 查看所有） |
| `/providers` | 列出所有可用的 AI 提供商 |
| `/api-url <base-url>` | 设置或查看 API Base URL |
| `/api-key <provider> <key>` | 设置 Provider 的 API Key |
| `/status` | 查看会话摘要（模型、费用、tokens） |
| `/cost` | 查看当前会话 token 和费用 |
| `/context` | 查看上下文窗口使用情况 |

### 权限与沙箱

| 命令 | 说明 |
|------|------|
| `/yolo` | 切换 YOLO 模式（自动批准所有工具） |
| `/plan` | 切换 Plan 模式（阻止所有写操作） |
| `/fullauto` | 切换 Full Auto 模式（静默批准所有工具） |
| `/perms` | 查看权限状态 |
| `/permissions [allow\|deny\|reset\|yolo\|normal] [tool]` | 管理工具权限 |
| `/allow <tool>` | 始终允许某个工具 |
| `/deny <tool>` | 始终拒绝某个工具 |
| `/sandbox` | 查看/管理沙箱状态 |

### 记忆与上下文

| 命令 | 说明 |
|------|------|
| `/memory [search\|list]` | 管理持久化记忆 |
| `/compact` | 压缩当前对话，降低上下文占用 |
| `/think` | 调整思考模式 |
| `/personalize` | 从对话中提取环境规则保存到 rules.md |
| `/dream` | 从当前对话提取记忆并续跑目标 |

### 工具与扩展

| 命令 | 说明 |
|------|------|
| `/tools` | 查看可用本地工具列表 |
| `/skills` | 列出 Skills |
| `/mcp` | 查看/管理 MCP 服务 |
| `/plugin` | 插件列表与管理 |
| `/agents` | 子 Agent 管理（含 `stop`） |
| `/goal [--max n] <goal>` | 设置持续目标，直到完成、阻塞或 `/goal clear` |
| `/lsp def <symbol>` / `/lsp search <query>` | 跳转到符号定义 / 搜索工作区符号 |
| `/config` | 查看当前配置 |
| `/init` | 初始化 .hax-agent 项目目录 |
| `/doctor` | 运行环境诊断 |
| `/theme <name>` | 切换终端颜色主题（`list` 查看所有） |
| `/keybindings` | 查看键位绑定 |

### Git 与工作区

| 命令 | 说明 |
|------|------|
| `/diff` | 查看工作区文件改动 |
| `/branch` | 切换/创建分支 |
| `/release-notes` | 查看版本发布说明 |
| `/reload` | 热重载 hooks/插件 |
| `/feedback <message>` | 发送反馈 |

---

## Skills 技能系统

Skills 是可复用的工作流封装，将重复性任务流程保存为 SKILL.md 文件，后续会话中以斜杠命令调用。

### 技能目录结构

```text
~/.haxagent/skills/          # 用户级技能（跨项目可用）
├── code-review/
│   └── SKILL.md
└── deploy-workflow/
    └── SKILL.md

.hax-agent/skills/           # 项目级技能（仅当前项目可用）
├── run-tests/
│   └── SKILL.md
└── ...
```

每个技能是一个目录，包含一个 `SKILL.md` 文件。

### SKILL.md 格式

```markdown
---
name: my-skill
description: 一句话描述这个技能的作用
allowed-tools:
  - file.read
  - file.write
  - shell.run
when_to_use: 描述何时自动调用此技能。以 "Use when..." 开头。
argument-hint: "[arg1] [arg2]"
arguments:
  - arg1
  - arg2
---

# 技能标题

详细描述此技能的工作流程。

## Steps

### 1. 步骤名称
此步骤要做什么。

**Success criteria**: 始终包含！这表明步骤已完成，可以继续下一步。
```

### 调用技能

```text
/code-review                    # 调用代码审查技能
/code-review src/index.ts       # 带参数调用
/skills                         # 列出所有可用技能
```

---

## 插件与 Hooks

插件通过 Hook 生命周期扩展 Agent 行为。插件放在 `~/.haxagent/plugins/`（用户级）或 `.hax-agent/plugins/`（项目级），启动时自动发现。

每个插件目录包含一个 `plugin.json` 清单，可以挂载 hooks 定义和 `skills/` 子目录。可用的生命周期钩子：

```text
session.start / session.end
pre.compact / post.compact
pre.tool_use / post.tool_use
user.prompt_submit
notification / stop / subagent.stop
```

会话中使用 `/plugin` 查看已加载的插件，`/reload` 热重载。

---

## 配置说明

### 配置层级

1. **默认配置** — 内置于 `src/config/settings.ts`
2. **用户配置** — `~/.haxagent/settings.json`
3. **环境变量** — Provider API Key 及路径覆盖
4. **运行时覆盖** — 斜杠命令修改后自动持久化

> 建议不要把包含 API Key 的配置提交到版本库。推荐使用环境变量或用户级配置文件。

### 配置文件示例

`~/.haxagent/settings.json` 的完整结构（默认值）：

```json
{
  "agent": {
    "provider": "anthropic",
    "model": "claude-sonnet-4-6",
    "maxTurns": 25
  },
  "permissions": { "mode": "normal" },
  "tools": { "shell": { "enabled": true } },
  "sandbox": {
    "enabled": true,
    "backend": "auto",
    "image": "node:18-alpine",
    "network": "none",
    "cpus": 2,
    "memory": "512m"
  },
  "ui": { "locale": "en", "autoClearScreen": true },
  "context": { "compactionEnabled": false, "compactionThreshold": 0.85 }
}
```

### 数据目录

| 路径 | 用途 |
|------|------|
| `~/.haxagent/settings.json` | 全局配置 |
| `~/.haxagent/skills/` | 用户级技能 |
| `~/.haxagent/plugins/` | 用户级插件 |
| `~/.haxagent/memories/` | 持久化记忆 |
| `~/.haxagent/sessions/` | 会话快照 |
| `~/.haxagent/data/tasks/` | 后台任务数据 |
| `~/.haxagent/data/cron_jobs.json` | 定时任务注册表 |
| `.hax-agent/` | 项目级运行时数据（配置、技能、插件、回收站） |

### 环境变量

| 变量 | 说明 |
|------|------|
| `HAX_AGENT_PROVIDER` | 默认 Provider 名称 |
| `ANTHROPIC_API_KEY` | Anthropic API Key |
| `OPENAI_API_KEY` | OpenAI API Key |
| `DEEPSEEK_API_KEY` | DeepSeek API Key |
| `GROQ_API_KEY` | Groq API Key |
| `MISTRAL_API_KEY` | Mistral API Key |
| `GOOGLE_API_KEY` | Google API Key |
| `MOONSHOT_API_KEY` | Moonshot API Key |
| `ZHIPUAI_API_KEY` | 智谱 API Key |
| `DASHSCOPE_API_KEY` | DashScope API Key |
| `OPENROUTER_API_KEY` | OpenRouter API Key |
| `HAXAGENT_CONFIG_DIR` | 覆盖全局配置目录（默认 `~/.haxagent`） |
| `HAXAGENT_DATA_DIR` | 覆盖数据目录 |
| `HAXAGENT_LOGS_DIR` | 覆盖日志目录 |
| `HAXAGENT_LEGACY=1` | 所有 shell 默认使用 readline 界面 |

Ollama 与 vLLM 为本地推理，无需 API Key。

---

## 本地工具

Agent 内置一个受限制的工具注册表，所有文件操作均限定在工作区根目录内，防止路径穿越攻击。

### 核心工具

| 工具 | 说明 | 安全限制 |
|------|------|----------|
| `file.read` | 读取工作区内文本文件（支持 offset/limit 分页） | 路径限制在工作区根目录内 |
| `file.write` | 写入工作区内文本文件 | 路径限制在工作区根目录内 |
| `file.edit` | 精准替换文件中的指定文本 | 路径限制在工作区根目录内 |
| `file.delete` | 删除文件（默认移到 `.hax-agent/trash` 回收站） | 路径限制在工作区根目录内 |
| `file.glob` | 按 Glob 模式匹配文件列表 | 路径限制在工作区根目录内 |
| `file.search` | 在文本文件中搜索内容（支持正则/大小写配置） | 支持 ReDoS 防护 |
| `shell.run` | 执行本地命令 | 非 yolo 模式下由权限确认决定；可配合 `--sandbox` 隔离 |
| `web.fetch` | 获取网页内容并转为纯文本 | URL 校验 |
| `web.search` | DuckDuckGo 网络搜索 | 无需 API Key |

### 扩展工具（自动注册）

- **多 Agent**：`agent`（子任务委派）、`send_message`（Agent 间消息）、`team.create` / `team.delete`
- **任务**：`task.create` / `task.get` / `task.list` / `task.output` / `task.stop` / `task.update`
- **计划模式**：`enter_plan_mode` / `exit_plan_mode`
- **工作树**：`enter_worktree` / `exit_worktree` / `list_worktrees`
- **其他**：`todo_write`、`cron.create` / `cron.delete` / `cron.list` / `cron.toggle`、`ask_user`、`grep`、`glob`、`skill`、`lsp`、`notebook_edit`
- **图像**：`image_to_text`、`image_generation`
- **MCP**：`list_mcp_tools`、`list_mcp_resources`、`read_mcp_resource`、`mcp_auth`

---

## 架构概览

代码库为 TypeScript + ESM（NodeNext），分层架构：`core/`（类型协议）→ `engine/`（Agent 运行时）→ `api/` / `tools/` / `services/`（实现层）。

```
src/
├── cli.ts                       # CLI 入口：参数解析、Ink TUI / readline 分发
├── index.ts                     # 库入口：全模块 barrel 导出
├── tui-ink/                     # Ink TUI（默认界面）
│   ├── App.tsx                  # 根组件：三段式布局 + Static 历史
│   ├── reducer.ts               # 状态机（engine 事件 → UI 状态）
│   ├── components/              # ToolCall、DiffView、CommandPalette、
│   │                            # ApprovalPrompt、StatusBar、ThinkingBlock…
│   └── ui/                      # 原语层：Select、Separator、useTerminalSize
├── core/                        # 基础层 — 类型化协议
│   ├── api/                     # Provider 适配器协议、错误分类
│   ├── messages/                # StandardMessage、ContentBlock 类型
│   ├── memory/                  # Token 估算与压缩工具
│   └── permissions/             # 权限检查器
├── engine/                      # Agent 运行时
│   ├── agent.ts                 # AgentEngine 主循环、Session、HookExecutor
│   ├── query.ts                 # QueryContext 状态追踪
│   └── cost-tracker.ts          # 费用追踪
├── api/                         # Provider 客户端（12+ 提供商）与重试
├── tools/                       # 工具注册表（核心 9 个 + 扩展集）
├── commands/                    # 55 个斜杠命令
├── config/                      # settings、profiles、paths
├── skills/ plugins/ hooks/      # 扩展生态
├── memory/                      # 持久化记忆（store、compact、relevance）
├── services/                    # LSP、MCP、cron、session 存储、token 估算
├── sandbox/                     # 沙箱后端：docker、bwrap、macos、win
├── swarm/                       # 多 Agent 协作（subprocess / in-process）
├── channels/                    # IM 通道适配器（飞书、Slack、Discord 等 11 个）
├── prompts/                     # 系统提示词组装
├── auth/                        # 认证管理
└── shared/                      # 主题、ANSI 工具

desktop/                         # Electron + Vue 3 桌面应用
├── main/                        # Electron 主进程（IPC、审批桥）
├── preload/                     # 预加载脚本
└── renderer/                    # Vue 3 前端（Vite）
```

### 核心数据流

```
用户输入 → cli.ts → commands/registry（斜杠命令）
                      ↓ 普通消息
                  engine/agent.ts（AgentEngine 工具循环）
                      ↓                    ↓
              api/provider.ts        tools/registry.ts
              （LLM 流式响应）        （工具执行 + 权限检查）
                      ↓
              tui-ink/reducer.ts（事件 → 状态）→ 终端渲染
```

AgentEngine 通过 async generator 产出类型化事件（`message.delta`、`tool.start`、`tool.result`、`turn.completed` 等），TUI 消费这些事件做增量渲染，桌面端通过 IPC 转发同一事件流。

---

## 会话与记忆

### 会话快照

会话快照保存在 `~/.haxagent/sessions/`，每个快照为 JSON 文件，包含消息、token 用量、工具调用计数和时间戳。`/continue` 恢复最近快照，`/session` 管理快照列表。

### 持久化记忆

记忆以 Markdown 文件保存在 `~/.haxagent/memories/`，自动生成 `MEMORY.md` 索引：

- 支持签名去重与 TTL 过期清理
- `/memory search <query>` 按标题、内容、标签打分检索
- `/dream` 从对话中自动提取记忆（LLM 提取，限最重要的 3 条）
- `/personalize` 提取环境规则保存到 rules.md
- 每条记忆支持 category、scope、importance、tags 元数据

---

## Provider 档案

通过 `/provider <name>` 或 `--profile <name>` 使用预置档案：

| 档案 | Provider | 模型 |
|------|----------|------|
| `claude` / `sonnet` | Anthropic | claude-sonnet-4-6 |
| `haiku` | Anthropic | claude-haiku-4-5-20251001 |
| `opus` | Anthropic | claude-opus-4-7 |
| `gpt` / `gpt-pro` / `gpt-mini` / `o3` | OpenAI | gpt-5.4-mini 等 |
| `deepseek` / `deepseek-pro` | DeepSeek | deepseek-v4-flash |
| `groq` | Groq | llama-3.3-70b-versatile |
| `mistral` | Mistral | mistral-large-latest |
| `google` / `gemini` / `gemini-pro` | Google | gemini-2.5-pro / gemini-2.5-flash |
| `moonshot` | Moonshot | moonshot-v1-8k |
| `zhipu` | 智谱 | glm-4.5-plus |
| `dashscope` | DashScope | qwen-max-latest |
| `openrouter` | OpenRouter | anthropic/claude-sonnet-4.6 |
| `ollama` | Ollama（本地） | llama3.3 |
| `local` | Ollama 兼容端点 | 本地配置 |
| `vllm` | vLLM（本地） | default |

自定义档案通过 `~/.haxagent/profiles.json` 管理，支持 `/provider <name>` 一键切换。

---

## 桌面端

桌面端与 CLI 共用同一套配置、会话存储和工具层。

```bash
npm run desktop:dev     # 启动开发模式
npm run desktop:build   # 构建前端资源
npm run desktop:start   # 直接启动 Electron
```

---

## 开发与测试

### 常用命令

| 命令 | 说明 |
|------|------|
| `npm run dev` | 以 tsx 直接运行 `src/cli.ts`（开发） |
| `npm run build` | TypeScript 编译到 `dist/`（tsc） |
| `npm start` | 运行编译产物 `dist/cli.js` |
| `npm run typecheck` | `tsc --noEmit` 类型检查 |
| `npm test` | 运行测试套件（node --test + tsx） |
| `npm run desktop:dev` | 桌面端开发模式 |

### 目录规范

- `src/` — TypeScript 源码（ESM，NodeNext），分层架构
- `desktop/` — 桌面端源码，与 CLI 共用核心层
- `test/` — 测试文件（需登记到 `scripts/run-tests.js` 才会执行）
- `dist/` — 构建产物（npm 发布内容）

### 开发与贡献

1. Fork 本仓库
2. 创建特性分支（`git checkout -b feat/amazing-feature`）
3. 提交修改（`git commit -m 'feat: add amazing feature'`）
4. 推送并打开 Pull Request

欢迎提交 Issue 和 PR。请确保新增功能包含对应测试用例，并将测试文件加入 `scripts/run-tests.js` 的执行列表。

---

## License

MIT © [IdiotTIQS](https://github.com/IdiotTIQS)

---

*Hax Agent — 让 AI 编码助手在你的终端里为你服务。*
