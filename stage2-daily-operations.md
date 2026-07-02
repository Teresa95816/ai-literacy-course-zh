# 第二階段, 日常操作與個人化設定

熟悉互動模式, 常用指令與個人偏好設定.

---

# 1. Interactive mode

> Complete reference for keyboard shortcuts, input modes, and interactive features in Claude Code sessions.

## Keyboard shortcuts

### General controls

| Shortcut | Description |
|---|---|
| `Ctrl+C` | Interrupt, or clear input. First press clears the prompt, second press exits |
| `Ctrl+D` | Exit Claude Code session (EOF signal) |
| `Ctrl+G` or `Ctrl+X Ctrl+E` | Open prompt in default text editor |
| `Ctrl+L` | Redraw screen, keeping input and history |
| `Ctrl+O` | Toggle transcript viewer, shows detailed tool usage |
| `Ctrl+R` | Reverse search command history |
| `Ctrl+V` / `Cmd+V` (iTerm2) / `Alt+V` (Windows/WSL) | Paste image from clipboard |
| `Ctrl+B` | Background running Bash commands and agents |
| `Ctrl+T` | Toggle task list |
| `Esc` | Interrupt Claude, stop current response or tool call |
| `Esc` + `Esc` | Clear input draft, or open the rewind menu when input is empty |
| `Shift+Tab` | Cycle permission modes: default, acceptEdits, plan, and any enabled optional modes |
| `Option+P` (macOS) / `Alt+P` | Switch model |
| `Option+T` (macOS) / `Alt+T` | Toggle extended thinking |
| `Option+O` (macOS) / `Alt+O` | Toggle fast mode |

### Text editing

| Shortcut | Description |
|---|---|
| `Ctrl+A` / `Ctrl+E` | Move cursor to start / end of current line |
| `Ctrl+K` | Delete to end of line |
| `Ctrl+U` | Delete from cursor to line start |
| `Ctrl+W` | Delete previous word |
| `Ctrl+Y` | Paste deleted text |
| `Alt+B` / `Alt+F` | Move cursor back / forward one word |

### Multiline input

