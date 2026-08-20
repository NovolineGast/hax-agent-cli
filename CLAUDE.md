# HaxAgent CLI — Developer Guide

## Project Overview

HaxAgent is a professional AI coding assistant with a Claude-like CLI experience. It supports 12+ LLM providers (Anthropic, OpenAI, DeepSeek, Groq, Mistral, Google, Moonshot, Zhipu, DashScope, Ollama, vLLM, OpenRouter), agent teams, plugins, skills, session memory, and a desktop UI.

The codebase is **TypeScript + ESM** (`"type": "module"`, module resolution `NodeNext`, target ES2022). All relative imports must carry the `.js` extension (the source file is `.ts`, but the specifier refers to the compiled output). The interactive default interface is an **Ink TUI** (`src/tui-ink/`); the classic readline interface remains available via `--legacy`.

- **Language:** TypeScript (strict), ESM, Node.js >= 18
- **Entry point (CLI):** `src/cli.ts` (bin: `hax-agent` → `dist/cli.js`)
- **Entry point (library):** `src/index.ts` (main: `hax-agent-cli` → `dist/index.js`)
- **Desktop UI:** Electron + Vue 3 under `desktop/`
- **Tests:** Node.js built-in test runner (`node --test`) executed through `tsx`
- **Runtime deps:** `@anthropic-ai/sdk`, `openai`, `@google/generative-ai`, `ink`, `react`, `ink-text-input`, `markdown-it`, `dompurify`

## Quick Start

```bash
npm install            # install dependencies
npm run dev            # run the CLI from source (tsx src/cli.ts)
npm run build          # compile TypeScript to dist/ (tsc)
npm start              # run the compiled dist/cli.js
npm run typecheck      # tsc --noEmit (this is also what `npm run lint` runs today)
npm test               # run the test suite

npm run desktop:dev    # desktop development mode
npm run desktop:build  # build desktop frontend assets
```

Before committing, `npm run typecheck` and `npm test` must both pass.

## Architecture & Data Flow

```
User input
  -> src/cli.ts                         -- arg parsing; dispatch Ink TUI or readline
    -> src/tui-ink/run.tsx              -- default: render <App/>, wire approval bridge
      -> src/tui-ink/App.tsx            -- handleSubmit -> engine.sendMessage()
        -> src/engine/agent.ts          -- AgentEngine: prompt build, tool loop
          -> src/api/provider.ts        -- LLM streaming (12+ providers, withRetry)
          -> src/tools/registry.ts      -- tool execution + permission checks
          -> src/core/permissions/      -- permission evaluation
        <- typed events (async generator): message.delta / tool.start /
           tool.result / turn.completed / turn.failed ...
      -> src/tui-ink/reducer.ts         -- events -> UI state
      -> terminal rendering (Static history + live area)
```

Key points for a user message:

1. `cli.ts` parses flags and starts the Ink TUI (default) or the legacy readline loop.
2. Slash commands route through `src/commands/registry.ts` and `src/commands/extended-commands.ts` (~55 commands); plain messages go to `engine.sendMessage()`.
3. `AgentEngine` assembles the system prompt (skills + context + history) and runs the tool loop.
4. `api/provider.ts` streams LLM responses; Anthropic tool names with dots are sanitized for the API.
5. Tool calls execute via `tools/registry.ts` after permission checks; results feed back into the conversation until the model stops calling tools.
6. The engine yields typed events; the Ink reducer consumes them for incremental rendering. The desktop app forwards the same event stream over Electron IPC.

The approval bridge between engine and TUI uses a deferred-promise pattern: `run.tsx` creates the approval callback before `App` renders, `dispatchRef` exposes the reducer dispatch, and resolutions are applied via `setImmediate` with a one-shot `resolvedRef` guard. Preserve this contract when touching approval code.

## Directory Structure

