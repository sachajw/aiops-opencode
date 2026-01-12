# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenCode is an open-source AI coding agent with a TUI, desktop app, and web interface. It features a client/server architecture with LSP integration, MCP server support, and multiple LLM providers.

## Development Commands

### Core Development
```bash
# Install dependencies
bun install

# Run OpenCode in dev mode (defaults to packages/opencode directory)
bun dev

# Run against a specific directory
bun dev <directory>

# Run against the repo root
bun dev .

# Build standalone executable
./packages/opencode/script/build.ts --single

# Run built executable
./packages/opencode/dist/opencode-<platform>/bin/opencode
```

### Testing
```bash
# Run tests in opencode package
bun run --cwd packages/opencode test

# Type checking
bun typecheck  # runs turbo typecheck across all packages
```

### UI Development
```bash
# Web app (SolidJS) - for testing UI changes
bun run --cwd packages/app dev
# Opens at http://localhost:5173

# Desktop app (Tauri) - native wrapper
bun run --cwd packages/desktop tauri dev
# Opens native window with dev server at http://localhost:1420

# Desktop build
bun run --cwd packages/desktop tauri build
```

### SDK and API Changes
```bash
# Regenerate SDK after API/server changes
./script/generate.ts

# Regenerate JavaScript SDK specifically
./packages/sdk/js/script/build.ts
```

## Architecture

### Monorepo Structure
- **Turborepo** with Bun workspaces
- **Main branch**: `dev` (not `main`)
- **Package manager**: Bun 1.3+

### Core Packages

#### `packages/opencode` - Core Server & CLI
The heart of OpenCode containing:
- **`src/agent/`** - Agent system with prompts for exploration, summarization, title generation
- **`src/server/`** - Hono-based HTTP server exposing REST API and WebSocket endpoints
- **`src/session/`** - Session management, message handling, LLM interaction, prompt building, compaction, retry logic
- **`src/tool/`** - Tool implementations (Bash, Edit, Read, Write, Glob, Grep, LSP, WebFetch, etc.)
- **`src/cli/`** - CLI commands and TUI implementation
  - **`src/cli/cmd/tui/`** - Terminal UI built with SolidJS and OpenTUI
- **`src/lsp/`** - Language Server Protocol integration for code intelligence
- **`src/mcp/`** - Model Context Protocol server support
- **`src/provider/`** - Multi-provider LLM support (Anthropic, OpenAI, Google, etc.)
- **`src/file/`** - File operations and ripgrep integration
- **`src/config/`** - Configuration management
- **`src/project/`** - Project and workspace management, VCS integration

#### `packages/app` - Shared Web UI
SolidJS components used by both web and desktop apps.

#### `packages/desktop` - Native Desktop App
Tauri-based native application wrapping `packages/app`.

#### `packages/plugin` - Plugin System
Source for `@opencode-ai/plugin` package.

#### `packages/sdk/js` - JavaScript SDK
Generated SDK for interacting with OpenCode server.

#### Other Packages
- `packages/console` - Console-related functionality
- `packages/enterprise` - Enterprise features
- `packages/function` - Serverless functions
- `packages/identity` - Auth and identity
- `packages/script` - Build and utility scripts
- `packages/slack` - Slack integration
- `packages/ui` - UI components library
- `packages/util` - Shared utilities
- `packages/web` - Web-specific code

### Key Architecture Patterns

#### Client/Server Model
OpenCode uses a client/server architecture where:
- Server can run standalone (`opencode serve`)
- TUI/desktop apps connect to server via HTTP/WebSocket
- Enables remote access (e.g., mobile app driving desktop instance)

#### Session Management
- Sessions are the core abstraction for AI interactions
- Each session maintains conversation history, agent state, and tool contexts
- Message processing in `session/processor.ts`
- Compaction handled by `session/compaction.ts` to manage context limits

#### Tool System
- Tools defined in `src/tool/` with `.ts` implementation and `.txt` prompt files
- Tool registry in `src/tool/registry.ts`
- Each tool has a schema and execution handler
- Tools include file operations, LSP queries, bash execution, web fetching, etc.

#### Agent System
- Two built-in agents: **build** (full access) and **plan** (read-only)
- **general** subagent for complex searches and multistep tasks
- Agent prompts in `src/agent/prompt/`
- Switch agents with Tab key in TUI

#### LSP Integration
- Out-of-the-box Language Server Protocol support
- Client/server management in `src/lsp/`
- Provides code intelligence (definitions, references, diagnostics)

#### MCP Server Support
- Model Context Protocol implementation in `src/mcp/`
- Extends capabilities with external context providers

#### Event Bus
- Global event bus in `src/bus/` for inter-component communication
- Used for coordinating between server, sessions, and UI

## Style Guide

Follow these conventions (from STYLE_GUIDE.md):

- **Functions**: Keep logic in one function unless composable/reusable
- **Destructuring**: Avoid unnecessary destructuring; use `obj.a` instead of `const { a } = obj`
- **Control flow**: Prefer early returns over `else` statements
- **Error handling**: Prefer `.catch(...)` over `try`/`catch` when possible
- **Types**: Use precise types, avoid `any`
- **Variables**: Prefer `const` over `let`, use immutable patterns
- **Naming**: Prefer single-word identifiers when descriptive
- **Runtime**: Use Bun APIs like `Bun.file()` when applicable

## Pull Request Guidelines

### Requirements
- All PRs must reference an existing issue (use `Fixes #123` or `Closes #123`)
- Keep PRs small and focused
- UI changes require screenshots/videos
- Logic changes need explanation of how you verified it works

### PR Title Format
Follow conventional commits:
- `feat:` - new feature
- `fix:` - bug fix
- `docs:` - documentation
- `chore:` - maintenance
- `refactor:` - code refactoring
- `test:` - test changes

Optional scope: `feat(app):`, `fix(desktop):`, `chore(opencode):`

### What Gets Merged
- Bug fixes
- Additional LSPs/Formatters
- LLM performance improvements
- New provider support
- Environment-specific fixes
- Missing standard behavior
- Documentation improvements

**Note**: UI or core product features require design review before implementation.

## Debugging

### Bun Debugging Tips
Most reliable method: Run manually with `bun run --inspect=<url> dev ...` and attach debugger.

#### For TUI + Server Breakpoints
```bash
bun dev spawn
```

#### Separate Server Debugging
```bash
# Terminal 1: Debug server
bun run --inspect=ws://localhost:6499/ ./src/index.ts serve --port 4096

# Terminal 2: Attach TUI
opencode attach http://localhost:4096
```

#### VSCode Setup
See `.vscode/settings.example.json` and `.vscode/launch.example.json` for configurations.

**Tip**: Export `BUN_OPTIONS=--inspect=ws://localhost:6499/` to avoid repeating `--inspect` flag.

## Important Notes

- Default branch is `dev`, not `main`
- Run `./script/generate.ts` after API/server changes to regenerate SDK
- Desktop app requires Rust toolchain and Tauri prerequisites
- Never use `git` commands with `-i` flag (interactive mode unsupported)
- LSP integration provides built-in code intelligence
- Multi-provider support allows using Claude, OpenAI, Google, or local models