| Method | Shortcut | Context |
|---|---|---|
| Quick escape | `\` + `Enter` | Works in all terminals |
| Option key | `Option+Enter` | After enabling Option as Meta on macOS |
| Shift+Enter | `Shift+Enter` | Native in iTerm2, WezTerm, Ghostty, Kitty, Warp, Apple Terminal, Windows Terminal |
| Control sequence | `Ctrl+J` | Works in any terminal without configuration |
| Paste mode | Paste directly | For code blocks, logs |

### Quick commands

| Shortcut | Description |
|---|---|
| `/` at start | Command or skill |
| `!` at start | Shell mode, runs a command directly and adds output to session |
| `@` | File path mention, triggers autocomplete |

## Vim editor mode

Enable via `/config` → Editor mode. Mode switching: `Esc` enters NORMAL mode, `i`/`I`/`a`/`A` insert, `o`/`O` open line below/above, `v`/`V` visual selection. Navigation uses `hjkl`, `w`/`e`/`b` for word motion, `0`/`$`/`^` for line position, `gg`/`G` for start/end of input. Editing uses `x`, `dd`, `D`, `cc`, `yy`, `p`/`P`, `u` for undo.

## Command history

Claude Code maintains command history per working directory. History resets on `/clear`. Reverse search with `Ctrl+R`: type a query, press `Ctrl+R` again to cycle matches, `Ctrl+S` to change scope (session/project/all projects), `Tab`/`Esc` to accept, `Enter` to execute immediately.

## Background Bash commands

Claude Code can run Bash commands in the background: prompt Claude to run in background, or press `Ctrl+B` to move a running command to background. Output is written to a file and retrievable via the Read tool. Background tasks are auto-terminated if output exceeds 5GB.

Shell mode with `!` prefix runs commands directly without Claude's approval, adds command and output to context, and supports `Ctrl+B` backgrounding and history-based autocomplete.

## Side questions with /btw

Use `/btw` to ask a quick question about current work without adding to conversation history. Has full visibility into context but no tool access, single response only. Press `f` to fork the exchange into its own session with full tool access.

## Task list

Claude creates a task list to track progress on complex work. Press `Ctrl+T` to toggle the view. Tasks persist across context compactions.

## Session recap

Claude Code shows a one-line recap when you return after stepping away (3+ minutes idle, 3+ turns in session). Run `/recap` to generate on demand.

---

# 2. Common workflows

> Step-by-step guides for exploring codebases, fixing bugs, refactoring, testing, and other everyday tasks with Claude Code.

## Understand new codebases

**Get a quick overview**: navigate to project root, start `claude`, ask "give me an overview of this codebase", then narrow down with "explain the main architecture patterns used here" or "how is authentication handled?"

**Find relevant code**: ask Claude to find relevant files ("find the files that handle user authentication"), then ask how components interact and trace the execution flow.

## Fix bugs efficiently

Share the error with Claude ("I'm seeing an error when I run npm test"), ask for fix recommendations, then apply the fix. Tell Claude the reproduction command and whether the error is intermittent or consistent.

## Refactor code

Identify legacy code ("find deprecated API usage"), get refactoring recommendations, apply changes safely while maintaining behavior, then verify with tests. Do refactoring in small, testable increments.

## Work with tests

Identify untested code, generate test scaffolding, add meaningful test cases for edge conditions, then run and verify. Claude examines existing test files to match style, frameworks, and assertion patterns.

## Create pull requests

Ask Claude directly ("create a pr for my changes") or step through: summarize changes, generate a PR, then review and refine the description. When you create a PR with `gh pr create`, the session links to it automatically; return later with `claude --from-pr <number>`.

## Handle documentation

Identify undocumented code, generate documentation (JSDoc, docstrings, etc.), review and enhance with context and examples, then verify it follows project standards.

## Work with images

Drag and drop, paste with `Ctrl+V`, or provide a path. Ask Claude to analyze, describe UI elements, or generate CSS/HTML matching a design mockup.

## Reference files and directories

Use `@` to include files or directories without waiting for Claude to read them. `@file1.js` includes full content; `@src/components` gives a directory listing; `@server:resource` fetches MCP resources.

## Run Claude on a schedule

| Option | Where it runs | Best for |
|---|---|---|
| Routines | Anthropic-managed infrastructure | Tasks that run even when your computer is off |
| Desktop scheduled tasks | Your machine via desktop app | Tasks needing local files or uncommitted changes |
| GitHub Actions | Your CI pipeline | Tasks tied to repo events or cron schedules |
| `/loop` | Current CLI session | Quick polling while a session is open |

## Resume previous conversations

```bash
claude --continue
```
Resumes the most recent session in the current directory. Use `claude --resume` to choose from a list, or `/resume` inside a running session.

## Run parallel sessions with worktrees

```bash
claude --worktree feature-auth
```
Each worktree is a separate checkout on its own branch, so edits don't collide.

## Plan before editing

```bash
claude --permission-mode plan
```
Claude reads files and proposes a plan but makes no edits until approved. Press `Shift+Tab` mid-session to toggle.

## Delegate research to subagents

"use a subagent to investigate how our auth system handles token refresh." The subagent reads files in its own context window and reports a summary.

## Pipe Claude into scripts

```bash
git log --oneline -20 | claude -p "summarize these recent commits"
```

---

# 3. Commands

> Complete reference for commands available in Claude Code, including built-in commands and bundled skills.

Type `/` to see every command, or `/` followed by letters to filter. A command is only recognized at the start of your message.

## Commands across a typical workflow

**First session in a repo**: `/init` generates a starter CLAUDE.md, `/memory` refines it, `/mcp` and `/agents` set up servers or subagents, `/permissions` sets approval rules.

**During a task**: `/plan` switches into plan mode, `/model` and `/effort` adjust reasoning, `/context` shows context usage, `/compact` summarizes it, `/btw` handles a quick aside.

**Running work in parallel**: `/agents` manages subagents, `/tasks` lists background work, `/background` detaches the session, `/batch` decomposes a large change across worktrees.

**Before you ship**: `/diff` shows changes, `/code-review` checks for bugs and cleanups (`--fix` applies findings, `--comment` posts inline), `/review` reviews a GitHub PR, `/security-review` gives a deeper pass, `/code-review ultra` runs a cloud multi-agent review.

**Between sessions**: `/clear` starts fresh keeping project memory, `/resume` and `/branch` return to or fork a conversation, `/teleport` pulls a web session into the terminal, `/remote-control` continues a local session from another device.

**When something is wrong**: `/rewind` rolls back to a checkpoint, `/doctor` and `/debug` diagnose issues, `/feedback` reports a bug with session context.

## Key commands reference

| Command | Purpose |
|---|---|
| `/add-dir <path>` | Add a working directory for file access during the current session |
| `/agents` | Manage agent configurations |
| `/background [prompt]` | Detach the current session to run as a background agent |
| `/batch <instruction>` | Orchestrate large-scale changes in parallel across isolated worktrees |
| `/branch [name]` | Create a branch of the current conversation |
| `/clear [name]` | Start a new conversation with empty context, previous stays in `/resume` |
| `/code-review [level] [--fix] [--comment] [target]` | Review the current diff for bugs and cleanups |
| `/compact [instructions]` | Free up context by summarizing the conversation |
| `/config [key=value]` | Open the Settings interface, or set a key directly |
| `/context [all]` | Visualize current context usage as a colored grid |
| `/effort [level|auto]` | Set the model effort level |
| `/goal [condition|clear]` | Keep Claude working across turns until a condition is met |
| `/hooks` | View hook configurations for tool events |
| `/init` | Initialize project with a CLAUDE.md guide |
| `/mcp` | Manage MCP server connections and OAuth |
| `/memory` | Edit CLAUDE.md files, toggle auto memory |
| `/model [model]` | Switch the AI model, adjust effort level |
| `/permissions` | Manage allow, ask, and deny rules for tool permissions |
| `/plan [description]` | Enter plan mode directly from the prompt |
| `/resume [session]` | Resume a conversation by ID or name |
| `/rewind` | Rewind conversation and/or code to a previous point |
| `/schedule [description]` | Create, update, list, or run routines |
| `/skills` | List available skills |
| `/statusline` | Configure the status line |
| `/tasks` | View and manage everything running in the background |
| `/usage` | Show session cost, plan usage limits, and activity stats |

## MCP prompts

MCP servers can expose prompts that appear as commands, using the format `/mcp__<server>__<prompt>`, dynamically discovered from connected servers.

---

# 4. CLI reference

> Complete reference for Claude Code command-line interface, including commands and flags.

## CLI commands

| Command | Description |
|---|---|
| `claude` | Start interactive session |
| `claude "query"` | Start interactive session with initial prompt |
| `claude -p "query"` | Query via SDK, then exit |
| `cat file \| claude -p "query"` | Process piped content |
| `claude -c` | Continue most recent conversation in current directory |
| `claude -r "<session>" "query"` | Resume session by ID or name |
| `claude update` | Update to latest version |
| `claude agents` | Open agent view to monitor and dispatch parallel background sessions |
| `claude attach <id>` | Attach to a background session in this terminal |
| `claude mcp` | Configure Model Context Protocol servers |
| `claude setup-token` | Generate a long-lived OAuth token for CI and scripts |
| `claude project purge [path]` | Delete all local Claude Code state for a project |

If you mistype a subcommand, Claude Code suggests the closest match, e.g. `claude udpate` prints "Did you mean claude update?".

## Key CLI flags

| Flag | Description |
|---|---|
| `--add-dir` | Add additional working directories for Claude to read and edit files |
| `--agent` | Specify an agent for the current session |
| `--bare` | Minimal mode, skip auto-discovery of hooks, skills, plugins, MCP servers, auto memory, CLAUDE.md |
| `--continue`, `-c` | Load the most recent conversation in the current directory |
| `--dangerously-skip-permissions` | Skip permission prompts, equivalent to `--permission-mode bypassPermissions` |
| `--effort` | Set the effort level for the current session |
| `--fallback-model` | Enable automatic fallback to specified model(s) when the primary is overloaded |
| `--model` | Sets the model for the current session |
| `--name`, `-n` | Set a display name for the session |
| `--output-format` | Specify output format for print mode: `text`, `json`, `stream-json` |
| `--permission-mode` | Begin in a specified permission mode |
| `--plugin-dir` | Load a plugin from a directory or `.zip` archive for this session only |
| `--print`, `-p` | Print response without interactive mode |
| `--remote` | Create a new web session on claude.ai with the provided task description |
| `--resume`, `-r` | Resume a specific session by ID or name, or show an interactive picker |
| `--safe-mode` | Start with all customizations disabled to troubleshoot a broken configuration |
| `--system-prompt` / `--append-system-prompt` | Replace or append to the default system prompt |
| `--verbose` | Enable verbose logging, shows full turn-by-turn output |
| `--worktree`, `-w` | Start Claude in an isolated git worktree |

### System prompt flags

Four flags customize the system prompt: `--system-prompt` (replace entirely), `--system-prompt-file` (replace with file contents), `--append-system-prompt` (append text), `--append-system-prompt-file` (append file contents). Use append flags when Claude should remain a coding assistant with extra rules; use replacement flags when the surface or identity differs entirely, like a non-coding agent in an unattended pipeline.

---

# 5. Choose a permission mode

> Control whether Claude asks before editing files or running commands.

When Claude wants to edit a file, run a shell command, or make a network request, it pauses and asks for approval. Permission modes control how often that pause happens.

## Available modes

| Mode | What runs without asking | Best for |
|---|---|---|
| `default` | Reads only | Getting started, sensitive work |
| `acceptEdits` | Reads, file edits, common filesystem commands (mkdir, touch, mv, cp) | Iterating on code you're reviewing |
| `plan` | Reads only | Exploring a codebase before changing it |
| `auto` | Everything, with background safety checks | Long tasks, reducing prompt fatigue |
| `dontAsk` | Only pre-approved tools | Locked-down CI and scripts |
| `bypassPermissions` | Everything | Isolated containers and VMs only |

In every mode except `bypassPermissions`, writes to protected paths (like `.git`, `.claude`, `.vscode`) are never auto-approved.

## Switch permission modes

In the CLI, press `Shift+Tab` to cycle `default` → `acceptEdits` → `plan`. Pass `claude --permission-mode plan` at startup, or set `defaultMode` in settings.json permanently.

## acceptEdits mode

Lets Claude create and edit files without prompting. Also auto-approves `mkdir`, `touch`, `rm`, `rmdir`, `mv`, `cp`, and `sed` within the working directory.

## Plan mode

Claude researches and proposes changes without making them. Enter with `Shift+Tab` or `/plan`. When ready, Claude presents a plan and you can approve and start editing, keep planning with feedback, or refine with Ultraplan for browser-based review. Press `Ctrl+G` to edit the plan directly in your text editor.

## Auto mode

A separate classifier model reviews actions before they run, blocking anything that escalates scope, targets unrecognized infrastructure, or appears driven by hostile content. Requires Claude Code v2.1.83+, an eligible plan and model (Opus 4.6+/Sonnet 4.6+ on Anthropic API), and Owner enablement on Team/Enterprise.

**Blocked by default**: downloading and executing code, sending sensitive data externally, production deploys/migrations, mass deletion, granting IAM permissions, force push, `git reset --hard`.

**Allowed by default**: local file operations, installing declared dependencies, reading `.env` and sending to matching API, read-only HTTP requests, pushing to your own branch.

If the classifier blocks an action 3 times consecutively or 20 times total, auto mode pauses and Claude Code resumes normal prompting.

## dontAsk mode

Auto-denies every tool call that would otherwise prompt; only `permissions.allow` rules and read-only Bash commands execute. Fully non-interactive, good for CI.

## bypassPermissions mode

Disables permission prompts and safety checks entirely. Only use in isolated containers or VMs without internet access. Cannot be entered without an explicit enabling flag at startup; refuses to start as root/sudo on Linux/macOS.

---

# 6. Configure permissions

> Control what Claude Code can access and do with fine-grained permission rules, modes, and managed policies.

## Permission system

| Tool type | Example | Approval required |
|---|---|---|
| Read-only | File reads, Grep | No |
| Bash commands | Shell execution | Yes |
| File modification | Edit/write files | Yes |

## Manage permissions

`/permissions` opens an interactive UI listing rules and their source file. **Allow** rules skip approval, **Ask** rules always prompt, **Deny** rules block. Rules are evaluated deny → ask → allow, first match wins.

## Permission rule syntax

Rules follow `Tool` or `Tool(specifier)`:

| Rule | Effect |
|---|---|
| `Bash` | Matches all Bash commands |
| `Bash(npm run build)` | Matches the exact command |
| `Read(./.env)` | Matches reading the `.env` file |
| `WebFetch(domain:example.com)` | Matches fetch requests to example.com |

### Wildcard patterns

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)", "Bash(git commit *)"],
    "deny": ["Bash(git push *)"]
  }
}
```
`Bash(ls *)` matches `ls -la` but not `lsof`; `Bash(ls*)` matches both since there's no word boundary.