```
src/
├── cli.ts                        # CLI entry: arg parsing, Ink/readline dispatch
├── index.ts                      # Library barrel: nested default export (api.engine.Session, api.tools.ToolRegistry, ...)
├── context.ts, session.ts, setup.ts, platforms.ts, pricing.ts
├── renderer*.ts                  # readline-era renderers (used by --legacy path)
├── tui-ink/                      # Ink TUI (default interface)
│   ├── App.tsx                   # root component, three-part layout + <Static>
│   ├── run.tsx                   # bootstrap: engine wiring, approval bridge
│   ├── reducer.ts                # state machine (engine events + UI actions)
│   ├── types.ts                  # AppState, actions, CommittedTurn
│   ├── keybindings.tsx           # key handling (incl. Ctrl+R detail toggle)
│   ├── markdown.ts               # markdown -> ink elements
│   ├── completions.ts            # slash command completion
│   ├── components/               # ToolCall, ToolList, DiffView, CommandPalette,
│   │                             # ApprovalPrompt, StatusBar, ConversationTurn,
│   │                             # ThinkingBlock, TextStream, SpinnerLine, UserInput
│   └── ui/                       # primitives: Select (+select-state), Separator, useTerminalSize
├── core/                         # Foundation layer — typed protocols (no deps upward)
│   ├── api/                      # errors.ts (error classification), provider-adapter.ts (ApiStreamEvent, adapters)
│   ├── messages/                 # StandardMessage, ContentBlock discriminated union
│   ├── memory/                   # compaction utilities
│   └── permissions/              # PermissionChecker, modes, sensitive paths
├── engine/                       # Agent runtime
│   ├── agent.ts                  # AgentEngine, Session, HookExecutor, PermissionChecker
│   ├── query.ts                  # QueryContext: task focus, file tracking, maxTurns (default 200)
│   ├── cost-tracker.ts           # token/cost accounting
│   └── stream-events.ts          # event type definitions
├── api/                          # Provider layer
│   ├── provider.ts               # REGISTRY (12+ providers), createProvider, streaming clients
│   ├── registry.ts, retry.ts     # provider registry helpers, exponential backoff with Retry-After
│   └── codex-client.ts, copilot-client.ts, copilot-auth.ts, usage.ts
├── tools/
│   ├── registry.ts               # ToolRegistry + 9 core tools (file.*, shell.run, web.*)
│   ├── extended.ts               # extended tool set (agent, task.*, team.*, cron.*, todo_write, ...)
│   ├── agent-tool.ts             # subagent delegation
│   ├── mcp-tools.ts              # list/read MCP resources and tools
│   ├── plan-mode-tool.ts, worktree-tool.ts, send-message-tool.ts, remote-trigger-tool.ts, image-tools.ts
├── commands/
│   ├── registry.ts               # ~37 core slash commands
│   └── extended-commands.ts      # ~18 extended commands (/session, /mcp, /plugin, /diff, ...)
├── config/
│   ├── settings.ts               # defaults + ~/.haxagent/settings.json merge
│   ├── profiles.ts               # ProfileManager, builtin profiles
│   └── paths.ts                  # all data directory resolution (HAXAGENT_* env overrides)
├── skills/                       # SKILL.md discovery, frontmatter parsing, prompt generation
├── plugins/                      # plugin.json manifest validation, registry, installer
├── hooks/                        # hook registry: command/http/prompt/agent hook types, hot-reload
├── memory/                       # persistent memory store, compaction, relevance ranking, scan, team memory
├── prompts/                      # system prompt assembly (skills, context, environment)
├── services/                     # lsp, mcp (+mcp-bootstrap), cron, session-storage, session-memory,
│                                 # token-estimation, memory-extract, personalization, autodream
├── sandbox/                      # docker / bwrap / macOS / Windows backends, session, path-validator
├── swarm/                        # multi-agent: registry, in-process + subprocess backends, mailbox, worktree
├── channels/                     # IM channel adapters (feishu, slack, discord, telegram, dingtalk, email,
│                                 # matrix, mochat, qq, wechat, whatsapp) + bus
├── tasks/                        # background task manager, local agent task, stop-task
├── auth/                         # auth manager, flows, storage, external
├── keybindings/                  # default bindings, loader, parser, resolver
├── autopilot/ bridge/ coordinator/ personalization/ output-styles/ state/ vim/ voice/
└── shared/                       # themes.ts, utils.ts (ANSI, styled())

desktop/                          # Electron + Vue 3 app
├── main/index.js                 # main process: IPC handlers (agent:sendMessage, approval:request/respond)
├── preload/index.js              # contextBridge exposing window.haxAgent
└── renderer/                     # Vue 3 + Vite frontend (App.vue, components/, composables/)

test/                             # node:test files (must be listed in scripts/run-tests.js)
test-helpers/                     # assertions, fixtures, mocks, temp dirs
scripts/                          # run-tests.js, dev/build/start helpers for desktop
docs/plans/, docs/superpowers/    # dated design documents and specs (historical records)
```

## Core Module Responsibilities

