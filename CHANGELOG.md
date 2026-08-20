# Changelog

All notable changes to Hax Agent CLI will be documented in this file.

## [1.6.0] - 2026-06-30

### Changed
- **TypeScript + ESM migration (full repo):** All `src/` modules converted from CommonJS `.js` to TypeScript (`.ts`). `"type": "module"`, NodeNext module resolution, ES2022 target; relative imports now require the `.js` extension. `npm run build` compiles to `dist/` via `tsc`, `npm run dev` runs `src/cli.ts` directly via tsx, and `bin`/`main` point to `dist/`.
- **Ink TUI is now the default interface:** Three-part layout (Static history + live area + input), slash command palette, Select-based approval menus, rich status bar with context usage visualization, and tool call folding with Ctrl+R detail toggle. Display components use native ink elements for correct CJK alignment. The classic readline interface remains available via `--legacy`, `--no-ink`, or `HAXAGENT_LEGACY=1`; `--ink` is kept as a no-op for compatibility.
- **Default models updated across providers:** Anthropic default is now `claude-sonnet-4-6`; profile defaults refreshed (opus, haiku, gemini, deepseek, qwen, glm, openrouter).

### Added
- `src/tui-ink/` module: `App.tsx` root component, pure `reducer.ts` state machine, keybindings, markdown rendering, slash completions; components (`CommandPalette`, `DiffView`, `ConversationTurn`, `StatusBar`, `ApprovalPrompt`, `ToolCall`, `ToolList`, `ThinkingBlock`, `TextStream`); `ui/` primitives (`Select` with pure navigation logic, `Separator`, `useTerminalSize`).
- Consolidated subsystem modules: `swarm/` (multi-agent collaboration with subprocess and in-process backends, mailbox, worktree isolation), `channels/` (IM adapters: Feishu, Slack, Discord, Telegram, DingTalk, Email, Matrix, WeChat, QQ, WhatsApp, MoChat), `sandbox/` (Docker, bubblewrap, macOS, Windows backends with path validation), `tasks/` (background task manager), `keybindings/`, `auth/`, `bridge/`, `autopilot/`, `coordinator/`, `personalization/`, `output-styles/`, `vim/`, `voice/`.
- Extended slash commands: `/session`, `/continue`, `/summary`, `/rewind`, `/mcp`, `/plugin`, `/agents`, `/dream`, `/diff`, `/branch`, `/keybindings`, `/feedback`, `/files`, `/turns`, `/release-notes`, `/reload`, `/tag`, `/tags`.
- `web.search` tool works out of the box via DuckDuckGo (no API key needed).
- `--sandbox` CLI flag and sandbox settings block (`backend`, `image`, `network`, `cpus`, `memory`) for shell command isolation.

### Fixed
- Anthropic tool names containing dots are sanitized before API calls (previously broke tool calls entirely: tools reported as nonexistent with missing parameters).

## [1.5.4] - 2026-05-30

### Changed
- **Architecture Rewrite (Complete):** Fully rebuilt following OpenHarness reference architecture and industry-grade standards. Flat src (~560 files) → layered modular (144 consolidated JS files). All 30+ OpenHarness subsystems covered with full feature parity.

## [1.5.3] - 2026-05-15

### Changed
- **Architecture Rewrite:** Migrated from flat `src/` structure to layered modular architecture inspired by OpenHarness.
  - New `core/` layer: typed messages (`StandardMessage`, `ContentBlock`), provider adapter protocol (`ApiStreamEvent` types), permission checker.
  - New `engine/` layer: `AgentEngine` with async generator tool loop, `QueryContext` for state tracking, `Session`, `HookExecutor`.
  - Consolidated provider layer: 12+ providers in `api/provider.js` (Anthropic, OpenAI, DeepSeek, Groq, Mistral, Google, Moonshot, Zhipu, DashScope, Ollama, vLLM, OpenRouter).
  - Consolidated tool registry: all 10 built-in tools in `tools/registry.js` with `isReadOnly` classification.
  - New `services/`: LSP code navigation, MCP integration, personalization, AutoDream goal continuation.
  - New `config/profiles.js`: Provider profile management with pre-configured profiles and runtime switching.
  - New `tui/`: Terminal UI with alt-screen buffer and event-driven rendering.
- Updated all documentation (CLAUDE.md, README.md, README.en.md) to reflect new architecture.

### Added
- `/lsp` command: go-to-definition and workspace symbol search.
- `/theme` command: switch terminal color themes at runtime.
- `/providers` command: list all 12+ available AI providers.
- `/personalize` command: extract environment rules from conversations.
- `/plan` command: toggle Plan mode (block all mutating tools).
- `/fullauto` command: toggle Full Auto mode (silent auto-approve).
- `/perms` command: show detailed permission status.
- `/export` command: export session to JSON file.
- `/api-key <provider> <key>` command: set per-provider API keys.
- Provider profiles: pre-configured claude, gpt, sonnet, haiku, gpt-mini, local profiles.

### Removed
- Legacy flat modules superseded by layered architecture (agent-engine, session, renderer, hub, batch, config, permissions, undo-stack, context-compaction, context-window, config-presets, init-wizard, updater, debug, i18n, and ~70 legacy subsystem directories).

## [1.4.0] - 2026-05-15