### Read and Edit path patterns

| Pattern | Meaning | Example |
|---|---|---|
| `//path` | Absolute path from filesystem root | `Read(//Users/alice/secrets/**)` |
| `~/path` | Path from home directory | `Read(~/Documents/*.pdf)` |
| `/path` | Path relative to project root | `Edit(/src/**/*.ts)` |
| `path` or `./path` | Path relative to current directory | `Read(*.env)` |

### WebFetch and MCP rules

`WebFetch(domain:example.com)` matches a hostname; `WebFetch(domain:*.example.com)` matches any subdomain. `mcp__puppeteer` matches any tool from that server; `mcp__puppeteer__puppeteer_navigate` matches one specific tool.

## Extend permissions with hooks

PreToolUse hooks run before the permission prompt and can deny, force a prompt, or skip it. Deny and ask rules still apply regardless of what a hook returns.

## Working directories

`--add-dir <path>` or `/add-dir` extends file access to additional directories. Most `.claude/` configuration is not discovered from these directories except skills, subagents, and a few settings keys.

## Managed settings

Organizations can deploy settings that user/project settings cannot override, delivered via MDM, managed-settings.json, server-managed settings, or a self-hosted gateway. Key managed-only settings include `allowManagedPermissionRulesOnly`, `strictKnownMarketplaces`, and `disableBypassPermissionsMode`.

