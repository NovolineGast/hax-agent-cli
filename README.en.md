# Hax Agent

> Lightweight, Claude-like local agent tooling with CLI as the primary entry point and an Electron + Vue desktop app · Developed by [IdiotTIQS](https://github.com/IdiotTIQS)

[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D18-brightgreen)](https://nodejs.org)
[![License](https://img.shields.io/badge/License-MIT-blue)](#license)
[![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen)](#development--contributing)
[![npm](https://img.shields.io/npm/v/hax-agent-cli)](https://www.npmjs.com/package/hax-agent-cli)

Hax Agent is an AI coding assistant for developers. The terminal runs an Ink TUI by default (three-part layout, streaming output, menu-driven approvals), with an Electron + Vue desktop app alongside. It supports 12+ AI providers (Anthropic, OpenAI, DeepSeek, Groq, Mistral, Google, Moonshot, Zhipu, DashScope, OpenRouter, Ollama, vLLM), built-in file tools, shell execution, web search (DuckDuckGo, no API key needed), Skills/Plugins/Hooks extension, MCP integration, Docker sandboxing, multi-agent collaboration (swarm), and persistent memory. The codebase is TypeScript + ESM.

---

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Command Line Usage](#command-line-usage)
- [Interactive Commands](#interactive-commands)
- [Skills System](#skills-system)
- [Plugins & Hooks](#plugins--hooks)
- [Configuration](#configuration)
- [Local Tools](#local-tools)
- [Architecture Overview](#architecture-overview)
- [Sessions & Memory](#sessions--memory)
- [Provider Profiles](#provider-profiles)
- [Desktop App](#desktop-app)
- [Development & Testing](#development--testing)
- [License](#license)

---

## Features

- **Ink TUI interface** — Full-screen terminal UI by default: fixed three-part layout (history + live area + input), `<Static>` rendering of completed turns, slash command palette, menu-driven tool approvals, rich status bar (model / context usage / cost), native ink elements with correct CJK alignment.
- **Classic readline fallback** — `--legacy` or `HAXAGENT_LEGACY=1` switches to the classic readline interface with feature parity.
- **Desktop GUI** — Electron + Vue interface that keeps the CLI workflow while adding session lists, file tree browsing, a right-side status panel, and session recovery.
- **Multi-Provider Support** — Built-in 12+ providers: Anthropic (Claude), OpenAI (GPT), Google (Gemini), DeepSeek, Groq, Mistral, Moonshot, Zhipu, DashScope, OpenRouter, Ollama, vLLM.
- **Provider Profiles** — 22 pre-configured profiles (claude, sonnet, haiku, opus, gpt, gpt-mini, o3, deepseek, gemini, local, etc.) with custom profile support and one-command switching.
- **Local Toolset** — File read/write/edit/search/glob, permission-gated shell execution, web fetching, and key-free DuckDuckGo web search.
- **Multi-Agent Collaboration** — `agent` subtask tool, task/team lifecycle management, git worktree isolation, subprocess and in-process swarm backends.
- **Scheduled Tasks** — `cron.create` and related tools, `/dream` goal continuation, background tasks.
- **Skills System** — Package repetitive workflows into SKILL.md files, auto-discovered at both user and project level.
- **Plugin & Hook Lifecycle** — 10 lifecycle hooks (session.start/end, pre/post.compact, pre/post.tool_use, etc.) for plugin and script extension.
- **MCP Integration** — MCP client management over stdio/http/ws transports with automatic tool and resource discovery.
- **Sandbox Isolation** — Docker / bubblewrap / macOS / Windows backends; enable shell isolation with `--sandbox`.
- **LSP Code Navigation** — `/lsp` command for go-to-definition, find-references, and workspace symbol search.
- **Sessions & Memory** — Persistent session snapshots, `/continue` context recovery, persistent memory (Markdown files + MEMORY.md index), automatic memory extraction, context compaction.
- **Permission Management** — Four modes: normal (ask), yolo (auto-approve all), plan (block writes), full_auto (silent auto-approve).
- **Cost Tracking** — Token usage and cost statistics with cache hit analysis.

---

## Prerequisites

- **Node.js** >= 18
- **npm** (or pnpm / yarn)
- **API Key** (at least one cloud provider; not needed for Ollama / vLLM local inference)

---

## Quick Start

### 1. Installation

```bash
# Install from npm
npm install -g hax-agent-cli

# Or install from source
git clone https://github.com/IdiotTIQS/hax-agent-cli.git
cd hax-agent-cli
npm install
npm run build
```

### 2. Start an Interactive Session

```bash
hax-agent          # Ink TUI (default)
npm run dev        # Development mode (tsx runs src/cli.ts directly)
```

On first launch, configure your API keys from within the session:

```text
/api-key anthropic sk-ant-xxxxxxxxxxxx
/api-key openai sk-xxxxxxxxxxxx
```

You can also use environment variables (see [Configuration](#configuration)).

### 3. Switch Provider Profiles

```text
/provider claude        # Switch to Anthropic Claude
/provider gpt           # Switch to OpenAI GPT
/provider local         # Switch to local Ollama
/provider list          # List all available profiles
```

Use `/models` to view available models for the current provider and `/model <id>` to switch at runtime.

---

## Command Line Usage

```bash
hax-agent                              # Interactive session (Ink TUI)
hax-agent "explain this code"          # Start with an initial prompt
hax-agent --provider deepseek          # Specify a provider
hax-agent --profile opus               # Use a profile shortcut
hax-agent -y "fix the bug in main.js"  # YOLO mode, auto-approve all tools
hax-agent --sandbox                    # Docker sandbox for shell commands
hax-agent --legacy                     # Classic readline interface
hax-agent --batch "explain this project"   # Single-turn batch, exits when done
hax-agent --batch --input task.txt --output result.md
hax-agent -v                           # Show version
hax-agent --help                       # Full help
```

### All Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show help |
| `-v`, `--version` | Show version |
| `--provider <name>` | LLM provider (anthropic, openai, deepseek, groq, mistral, google, moonshot, zhipu, dashscope, openrouter, ollama, vllm) |
| `--model <name>` | Model identifier |
| `--profile <name>` | Use a builtin profile (see [Provider Profiles](#provider-profiles)) |
| `--permission-mode <mode>` | Permission mode: normal, yolo, plan, full_auto |
| `-y` | Shorthand for `--permission-mode yolo` |
| `--api-key <key>` | API key (overrides env var and saved key) |
| `--max-turns <n>` | Maximum tool execution turns (default: 200) |
| `--sandbox` | Enable Docker sandbox isolation for shell commands |
| `--no-color` | Disable ANSI colors |
| `--legacy`, `--no-ink` | Use the classic readline interface |
| `--ink` | No-op (ink is now the default; kept for compat) |
| `--batch <prompt>` | Single non-interactive turn |
| `--input <file>` | Read prompt from file (with `--batch`) |
| `--output <file>` | Write response to file (with `--batch`) |

---

## Interactive Commands

Type these slash commands in a session (typing `/` in the Ink TUI opens the command palette).

### Session Management

| Command | Description |
|---------|-------------|
| `/help` | Show all available commands |
| `/exit` `/quit` `/q` | Exit the session |
| `/clear` | Clear the current context and start a new session |
| `/continue` | Resume the most recent session snapshot |
| `/session` | Manage session snapshots |
| `/summary` | Generate a summary of the current session |
| `/rewind [n]` | Roll back the last n turns (default 2) |
| `/export` | Export the session to a JSON file |
| `/copy` | Copy the last AI reply to the clipboard |
| `/turns` | Show turn statistics |
| `/files` | List files modified in this session |
| `/tag <name>` / `/tags` | Tag the session / list tags |
| `/version` | Show version information |

### Model & Provider

| Command | Description |
|---------|-------------|
| `/models` | List available models for the current provider |
| `/model <id>` | Switch model |
| `/provider <name>` | Switch provider profile (`list` shows all) |
| `/providers` | List all available AI providers |
| `/api-url <base-url>` | Set or view the API base URL |
| `/api-key <provider> <key>` | Set a provider API key |
| `/status` | Session summary (model, cost, tokens) |
| `/cost` | Token and cost usage for this session |
| `/context` | Context window usage |

### Permissions & Sandbox

| Command | Description |
|---------|-------------|
| `/yolo` | Toggle YOLO mode (auto-approve all tools) |
| `/plan` | Toggle Plan mode (block all mutating tools) |
| `/fullauto` | Toggle Full Auto mode (silent auto-approve) |
| `/perms` | Show permission status |
| `/permissions [allow\|deny\|reset\|yolo\|normal] [tool]` | Manage tool permissions |
| `/allow <tool>` | Always allow a tool |
| `/deny <tool>` | Always deny a tool |
| `/sandbox` | View/manage sandbox status |

### Memory & Context

| Command | Description |
|---------|-------------|
| `/memory [search\|list]` | Manage persistent memory |
| `/compact` | Compact the conversation to reduce context usage |
| `/think` | Adjust thinking mode |
| `/personalize` | Extract environment rules into rules.md |
| `/dream` | Extract memories from the conversation and continue goals |

### Tools & Extensions

| Command | Description |
|---------|-------------|
| `/tools` | List available local tools |
| `/skills` | List skills |
| `/mcp` | View/manage MCP servers |
| `/plugin` | Plugin list and management |
| `/agents` | Sub-agent management (including `stop`) |
| `/goal [--max n] <goal>` | Set a persistent goal until done, blocked, or `/goal clear` |
| `/lsp def <symbol>` / `/lsp search <query>` | Go to definition / search workspace symbols |
| `/config` | Show current configuration |
| `/init` | Initialize the .hax-agent project directory |
| `/doctor` | Run environment diagnostics |
| `/theme <name>` | Switch terminal color theme (`list` shows all) |
| `/keybindings` | Show keybindings |

### Git & Workspace

| Command | Description |
|---------|-------------|
| `/diff` | Show workspace file changes |
| `/branch` | Switch/create a branch |
| `/release-notes` | Show release notes |
| `/reload` | Hot-reload hooks/plugins |
| `/feedback <message>` | Send feedback |

---

## Skills System

Skills are reusable workflow packages: save a repetitive task flow as a SKILL.md file and invoke it as a slash command in later sessions.

### Directory Layout

```text
~/.haxagent/skills/          # User-level skills (available across projects)
├── code-review/
│   └── SKILL.md
└── deploy-workflow/
    └── SKILL.md

.hax-agent/skills/           # Project-level skills (current project only)
├── run-tests/
│   └── SKILL.md
└── ...
```

Each skill is a directory containing one `SKILL.md` file.

### SKILL.md Format

```markdown
---
name: my-skill
description: One-line description of what this skill does
allowed-tools:
  - file.read
  - file.write
  - shell.run
when_to_use: Describe when to invoke this skill. Start with "Use when...".
argument-hint: "[arg1] [arg2]"
arguments:
  - arg1
  - arg2
---

# Skill Title

Describe the workflow in detail.

## Steps

### 1. Step name
What this step does.

**Success criteria**: Always include! This signals the step is complete.
```

### Invoking a Skill

```text
/code-review                    # Invoke the code review skill
/code-review src/index.ts       # With arguments
/skills                         # List all available skills
```

---

## Plugins & Hooks

Plugins extend agent behavior through the hook lifecycle. Place them in `~/.haxagent/plugins/` (user-level) or `.hax-agent/plugins/` (project-level); they are discovered automatically at startup.

Each plugin directory contains a `plugin.json` manifest and may mount hook definitions and a `skills/` subdirectory. Available lifecycle hooks:

```text
session.start / session.end
pre.compact / post.compact
pre.tool_use / post.tool_use
user.prompt_submit
notification / stop / subagent.stop
```

Use `/plugin` to view loaded plugins and `/reload` to hot-reload.

---

## Configuration

### Configuration Layers

1. **Defaults** — built into `src/config/settings.ts`
2. **User config** — `~/.haxagent/settings.json`
3. **Environment variables** — provider API keys and path overrides
4. **Runtime overrides** — slash command changes are persisted automatically

> Do not commit configuration containing API keys. Prefer environment variables or the user-level config file.

### Config File Example

Full structure of `~/.haxagent/settings.json` (defaults):

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

### Data Directories

| Path | Purpose |
|------|---------|
| `~/.haxagent/settings.json` | Global configuration |
| `~/.haxagent/skills/` | User-level skills |
| `~/.haxagent/plugins/` | User-level plugins |
| `~/.haxagent/memories/` | Persistent memory |
| `~/.haxagent/sessions/` | Session snapshots |
| `~/.haxagent/data/tasks/` | Background task data |
| `~/.haxagent/data/cron_jobs.json` | Cron job registry |
| `.hax-agent/` | Project-level runtime data (config, skills, plugins, trash) |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `HAX_AGENT_PROVIDER` | Default provider name |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `OPENAI_API_KEY` | OpenAI API key |
| `DEEPSEEK_API_KEY` | DeepSeek API key |
| `GROQ_API_KEY` | Groq API key |
| `MISTRAL_API_KEY` | Mistral API key |
| `GOOGLE_API_KEY` | Google API key |
| `MOONSHOT_API_KEY` | Moonshot API key |
| `ZHIPUAI_API_KEY` | Zhipu API key |
| `DASHSCOPE_API_KEY` | DashScope API key |
| `OPENROUTER_API_KEY` | OpenRouter API key |
| `HAXAGENT_CONFIG_DIR` | Override the global config directory (default `~/.haxagent`) |
| `HAXAGENT_DATA_DIR` | Override the data directory |
| `HAXAGENT_LOGS_DIR` | Override the logs directory |
| `HAXAGENT_LEGACY=1` | Make readline the default interface in all shells |

Ollama and vLLM run local inference and need no API key.

---

## Local Tools

The agent has a restricted tool registry; all file operations are confined to the workspace root to prevent path traversal attacks.

### Core Tools

| Tool | Description | Safety Constraint |
|------|-------------|-------------------|
| `file.read` | Read a workspace text file (offset/limit pagination) | Path confined to workspace root |
| `file.write` | Write a workspace text file | Path confined to workspace root |
| `file.edit` | Replace exact text in a file | Path confined to workspace root |
| `file.delete` | Delete a file (moved to `.hax-agent/trash` by default) | Path confined to workspace root |
| `file.glob` | Match files by glob pattern | Path confined to workspace root |
| `file.search` | Search file contents (regex/case options) | ReDoS protection |
| `shell.run` | Run a local command | Permission-gated outside yolo mode; `--sandbox` isolation optional |
| `web.fetch` | Fetch a URL as plain text | URL validation |
| `web.search` | DuckDuckGo web search | No API key needed |

### Extended Tools (auto-registered)

- **Multi-agent**: `agent` (subtask delegation), `send_message` (inter-agent messaging), `team.create` / `team.delete`
- **Tasks**: `task.create` / `task.get` / `task.list` / `task.output` / `task.stop` / `task.update`
- **Plan mode**: `enter_plan_mode` / `exit_plan_mode`
- **Worktrees**: `enter_worktree` / `exit_worktree` / `list_worktrees`
- **Others**: `todo_write`, `cron.create` / `cron.delete` / `cron.list` / `cron.toggle`, `ask_user`, `grep`, `glob`, `skill`, `lsp`, `notebook_edit`
- **Images**: `image_to_text`, `image_generation`
- **MCP**: `list_mcp_tools`, `list_mcp_resources`, `read_mcp_resource`, `mcp_auth`

---

## Architecture Overview

The codebase is TypeScript + ESM (NodeNext) with a layered architecture: `core/` (typed protocols) → `engine/` (agent runtime) → `api/` / `tools/` / `services/` (implementations).

```
src/
├── cli.ts                       # CLI entry: arg parsing, Ink TUI / readline dispatch
├── index.ts                     # Library entry: full module barrel export
├── tui-ink/                     # Ink TUI (default interface)
│   ├── App.tsx                  # Root component: three-part layout + Static history
│   ├── reducer.ts               # State machine (engine events → UI state)
│   ├── components/              # ToolCall, DiffView, CommandPalette,
│   │                            # ApprovalPrompt, StatusBar, ThinkingBlock…
│   └── ui/                      # Primitives: Select, Separator, useTerminalSize
├── core/                        # Foundation layer — typed protocols
│   ├── api/                     # Provider adapter protocol, error classification
│   ├── messages/                # StandardMessage, ContentBlock types
│   ├── memory/                  # Token estimation and compaction utilities
│   └── permissions/             # Permission checker
├── engine/                      # Agent runtime
│   ├── agent.ts                 # AgentEngine main loop, Session, HookExecutor
│   ├── query.ts                 # QueryContext state tracking
│   └── cost-tracker.ts          # Cost tracking
├── api/                         # Provider clients (12+ providers) and retry
├── tools/                       # Tool registry (9 core + extended set)
├── commands/                    # 55 slash commands
├── config/                      # settings, profiles, paths
├── skills/ plugins/ hooks/      # Extension ecosystem
├── memory/                      # Persistent memory (store, compact, relevance)
├── services/                    # LSP, MCP, cron, session storage, token estimation
├── sandbox/                     # Sandbox backends: docker, bwrap, macos, win
├── swarm/                       # Multi-agent collaboration (subprocess / in-process)
├── channels/                    # IM channel adapters (Feishu, Slack, Discord, etc. — 11)
├── prompts/                     # System prompt assembly
├── auth/                        # Authentication management
└── shared/                      # Themes, ANSI utilities

desktop/                         # Electron + Vue 3 desktop app
├── main/                        # Electron main process (IPC, approval bridge)
├── preload/                     # Preload scripts
└── renderer/                    # Vue 3 frontend (Vite)
```

### Core Data Flow

```
User input → cli.ts → commands/registry (slash commands)
                      ↓ chat message
                  engine/agent.ts (AgentEngine tool loop)
                      ↓                    ↓
              api/provider.ts        tools/registry.ts
              (LLM streaming)        (tool execution + permission checks)
                      ↓
              tui-ink/reducer.ts (events → state) → terminal rendering
```

AgentEngine is an async generator that yields typed events (`message.delta`, `tool.start`, `tool.result`, `turn.completed`, etc.). The TUI consumes these events for incremental rendering; the desktop app forwards the same event stream over IPC.

---

## Sessions & Memory

### Session Snapshots

Session snapshots are stored in `~/.haxagent/sessions/` as JSON files containing messages, token usage, tool call counts, and timestamps. `/continue` resumes the most recent snapshot; `/session` manages the snapshot list.

### Persistent Memory

Memories are stored as Markdown files in `~/.haxagent/memories/` with an auto-generated `MEMORY.md` index:

- Signature deduplication and TTL expiry cleanup
- `/memory search <query>` scores matches by title, content, and tags
- `/dream` automatically extracts memories from conversation (LLM extraction, top 3)
- `/personalize` extracts environment rules into rules.md
- Each memory supports category, scope, importance, and tags metadata

---

## Provider Profiles

Use `/provider <name>` or `--profile <name>`:

| Profile | Provider | Model |
|---------|----------|-------|
| `claude` / `sonnet` | Anthropic | claude-sonnet-4-6 |
| `haiku` | Anthropic | claude-haiku-4-5-20251001 |
| `opus` | Anthropic | claude-opus-4-7 |
| `gpt` / `gpt-pro` / `gpt-mini` / `o3` | OpenAI | gpt-5.4-mini etc. |
| `deepseek` / `deepseek-pro` | DeepSeek | deepseek-v4-flash |
| `groq` | Groq | llama-3.3-70b-versatile |
| `mistral` | Mistral | mistral-large-latest |
| `google` / `gemini` / `gemini-pro` | Google | gemini-2.5-pro / gemini-2.5-flash |
| `moonshot` | Moonshot | moonshot-v1-8k |
| `zhipu` | Zhipu | glm-4.5-plus |
| `dashscope` | DashScope | qwen-max-latest |
| `openrouter` | OpenRouter | anthropic/claude-sonnet-4.6 |
| `ollama` | Ollama (local) | llama3.3 |
| `local` | Ollama-compatible endpoint | local config |
| `vllm` | vLLM (local) | default |

Custom profiles are managed via `~/.haxagent/profiles.json` and switchable with `/provider <name>`.

---

## Desktop App

The desktop app shares the same configuration, session storage, and tool layer as the CLI.

```bash
npm run desktop:dev     # Start dev mode
npm run desktop:build   # Build frontend assets
npm run desktop:start   # Launch Electron directly
```

---

## Development & Testing

### Common Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Run `src/cli.ts` directly via tsx (development) |
| `npm run build` | Compile TypeScript to `dist/` (tsc) |
| `npm start` | Run the compiled `dist/cli.js` |
| `npm run typecheck` | `tsc --noEmit` type check |
| `npm test` | Run the test suite (node --test + tsx) |
| `npm run desktop:dev` | Desktop development mode |

### Directory Conventions

- `src/` — TypeScript source (ESM, NodeNext), layered architecture
- `desktop/` — Desktop app source, shares the core layer with the CLI
- `test/` — Test files (must be registered in `scripts/run-tests.js` to run)
- `dist/` — Build output (npm package content)

### Development & Contributing

1. Fork this repository
2. Create a feature branch (`git checkout -b feat/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push and open a Pull Request

Issues and PRs are welcome. Please include tests for new features and register the test file in the `scripts/run-tests.js` execution list.

---

## License

MIT © [IdiotTIQS](https://github.com/IdiotTIQS)

---

*Hax Agent — an AI coding assistant at your service in the terminal.*