### Added
- `/context` command and `/cache` alias to view and tune context cache budgets.
- Tab completion for context cache subcommands (`status`, `window`, `reserve`, `chars-per-token`, `auto`, `on`, `off`).
- Broader model context-window inference for GPT-5, Claude 4.x, Gemini, DeepSeek, Qwen, Kimi, GLM, Doubao, Hunyuan, MiniMax, Yi, and Baichuan model IDs.
- Token-count context meter now shows sub-percent usage and input-budget counts.

### Changed
- YOLO/full permission mode now maps consistently across CLI and desktop flows.
- Shell execution no longer uses a hard allowlist gate; normal mode asks for permission and YOLO mode auto-approves.
- CLI status line is rendered outside chat history so workspace/model/context metadata stays visible without becoming user input.
- Windows shell command resolution now prefers executable shims and wraps `.cmd`/`.bat` launchers correctly.

### Fixed
- Repeated empty tool preamble loops from OpenAI-compatible providers.
- `shell.run` spinner flooding the terminal while commands are running.
- Windows `npm` execution failures caused by spawning extensionless npm shims.
- Desktop permission mode and approval flow edge cases.

## [1.3.14] - 2026-05-14

### Fixed
- Desktop i18n path corrections and component updates
- CLI paste detection multi-line splitting
- Desktop test i18n injection missing
- 7 HIGH severity security issues (path traversal hardening, env var sanitization)
- ReDoS protection added to regex patterns in file.search
- LICENSE file and .gitignore hardening

## [1.3.13] - 2026-05-14

### Added
- `stock.quote` tool for real-time stock/index quotes (A-shares, HK stocks, US stocks)
- `file.read` auto-truncation + `offset`/`limit` line pagination for large files
- Inline diff view for `file.edit` tool
- Context window usage meter in status line
- Multi-line input support (`\` line continuation)
- Ctrl+←/→ word jump navigation, status bar cwd, `/help` shortcut cheatsheet
- Ctrl+R reverse history search
- Command syntax highlighting and improved error messages
- Session file change summary on `/exit`
- `/copy` command to copy last AI response to clipboard
- `/rename` command to name the current session
- Tool execution timing displayed in file modification notices
- `-v` shorthand for `--version` flag

### Changed
- Local tools modularized into separate files (`file-edit`, `file-readdir`, `file-delete`, `web-fetch`, `web-search`, `stock-quote`)
- Enhanced tab autocomplete with first-run onboarding
- Enhanced `/clear` with cleared message count and user guidance
- `file.read`/`file.search`/`file.write` tool descriptions discourage AI from passing tiny `maxBytes`
- Increased empty tool preamble retry tolerance (1→3) with stronger continuation prompts

### Fixed
- Google provider dependency: `require("@google/genai")` → `@google/generative-ai`
- Stream `finalMessage()` returning `null` on non-Anthropic endpoints
- Chinese text falsely detected as tool preamble in `forceTextResponse` mode
- Tab autocomplete not working due to readline inserting `\t` before keypress event

## [1.3.12] - 2026-05-14

### Added
- Quick setup mode in init wizard: skip optional questions with recommended defaults
- `hax-agent config` command to view current configuration
- `hax-agent config edit` to open config file in default editor
- Smart API key detection during init (auto-detects env vars to skip input)
- `hax-agent --version` / `-V` flag to print version number
- `hax-agent doctor` command for one-line diagnostics
- `--no-color` flag to disable ANSI terminal output

### Changed
- Init wizard flow: quick mode asks only Provider + Key, full mode keeps all 9 questions
- CLI better supports piping and non-TTY environments for config viewing
- Improved i18n for Chinese (zh-CN) translations

## [1.3.11] - 2026-05-07

### Added
- Desktop approval dialog for tool permissions
- Workspace search in desktop app
- Git diff/review tools in desktop app

### Changed
- Refactored shared serialization utilities to `src/utils/serialization.js`
- Improved web-search with Bing RSS fallback when DuckDuckGo fails
- Updated User-Agent version string

## [1.3.10] - 2026-05-05

### Changed
- Improved context handling and tool-call recovery
- Better error messages for empty tool preamble detection

## [1.3.9] - 2026-04-28

### Changed
- Modularized tools into separate files
- Unified DSML tool call parsing across providers

## [1.3.8] - 2026-04-20

### Added
- Desktop app renderer with Vue.js components
- File tree, chat area, sidebar, and settings UI

## [1.3.7] - 2026-04-15

### Added
- Command suggestions for slash and CLI commands
- Typo-tolerant command matching

## [1.3.6] - 2026-04-10

### Added
- Interactive initialization wizard (`hax-agent init`)
- Enhanced permission management system

## [1.3.5] - 2026-04-05

### Added
- CI/CD workflow for automated testing and npm publishing
- Manual release trigger via `workflow_dispatch`

## [1.3.4] - 2026-03-30

### Added
- Self-update check and installation (`/update` command)

### Changed
- Refactored provider code for better maintainability
- Raised tool execution limits

## [1.3.3] - 2026-03-25

### Added
- Shell command execution with allowlist control
- Improved terminal UI rendering

## [1.3.2] - 2026-03-20

### Fixed
- CLI test suite failures
- Various bug fixes and refactoring

## [1.3.1] - 2026-03-15

### Added
- Permission management system
- DeepSeek DSML format support
- `reasoning_content` handling for extended thinking models

### Fixed
- Single-call tool deduplication
- Recursive web.fetch loop prevention
- Windows shell spawn issues