## Settings precedence

1. Managed settings (highest, can't be overridden)
2. Command line arguments
3. Local project settings (`.claude/settings.local.json`)
4. Shared project settings (`.claude/settings.json`)
5. User settings (`~/.claude/settings.json`)

If a tool is denied at any level, no other level can allow it.

---

# 7. Claude Code settings

> Configure Claude Code with global and project-level settings, and environment variables.

## Configuration scopes

| Scope | Location | Who it affects | Shared with team? |
|---|---|---|---|
| Managed | Server-managed, plist/registry, or system managed-settings.json | Org members or machine users | Yes (deployed by IT) |
| User | `~/.claude/` | You, across all projects | No |
| Project | `.claude/` in repository | All collaborators | Yes (committed to git) |
| Local | `.claude/settings.local.json` | You, in this repository only | No (gitignored) |

## How scopes interact

Priority order: Managed (highest) > command line arguments > Local > Project > User (lowest). Permission rules merge across scopes instead of overriding.

## Settings files

- **User settings**: `~/.claude/settings.json`, applies to all projects
- **Project settings**: `.claude/settings.json` (checked into git) or `.claude/settings.local.json` (gitignored)
- **Managed settings**: delivered via server-managed settings, MDM/OS-level policies, or file-based `managed-settings.json` at system directories

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(npm run lint)", "Bash(npm run test *)"],
    "deny": ["Bash(curl *)", "Read(./.env)"]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1"
  }
}
```

Claude Code watches settings files and reloads most keys without a restart. A few keys, like `model` and `outputStyle`, are read once at session start.

## Key available settings

| Key | Description |
|---|---|
| `agent` | Run the main thread as a named subagent |
| `alwaysThinkingEnabled` | Enable extended thinking by default for all sessions |
| `apiKeyHelper` | Custom command to generate an auth value |
| `autoCompactEnabled` | Default true, automatically compact conversation near the context limit |
| `autoMemoryEnabled` | Default true, enable auto memory |
| `autoUpdatesChannel` | `"latest"` or `"stable"` release channel |
| `cleanupPeriodDays` | Default 30, session files older than this are deleted at startup |
| `defaultShell` | Default shell for `!` commands, `"bash"` or `"powershell"` |
| `editorMode` | `"normal"` or `"vim"` for the input prompt |
| `effortLevel` | Persist effort level across sessions |
| `fallbackModel` | Fallback model(s) to try when the primary is overloaded |
| `fastModePerSessionOptIn` | When true, fast mode resets to off each session |
| `fileCheckpointingEnabled` | Default true, snapshot files before edits for `/rewind` |
| `hooks` | Configure custom commands to run at lifecycle events |
| `model` | Override the default model |
| `outputStyle` | Configure an output style to adjust the system prompt |
| `permissions` | Allow, ask, deny rules and default mode |
| `statusLine` | Configure a custom status bar script |

---

# 8. How Claude remembers your project

> Give Claude persistent instructions with CLAUDE.md files, and let Claude accumulate learnings automatically with auto memory.

## CLAUDE.md vs auto memory

| | CLAUDE.md files | Auto memory |
|---|---|---|
| Who writes it | You | Claude |
| What it contains | Instructions and rules | Learnings and patterns |
| Scope | Project, user, or org | Per repository, shared across worktrees |
| Loaded into | Every session | Every session (first 200 lines or 25KB) |
| Use for | Coding standards, workflows, architecture | Build commands, debugging insights, preferences |

## CLAUDE.md files

Add to CLAUDE.md when Claude makes the same mistake twice, a code review catches something Claude should have known, or a new teammate would need the same context.

| Scope | Location | Shared with |
|---|---|---|
| Managed policy | `/Library/Application Support/ClaudeCode/CLAUDE.md` etc. | All users in organization |
| User instructions | `~/.claude/CLAUDE.md` | Just you (all projects) |
| Project instructions | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team via source control |
| Local instructions | `./CLAUDE.local.md` | Just you (current project) |

Run `/init` to generate a starting CLAUDE.md automatically. Target under 200 lines per file; longer files reduce adherence. Use markdown headers and bullets. Write specific, verifiable instructions: "Use 2-space indentation" instead of "Format code properly."

### Import additional files

```text
See @README for project overview and @package.json for available npm commands.
```
Relative paths resolve relative to the file containing the import; max depth of four hops. Wrap a path in backticks to mention without importing.

### AGENTS.md

Claude Code reads `CLAUDE.md`, not `AGENTS.md`. Create a `CLAUDE.md` that imports it: `@AGENTS.md` followed by Claude-specific instructions.

### Organize rules with .claude/rules/

Place markdown files in `.claude/rules/`, each covering one topic. Use YAML frontmatter with `paths:` to scope a rule to matching files only:

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API Development Rules
- All API endpoints must include input validation
```

