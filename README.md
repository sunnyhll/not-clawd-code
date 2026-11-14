# not_clawd_code

`not_clawd_code` is a reconstructed local build of the publicly exposed Claude Code source snapshot from March 31, 2026. The original public leak exposed an incomplete `src/` tree. This repository was later rebuilt into a runnable Bun project by restoring missing root files, adding compatibility shims, reconstructing missing source/runtime pieces, and patching runtime gaps.

This is not an official Anthropic release.

## Current Status

The current local tree is able to start the interactive CLI again.

Verified in this workspace:

```bash
bun install
bun run version
bun run dev
bun run build
```

Current package metadata:

- `name`: `not_clawd_code`
- `version`: `1.0.24`
- `packageManager`: `bun@1.3.5`
- `engines.bun`: `>=1.1.0`

Verified local Bun version:

- `bun 1.3.11`

Interactive proof:

![Interactive not_clawd_code session](./media/image.png)

## What Was Reconstructed

The leaked source snapshot was not originally a complete runnable repository. This working tree now includes:

- restored root build files such as `package.json`, `tsconfig.json`, `bunfig.toml`, `bun.lock`, and `scripts/`
- reconstructed source/runtime pieces needed to complete the partial snapshot
- local compatibility packages under `shims/` for internal or unavailable modules
- extra runtime/vendor material under `vendor/`
- compatibility patches needed to make the interactive CLI start again under Bun

As checked in this workspace:

- `src/` currently contains `2047` files
- `node_modules` is symlinked to `/home/jovyan/shared/claude-code-build/node_modules`
- `dist` is symlinked to `/home/jovyan/shared/claude-code-build/dist`

## Project Structure

Current top-level layout in this reconstructed tree:

```text
.
├── src/                   # Restored TypeScript source tree
├── scripts/               # Build, packaging, install, and test helpers
├── shims/                 # Local compatibility packages for missing/internal modules
├── vendor/                # Native/runtime support sources
├── media/                 # Local media assets
├── package.json           # Package metadata and scripts
├── bun.lock               # Bun lockfile
├── bunfig.toml            # Bun preload configuration
├── tsconfig.json          # TypeScript config
```

Important source areas:

```text
src/
├── entrypoints/           # CLI bootstraps and SDK entrypoints
├── main.tsx               # Main CLI setup path
├── dev-entry.ts           # Reconstructed interactive dev launcher
├── QueryEngine.ts         # Core query/runtime loop
├── commands/              # Slash command implementations
├── tools/                 # Tool implementations
├── components/            # React + Ink terminal UI
├── services/              # Auth, MCP, telemetry, config, compact, etc.
├── bridge/                # IDE / remote-control bridge logic
├── coordinator/           # Multi-agent orchestration
├── tasks/                 # Task system
├── skills/                # Bundled skill logic
├── plugins/               # Plugin support
├── server/                # Server-side / IDE service entrypoints
├── remote/                # Remote session features
├── memdir/                # Persistent memory features
├── keybindings/           # Keyboard handling
├── vim/                   # Vim mode
├── ink/                   # Ink renderer integration
├── utils/                 # Shared helpers
└── types/                 # Shared types
```

## Toolchain And Runtime

This repo is a Bun-first TypeScript CLI project.

- entrypoint: `src/entrypoints/cli.tsx`
- interactive dev entry: `src/dev-entry.ts`
- Bun preload config: `bunfig.toml` preloads `scripts/bun-plugin-shims.ts`
- build output: `dist/cli.mjs`

Important runtime pieces in the current reconstruction:

- `react ^19.0.0`
- `react-reconciler ^0.33.0`
- `ink ^6.8.0`
- local file dependencies for internal modules such as:
  - `@ant/claude-for-chrome-mcp`
  - `@ant/computer-use-input`
  - `@ant/computer-use-mcp`
  - `@ant/computer-use-swift`
  - `color-diff-napi`
  - `modifiers-napi`
  - `url-handler-napi`

## Install And Run

From the repo root:

```bash
bun install
bun run version
bun run dev
```

Other useful commands:

```bash
bun run build
bun run start
bun run check
bun run test
```

In this workspace there is also an external helper launcher:

```bash
not-claude
```