| Module | Responsibility |
|--------|---------------|
| `src/engine/agent.ts` | AgentEngine async-generator tool loop, Session state, HookExecutor lifecycle dispatch, approval prompts |
| `src/engine/query.ts` | QueryContext: task focus, read files, skills invoked, work log, maxTurns (default 200) |
| `src/api/provider.ts` | Unified streaming provider clients; `REGISTRY` maps provider names to class + env key + default model; `createProvider` factory |
| `src/api/retry.ts` | `withRetry`: retryable status codes/messages, Retry-After parsing, exponential backoff with jitter |
| `src/core/api/provider-adapter.ts` | Typed adapter protocol (`ApiStreamEvent`), Anthropic/OpenAI adapters |
| `src/core/messages/types.ts` | StandardMessage, ContentBlock discriminated union, token estimation |
| `src/core/permissions/checker.ts` | PermissionChecker: normal/yolo/plan/full_auto, always-allow/deny sets, sensitive path detection |
| `src/tools/registry.ts` | ToolRegistry: 9 core tools, `isReadOnly` classification, workspace path sandboxing |
| `src/commands/registry.ts` + `extended-commands.ts` | ~55 slash commands: `register(name, handler, description)` |
| `src/tui-ink/reducer.ts` | Pure state machine: engine events and UI actions -> AppState |
| `src/config/settings.ts` | Defaults deep-merged with `~/.haxagent/settings.json` (cached; `reloadSettings` busts cache) |
| `src/config/paths.ts` | All directory resolution: `~/.haxagent/` base, `data/`, `logs/`, project `.hax-agent/`; `HAXAGENT_CONFIG_DIR` / `HAXAGENT_DATA_DIR` / `HAXAGENT_LOGS_DIR` overrides |
| `src/memory/store.ts` | MemoryStore (Markdown memories + MEMORY.md index, signature dedup, TTL) and SessionMemoryStore (`~/.haxagent/sessions/` JSON snapshots) |
| `src/memory/compact.ts` | Micro-compaction (old tool results) and full LLM summarization; CompactionManager |
| `src/services/lsp.ts` | Symbol extraction for JS/TS/Python, go-to-definition, find-references, workspace search |
| `src/services/mcp.ts` | MCP client manager: stdio/http/ws transports, tool discovery, JSON config |
| `src/skills/registry.ts` | SKILL.md frontmatter parsing, user + project discovery, `<available_skills>` prompt block |

## Module Organization Principles

1. **Layered architecture.** `core/` (protocols, types) → `engine/` (runtime) → `api/` / `tools/` / `services/` (implementations). Dependencies flow downward only.
2. **Provider-agnostic engine.** AgentEngine works with any provider through the streaming interface; provider specifics live in `api/`.
3. **Tools are self-describing.** Each tool registers `{ name, description, inputSchema, execute, isReadOnly }`. The schema is sent to the LLM for function calling.
4. **Event-driven streaming.** The engine loop is an async generator yielding typed events; the Ink reducer and the desktop IPC bridge both consume the same stream.
5. **Hooks for lifecycle extension.** HookExecutor dispatches: `session.start`, `session.end`, `pre.compact`, `post.compact`, `pre.tool_use`, `post.tool_use`, `user.prompt_submit`, `notification`, `stop`, `subagent.stop`.
6. **Permission modes.** `normal` (ask), `yolo` (auto-approve), `plan` (block mutating tools), `full_auto` (silent auto-approve), plus per-tool always-allow/always-deny sets.
7. **Tool output offloading.** Large tool outputs (>8000 chars) are written to disk with inline previews to avoid context bloat.
8. **ESM everywhere.** Relative imports require the `.js` extension. `__dirname` is `import.meta.dirname`. JSON is read with `fs` or imported with `with { type: "json" }`.

## How to Add a New Feature

### Add a Slash Command

In `src/commands/registry.ts` (or `extended-commands.ts` for extended set):

```ts
register("commandname", (args, ctx) => {
  // args: string[]  —  ctx: { screen, session, rl, settings, engine, ... }
  ctx.screen.write("output\n");
  ctx.rl?.prompt?.();
}, "One-line description shown in /help");
```

### Add a Tool

Add the tool definition to `src/tools/registry.ts` (core) or `src/tools/extended.ts` (extended set):

```ts
const myTool: ToolDefinition = {
  name: "tool.name",
  description: "What it does",
  inputSchema: { type: "object", required: ["path"], properties: { path: { type: "string" } } },
  async execute(args, ctx) {
    // return { ok: true, data: {...} } or { ok: false, error: { code, message } }
  },
  isReadOnly: (args) => true,
};
```

Tools in `extended.ts` are merged automatically by `createDefaultRegistry()`.

### Add a Provider

Add an entry to the `REGISTRY` object in `src/api/provider.ts`:

```ts
providername: { cls: BaseOpenAICompatible, envKey: "PROVIDER_API_KEY", url: "https://api.example.com/v1", model: "default-model", name: "providername" },
```