## Auto memory

Enabled by default. Each project gets a memory directory at `~/.claude/projects/<project>/memory/` containing `MEMORY.md` (index, loaded at session start, capped at 200 lines/25KB) and topic files (loaded on demand). Toggle with `/memory` or `autoMemoryEnabled` setting.

## Troubleshoot memory issues

If Claude isn't following CLAUDE.md: run `/memory` to verify files are loaded, make instructions more specific, check for conflicting instructions. For anything that must run at a specific point, use a hook instead. Project-root CLAUDE.md survives `/compact` and reloads from disk; nested CLAUDE.md files reload only when Claude reads a file in that subdirectory.

---

# 9. Explore the .claude directory

> Where Claude Code reads CLAUDE.md, settings.json, hooks, skills, commands, subagents, workflows, rules, and auto memory.

## Project-level files

| File/folder | Committed? | Purpose |
|---|---|---|
| `CLAUDE.md` | Yes | Project instructions Claude reads every session |
| `.mcp.json` | Yes | Project-scoped MCP servers, shared with your team |
| `.worktreeinclude` | Yes | Gitignored files to copy into new git worktrees |
| `.claude/settings.json` | Yes | Permissions, hooks, and configuration enforced directly |
| `.claude/settings.local.json` | Gitignored | Personal settings overrides for this project |
| `.claude/rules/` | Yes | Topic-scoped instructions, optionally gated by file paths |
| `.claude/skills/` | Yes | Reusable prompts invoked by name, e.g. `/deploy` |
| `.claude/commands/` | Yes | Single-file prompts invoked with `/name` (skills are the recommended successor) |
| `.claude/output-styles/` | Yes | Project-scoped output styles |
| `.claude/agents/` | Yes | Specialized subagents with their own context window |
| `.claude/workflows/` | Yes | Dynamic workflow scripts orchestrating many subagents |
| `.claude/agent-memory/` | Auto-generated | Subagent persistent memory, separate from main session auto memory |