That launcher is only a convenience wrapper around this repo. The repo-native command is still:

```bash
bun run dev
```

## Research Context

This repository exists for educational use, defensive security research, software supply-chain analysis, and architecture study. It should not be treated as an official Anthropic repository or an authoritative release artifact.

The current local state is a reconstruction. It is usable, but it may still differ from Anthropic's original internal repository in structure, history, packaging, and exact runtime behavior.

## Maintainer

- Yaswanth-ampolu
- `ampoluyaswanth2002@gmail.com`

# architecture

```mermaid
flowchart TB
    subgraph UI["终端 UI 层 (React + Ink)"]
        REPL[REPL 主界面] --> INPUT[输入框/命令面板]
        UIHOOKS[UI 钩子 useInput/useAppState]
    end

    subgraph ENTRY["入口层"]
        CLI[cli.tsx 参数解析] --> MODE{模式分发}
        MODE -->|交互式| REPL
        MODE -->|headless/-p| SDK[SDK 单次执行]
    end

    subgraph CORE["Agent 核心"]
        QE[QueryEngine<br/>会话编排/中断/持久化] --> LOOP[query 主循环<br/>agentic loop]
        LOOP --> PERM[权限系统<br/>canUseTool / PermissionMode]
        PERM --> TOOLREG[工具注册表 tools.ts]
        TOOLREG --> TOOLEXEC[Tool.call 执行器]
        LOOP --> COMPACT[上下文压缩 compact]
    end

    subgraph TOOLS["工具系统 (50+ 工具)"]
        FILE[文件类<br/>Read/Write/Edit/Glob/Grep]
        SHELL[执行类<br/>Bash/PowerShell]
        WEB[网络类<br/>WebFetch/WebSearch]
        META[元工具<br/>AgentTool/TodoWrite/Task]
        MCPT[MCPTool 透传外部工具]
    end

    subgraph SERVICES["服务层"]
        LLM[LLM API 客户端<br/>流式/重试/用量统计]
        MCP[MCP 客户端<br/>连接外部 server]
        AUTH[OAuth 认证]
        MEM[记忆/会话存储<br/>transcript/CLAUDE.md]
        TELEM[遥测/成本追踪]
    end

    subgraph EXT["扩展机制"]
        HOOKS[生命周期 Hooks<br/>PreToolUse/PostToolUse]
        SKILLS[Skills/Plugins]
        TASKS[后台任务系统]
    end

    REPL -- 用户输入/命令 --> QE
    SDK --> QE
    QE -- history + systemPrompt --> LLM
    LLM -- tool_use 块 --> LOOP
    TOOLEXEC --> FILE & SHELL & WEB & META & MCPT
    TOOLEXEC --> HOOKS
    MCPT <--> MCP
    TOOLREG -.注册.-> SKILLS
    LOOP -- tool_result 回传 LLM --> LLM
    QE --> MEM
    TELEM -.观测.-> LOOP


```

```mermaid
classDiagram
    class Tool {
        <<interface>>
        +inputSchema: ZodSchema
        +call(args, context): Promise~ToolResult~
        +description(input): Promise~string~
        +isEnabled() bool
        +isReadOnly(input) bool
        +isConcurrencySafe(input) bool
    }
    class QueryEngine {
        +queryLoop()
        +processUserInput()
        +processToolUse()
        +abortController
    }
    class ToolUseContext {
        +cwd: string
        +fileStateCache: FileStateCache
        +abortController: AbortController
        +messageHistory
    }
    class CanUseToolFn {
        <<interface>>
        +invoke(tool, input): PermissionResult
    }
    class LLMClient {
        +createMessage() AsyncStream
        +retry/backoff()
        +accumulateUsage()
    }
    class Command {
        <<interface>>
        +name
        +call(args, context)
    }
    class MCPServer {
        +listTools()
        +callTool(name, args)
    }
    QueryEngine --> LLMClient : 发送对话
    QueryEngine --> Tool : tool_use 分发
    QueryEngine --> Command : 斜杠命令
    Tool ..> ToolUseContext : 依赖
    Tool ..> CanUseToolFn : 权限关卡
    MCPServer ..|> Tool : 动态适配为工具


```