For non-OpenAI-compatible APIs, implement a class with an async-generator `stream()` method following `AnthropicProvider`. Consider adding a matching profile in `src/config/profiles.ts`.

### Add a Skill

Create `~/.haxagent/skills/<name>/SKILL.md` or `<project>/.hax-agent/skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`, `when_to_use`, optional `allowed-tools`, `arguments`). Discovery is automatic.

### Add a Plugin

Create a plugin directory with a `plugin.json` manifest under `~/.haxagent/plugins/` or `<project>/.hax-agent/plugins/`. Plugins register hooks on lifecycle events; see `src/hooks/registry.ts` for hook types (command/http/prompt/agent).

## Coding Conventions

- **Language:** TypeScript, strict mode, ESM with NodeNext resolution. Relative imports must end with `.js`.
- **Formatting:** 2-space indentation, semicolons, double quotes (match existing files).
- **Naming:** camelCase for functions/variables, PascalCase for classes/components, UPPER_SNAKE for constants.
- **Error handling:** try/catch for async operations. Tool errors return `{ ok: false, error: { code, message } }`; never throw across the engine boundary.
- **JSDoc:** document public APIs with `@param` / `@returns` / `@throws`; explain why, not what.
- **Imports:** order Node built-ins → npm packages → local modules; use destructuring for named imports.
- **Ink components:** use native ink elements (`<Box>`, `<Text color>`) — do not introduce ANSI-passthrough rendering. New interactive primitives belong in `src/tui-ink/ui/`.

## Testing Conventions

- **Framework:** Node.js built-in test runner (`node:test`, `node:assert/strict`).
- **Runner:** `npm test` executes `scripts/run-tests.js`, which spawns `node --import tsx --test <files>` with a hard-coded file list, a 15s per-test timeout (Node >= 20.10), and a 300s global timeout.
- **Critical gotcha:** a new test file is NOT picked up automatically — add its path to the list in `scripts/run-tests.js` or it will never run.
- **File naming:** `test/<module>.test.ts` mirrors `src/<module>.ts`.
- **Run a single file:** `node --import tsx --test test/tui-ink-reducer.test.ts`
- **Helpers:** `test-helpers/` provides assertions, fixtures, mocks, and temp directory utilities.
- **Ink tests:** reducer and pure-logic tests run headless (see `tui-ink-reducer.test.ts`, `tui-ink-select-state.test.ts`); interactive rendering is verified via the preview harnesses (`src/tui-ink/*-preview.tsx`) and manual TTY checks.

## Key Design Patterns

1. **Async generator streaming:** `AgentEngine.sendMessage()` yields typed events consumed by the TUI for incremental rendering.
2. **Reducer state machine:** `tui-ink/reducer.ts` is pure — engine events and UI actions are the only inputs; this keeps rendering testable.
3. **Hook system (observer):** HookExecutor dispatches lifecycle events with fnmatch-style tool name matching.
4. **Strategy for providers:** adapters implement a common streaming interface so the engine stays provider-agnostic.
5. **Factory functions:** `createProvider()`, `createDefaultRegistry()`, `loadSkillRegistry()` — dependency injection without DI frameworks.
6. **Composite config:** defaults → user file → environment variables → runtime overrides (persisted on slash command changes).

## Common Development Tasks

```bash
# After editing the agent engine:
node --import tsx --test test/engine-system-prompt.test.ts test/engine-tool-result.test.ts

# After editing TUI state/components:
node --import tsx --test test/tui-ink-reducer.test.ts test/tui-ink-reducer-redesign.test.ts

# After editing the provider layer:
node --import tsx --test test/anthropic-provider.test.ts test/anthropic-tool-names.test.ts

# After editing permissions:
node --import tsx --test test/permissions-checker.test.ts

# Full suite before pushing:
npm run typecheck && npm test
```

## Known Gaps (as of v1.6.0)

- `npm run lint` currently aliases `tsc --noEmit`; ESLint is configured (`.eslintrc.json`) but not wired up.
- `npm run test:desktop` points at a `test/desktop/` directory that does not exist yet.
- Test coverage is concentrated in core/engine/api/TUI modules; tools, memory, skills, plugins, channels, and sandbox have no tests.
- CI runs on Node 18/ubuntu only and marks both test steps `continue-on-error: true` (failures do not fail the build).
- `examples/` still references the pre-1.5 `src/hub` API, which was removed in the architecture rewrite; example code does not run.
- Improvement roadmap: `docs/superpowers/specs/2026-08-21-v1.7-quality-and-features-spec.md`.