## User-level files (~/)

| File/folder | Purpose |
|---|---|
| `.claude.json` | App state, UI preferences, personal MCP servers, per-project trust |
| `.claude/CLAUDE.md` | Personal preferences across every project |
| `.claude/settings.json` | Default settings for all projects |
| `.claude/keybindings.json` | Custom keyboard shortcuts |
| `.claude/themes/` | Custom color themes |
| `.claude/projects/` | Auto memory, Claude's notes to itself, per project |
| `.claude/rules/`, `.claude/skills/`, `.claude/commands/`, `.claude/agents/`, `.claude/workflows/`, `.claude/output-styles/` | Personal versions available in every project |

Settings follow precedence (project overrides user); CLAUDE.md files are all concatenated into context rather than overriding each other, ordered broadest to most specific.

---

# 10. Manage sessions

> Name, resume, branch, and switch between Claude Code conversations.

## Resume a session

| Command | What it does |
|---|---|
| `claude --continue` | Resumes the most recent session in the current directory |
| `claude --resume` | Opens the session picker |
| `claude --resume <name>` | Resumes the named session directly |
| `claude --from-pr <number>` | Resumes the session linked to that pull request |
| `/resume` | Switches to a different conversation from inside an active session |

## Name your sessions

| When | How |
|---|---|
| At startup | `claude -n auth-refactor` |
| During a session | `/rename auth-refactor` |
| From the session picker | Highlight a session, press `Ctrl+R` |
| On plan accept | Names the session from the plan content automatically |

## Use the session picker

Run `/resume` or `claude --resume` with no arguments. Navigate with arrows, `Enter` to resume, `Space` to preview, `Ctrl+R` to rename, `Ctrl+A` to show all projects, `Ctrl+W` to show all worktrees, `Ctrl+B` to filter by branch.

## Branch a session

```text
/branch try-streaming-approach
```
Creates a copy of the conversation so far and switches into it, leaving the original intact. Permissions approved with "allow for this session" do not carry over.

## Manage context within a session

- `/clear`: start fresh with empty context, previous conversation stays resumable
- `/compact [instructions]`: replace history with a summary
- `/context`: show what is currently consuming context

## Export and locate session data

`/export` opens a menu to copy or save the conversation as readable text. Transcripts are stored as JSONL at `~/.claude/projects/<project>/<session-id>.jsonl`. Configure retention with `cleanupPeriodDays`, or move storage with `CLAUDE_CONFIG_DIR`.

---

# 11. Customize your status line

> Configure a custom status bar to monitor context window usage, costs, and git status in Claude Code.

The status line is a customizable bar that runs any shell script you configure, receiving JSON session data on stdin and displaying whatever the script prints.

## Set up a status line

Use `/statusline show model name and context percentage with a progress bar` to have Claude generate a script, or configure manually:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2
  }
}
```

## How status lines work

The script runs after each new assistant message, after `/compact`, when permission mode changes, or when vim mode toggles, debounced at 300ms. Set `refreshInterval` to also re-run on a fixed timer during idle periods.

## Available data (key fields)

| Field | Description |
|---|---|
| `model.display_name` | Current model display name |
| `workspace.current_dir` | Current working directory |
| `cost.total_cost_usd` | Estimated session cost in USD |
| `cost.total_duration_ms` | Total wall-clock time since session started |
| `context_window.used_percentage` | Pre-calculated percentage of context window used |
| `rate_limits.five_hour.used_percentage` | Percentage of the 5-hour rate limit consumed |
| `session_id` | Unique session identifier |
| `pr.number`, `pr.review_state` | Open pull request info for the current branch |
| `vim.mode` | Current vim mode when enabled |
| `worktree.name` | Active worktree name, present only during `--worktree` sessions |

## Example: context window usage bar

```bash
#!/bin/bash
input=$(cat)
MODEL=$(echo "$input" | jq -r '.model.display_name')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)
BAR_WIDTH=10
FILLED=$((PCT * BAR_WIDTH / 100))
echo "[$MODEL] progress: $FILLED/$BAR_WIDTH blocks, $PCT%"
```

## Subagent status lines

The `subagentStatusLine` setting renders a custom row for each subagent shown in the agent panel, receiving a `tasks` array on stdin and writing `{"id": "...", "content": "..."}` lines to override specific rows.

## Tips

Test with mock JSON input piped to your script. Keep output short since the bar has limited width. Cache slow operations like `git status` using the `session_id` as a stable cache key.

---

# 12. Customize keyboard shortcuts

> Customize keyboard shortcuts in Claude Code with a keybindings configuration file.

Run `/keybindings` to create or open `~/.claude/keybindings.json`. Changes are detected and applied automatically without restarting.

## Configuration file

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null
      }
    }
  ]
}
```

## Contexts

Each binding block specifies where it applies: `Global`, `Chat`, `Autocomplete`, `Settings`, `Confirmation`, `Transcript`, `HistorySearch`, `Task`, `ThemePicker`, `Footer`, `DiffDialog`, `ModelPicker`, `Select`, `Plugin`, `Scroll`, and more.

## Keystroke syntax

**Modifiers**: `ctrl`/`control`, `shift`, `alt`/`opt`/`option`/`meta`, `cmd`/`command`/`super`/`win`. Example: `meta+p` means Option+P on macOS, Alt+P elsewhere.

**Uppercase letters** imply Shift: `K` equals `shift+k`. With modifiers, `ctrl+K` equals `ctrl+k`.

**Chords** are sequences separated by spaces: `ctrl+k ctrl+s` presses Ctrl+K, releases, then Ctrl+S.

**Special keys**: `escape`/`esc`, `enter`/`return`, `tab`, `space`, arrow keys, `backspace`/`delete`.

## Unbind default shortcuts

```json
{
  "bindings": [
    { "context": "Chat", "bindings": { "ctrl+s": null } }
  ]
}
```

## Reserved shortcuts

`Ctrl+C` (interrupt), `Ctrl+D` (exit), `Ctrl+M` (identical to Enter), Caps Lock cannot be rebound.

## Terminal conflicts

`Ctrl+B` conflicts with tmux prefix (press twice), `Ctrl+A` with GNU screen prefix, `Ctrl+Z` with Unix process suspend.

Run `/doctor` to see keybinding validation warnings: parse errors, invalid contexts, reserved conflicts, duplicate bindings.

---

# 13. Configure your terminal for Claude Code

> Fix Shift+Enter for newlines, get a terminal bell when Claude finishes, configure tmux, match the color theme, and enable Vim mode.

## Enter multiline prompts

Press `Ctrl+J` or type `\` then Enter to add a line break without submitting, works in every terminal. Shift+Enter works natively in Ghostty, Kitty, iTerm2, WezTerm, Warp, Apple Terminal, Windows Terminal; for VS Code, Cursor, Devin Desktop, Alacritty, Zed, run `/terminal-setup` once.

## Enable Option key shortcuts on macOS

Some shortcuts use Option, like Option+Enter for newline. Enable "Use Option as Meta Key" in Apple Terminal or set Left/Right Option to "Esc+" in iTerm2 Settings → Profiles → Keys.

## Get a terminal bell or notification

By default, desktop notifications appear only in Ghostty, Kitty, and iTerm2. Set `preferredNotifChannel` to `"terminal_bell"` for other terminals, or configure a Notification hook to play a custom sound:

```json
{
  "hooks": {
    "Notification": [
      { "hooks": [{ "type": "command", "command": "afplay /System/Library/Sounds/Glass.aiff" }] }
    ]
  }
}
```

## Configure tmux

Add to `~/.tmux.conf`:
```bash
set -g allow-passthrough on
set -s extended-keys on
set -as terminal-features 'xterm*:extkeys'
```
This lets notifications reach the outer terminal and lets tmux distinguish Shift+Enter from plain Enter.

## Match the color theme

Use `/theme` to pick a theme matching your terminal; the auto option follows OS light/dark appearance. Create custom themes as JSON files in `~/.claude/themes/` with `name`, `base` (dark/light preset), and `overrides` (color token map).

## Switch to fullscreen rendering

If display flickers or scroll position jumps, run `/tui fullscreen`, or set `CLAUDE_CODE_NO_FLICKER=1` before starting.

## Edit prompts with Vim keybindings

Enable via `/config` → Editor mode, or set `editorMode` to `"vim"` in settings.json. Pressing Enter still submits in INSERT mode, unlike standard Vim; use `o`/`O` or `Ctrl+J` for a newline instead.

---

# 14. Fullscreen rendering

> Enable a smoother, flicker-free rendering mode with mouse support and stable memory usage in long conversations.

Fullscreen rendering is a research preview alternative rendering path that eliminates flicker, keeps memory flat in long conversations, and adds mouse support, drawing on the terminal's alternate screen buffer like `vim` or `htop`.

## Enable fullscreen rendering

Run `/tui fullscreen` inside any conversation to relaunch with your conversation intact, or set `CLAUDE_CODE_NO_FLICKER=1` before starting. Run `/tui default` to switch back.

## What changes

| Before | Now |
|---|---|
| `Cmd+f` or tmux search to find text | `Ctrl+o` for transcript mode, then `/` to search or `[` to write to scrollback |
| Native click-and-drag to select and copy | In-app selection, copies automatically on mouse release |
| `Cmd`-click to open a URL | `Cmd`-click on macOS, `Ctrl`-click elsewhere |

## Use the mouse

Click in the prompt to position cursor; click a suggestion to accept it; click a collapsed tool result to expand it; hold `Cmd`/`Ctrl` and click a URL or path to open it; click and drag to select text, which copies to clipboard automatically on release.

## Scroll the conversation

| Shortcut | Action |
|---|---|
| `PgUp` / `PgDn` | Scroll up or down by half a screen |
| `Ctrl+Home` | Jump to the start of the conversation |
| `Ctrl+End` | Jump to the latest message, re-enable auto-follow |
| Mouse wheel | Scroll a few lines at a time |

## Search and review the conversation

`Ctrl+o` toggles transcript mode, which gains `less`-style navigation: `/` to search, `n`/`N` for next/previous match, `g`/`G` to jump top/bottom. Press `[` to write the full conversation into native scrollback so `Cmd+f` and tmux search work.

## Keep native text selection

Mouse capture stops native copy-on-select. Hold a modifier key (varies by terminal, e.g. `Option` in iTerm2, `Shift` elsewhere) for one-off native selection, or set `CLAUDE_CODE_DISABLE_MOUSE=1` to opt out entirely while keeping flicker-free rendering.

---

# 15. Speed up responses with fast mode

> Get faster Opus responses in Claude Code by toggling fast mode.

Fast mode is a high-speed configuration for Claude Opus, up to 2.5x faster at a higher cost per token. It is not a different model, just a different API configuration prioritizing speed. Supported on Opus 4.8 and Opus 4.7 only.

## Toggle fast mode

Type `/fast` and press Tab to toggle, or set `"fastMode": true` in user settings. Enabling switches you to Opus if on a different model, and shows a `↯` icon while active. Enable at the start of a session for the best cost efficiency.

## Cost tradeoff

| Model | Input (MTok) | Output (MTok) |
|---|---|---|
| Opus 4.8 | $10 | $50 |
| Opus 4.7 | $30 | $150 |

The first time you enable fast mode in a conversation, you pay the full fast mode price for the entire existing context, so enabling from the start is cheaper.

## When to use it

Best for rapid iteration, live debugging, time-sensitive work. Standard mode is better for long autonomous tasks, batch processing, or cost-sensitive workloads. Can be combined with a lower effort level for maximum speed on straightforward tasks.

## Requirements

Requires Anthropic API or subscription (not available on Bedrock, Vertex, Foundry, or Claude Platform on AWS), usage credits turned on, and Owner enablement for Team/Enterprise organizations.

## Rate limits

Fast mode has separate rate limits shared between Opus 4.8 and 4.7. When exhausted, it falls back to standard speed automatically and the icon turns gray; it re-enables when the cooldown expires.

---

# 16. Model configuration

> Learn about the Claude Code model configuration, including model aliases like `opusplan`.

## Available models

Configure either a model alias or a model name (a full model name for Anthropic API, an inference profile ARN for Bedrock, a deployment name for Foundry, or a version name for Vertex).

## Model aliases

| Alias | Behavior |
|---|---|
| `default` | Clears any override, reverts to the recommended model for your account type |
| `sonnet` | Latest Sonnet model |
| `opus` | Latest Opus model |
| `haiku` | Latest Haiku model |
| `fable` | Latest Fable model |
| `opusplan` | Uses Opus for plan mode, then switches to Sonnet for implementation |

## Setting your model

Switch with `/model` during a session (opens a picker; press `s` to switch for the current session only), or start with `claude --model <name>`. Persisted choices are saved as your default for new sessions.

## Adjust effort level

Effort controls how much of the adaptive-reasoning thinking budget Claude uses. Set with `/effort [level]`, accepting `low`, `medium`, `high`, `xhigh`, `max`, or `ultracode` (session-only, combines xhigh reasoning with automatic workflow orchestration). Supported on Fable 5, Opus 4.6+, and Sonnet 4.6+.

## Fallback model chains

```bash
claude --fallback-model sonnet,haiku
```
Claude Code switches to the next available model in the chain for the rest of the turn when the primary is overloaded, capped at three models.

## Restrict model selection

Administrators can set `availableModels` in settings to restrict which models users, subagents, skills, and the advisor can select, e.g. `["sonnet", "haiku"]`. Setting `enforceAvailableModels` extends the allowlist to the Default option as well.
