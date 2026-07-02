# 第一階段, 入門與基本認識

先了解 Claude Code 是什麼, 如何安裝與啟動.

---

# 1. Overview

> Claude Code is an agentic coding tool that reads your codebase, edits files, runs commands, and integrates with your development tools. Available in your terminal, IDE, desktop app, and browser.

Claude Code is an AI-powered coding assistant that helps you build features, fix bugs, and automate development tasks. It understands your entire codebase and can work across multiple files and tools to get things done.

## Get started

Choose your environment to get started. Most surfaces require a Claude subscription or Anthropic Console account. The Terminal CLI and VS Code also support third-party providers.

### Terminal

The full-featured CLI for working with Claude Code directly in your terminal. Edit files, run commands, and manage your entire project from the command line.

To install Claude Code, use one of the following methods:

**Native Install (Recommended)**

macOS, Linux, WSL:
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Windows PowerShell:
```powershell
irm https://claude.ai/install.ps1 | iex
```

Windows CMD:
```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

If you see `The token '&&' is not a valid statement separator`, you're in PowerShell, not CMD. If you see `'irm' is not recognized as an internal or external command`, you're in CMD, not PowerShell.

Git for Windows is recommended on native Windows so Claude Code can use the Bash tool. If Git for Windows is not installed, Claude Code uses PowerShell as the shell tool instead. WSL setups do not need Git for Windows.

Native installations automatically update in the background to keep you on the latest version.

**Homebrew**
```bash
brew install --cask claude-code
```
Homebrew offers two casks. `claude-code` tracks the stable release channel; `claude-code@latest` tracks the latest channel and receives new versions as soon as they ship. Homebrew installations do not auto-update; run `brew upgrade claude-code` periodically.

**WinGet**
```powershell
winget install Anthropic.ClaudeCode
```
WinGet installations do not auto-update; run `winget upgrade Anthropic.ClaudeCode` periodically.

You can also install with apt, dnf, or apk on Debian, Fedora, RHEL, and Alpine.

Then start Claude Code in any project:
```bash
cd your-project
claude
```

You'll be prompted to log in on first use.

### VS Code

The VS Code extension provides inline diffs, @-mentions, plan review, and conversation history directly in your editor. Search for "Claude Code" in the Extensions view, then open the Command Palette, type "Claude Code", and select Open in New Tab.

### Desktop app

A standalone app for running Claude Code outside your IDE or terminal. Review diffs visually, run multiple sessions side by side, schedule recurring tasks, and kick off cloud sessions. Available for macOS (Intel and Apple Silicon), Windows (x64), and Windows ARM64. After installing, launch Claude, sign in, and click the Code tab to start coding. A paid subscription is required.

### Web

Run Claude Code in your browser with no local setup. Kick off long-running tasks and check back when they're done, work on repos you don't have locally, or run multiple tasks in parallel. Available on desktop browsers and the Claude iOS app. Start coding at claude.ai/code.

### JetBrains

A plugin for IntelliJ IDEA, PyCharm, WebStorm, and other JetBrains IDEs with interactive diff viewing and selection context sharing. Install the Claude Code plugin from the JetBrains Marketplace and restart your IDE. The plugin requires the Claude Code CLI, installed separately.

## What you can do

- **Automate the work you keep putting off**: writing tests for untested code, fixing lint errors across a project, resolving merge conflicts, updating dependencies, and writing release notes.
- **Build features and fix bugs**: describe what you want in plain language. Claude Code plans the approach, writes the code across multiple files, and verifies it works.
- **Create commits and pull requests**: Claude Code works directly with git, stages changes, writes commit messages, creates branches, and opens pull requests.
- **Connect your tools with MCP**: the Model Context Protocol is an open standard for connecting AI tools to external data sources, like Google Drive, Jira, Slack, or your own custom tooling.
- **Customize with instructions, skills, and hooks**: CLAUDE.md is a markdown file read at the start of every session. Skills package repeatable workflows. Hooks run shell commands before or after Claude Code actions.
- **Run agent teams and build custom agents**: spawn multiple Claude Code agents working on different parts of a task simultaneously. The Agent SDK lets you build your own agents.
- **Pipe, script, and automate with the CLI**: Claude Code is composable and follows the Unix philosophy, so you can pipe logs into it, run it in CI, or chain it with other tools.
- **Schedule recurring tasks**: Routines run on Anthropic-managed infrastructure. Desktop scheduled tasks run on your machine. `/loop` repeats a prompt within a CLI session.
- **Work from anywhere**: sessions aren't tied to a single surface. Move work between environments as your context changes, with Remote Control, Dispatch, web, or `claude --teleport`.

## Use Claude Code everywhere

Each surface connects to the same underlying Claude Code engine, so your CLAUDE.md files, settings, and MCP servers work across all of them. Claude Code also integrates with CI/CD, chat, and browser workflows: GitHub Actions, GitLab CI/CD, GitHub Code Review, Slack, Chrome, and the Agent SDK.

## Next steps

- Quickstart: walk through your first real task, from exploring a codebase to committing a fix
- Store instructions and memories: give Claude persistent instructions with CLAUDE.md files and auto memory
- Common workflows and best practices: patterns for getting the most out of Claude Code
- Settings: customize Claude Code for your workflow
- Troubleshooting: solutions for common issues

---

# 2. Quickstart

> Welcome to Claude Code!

This quickstart guide will have you using AI-powered coding assistance in a few minutes. By the end, you'll understand how to use Claude Code for common development tasks.

## Before you begin

Make sure you have a terminal or command prompt open, a code project to work with, and a Claude subscription (Pro, Max, Team, or Enterprise), Claude Console account, or access through a supported cloud provider.

This guide covers the terminal CLI. Claude Code is also available on the web, as a desktop app, in VS Code and JetBrains IDEs, in Slack, and in CI/CD with GitHub Actions and GitLab.

## Step 1: Install Claude Code

See the installation methods above (Native Install, Homebrew, WinGet). You can also install with apt, dnf, or apk on Debian, Fedora, RHEL, and Alpine.

## Step 2: Log in to your account

Claude Code requires an account to use. Start an interactive session with the `claude` command and you'll be prompted to log in on first use:
```bash
claude
```
To switch accounts later or re-authenticate, type `/login` inside the running session. You can log in using Claude Pro/Max/Team/Enterprise (recommended), Claude Console, Amazon Bedrock/Google Vertex AI/Microsoft Foundry, or a self-hosted Claude apps gateway.

Once logged in, your credentials are stored and you won't need to log in again.

## Step 3: Start your first session

Open your terminal in any project directory and start Claude Code:
```bash
cd /path/to/your/project
claude
```
You'll see the Claude Code prompt with the version, current model, and working directory shown above it. Type `/help` for available commands or `/resume` to continue a previous conversation.

## Step 4: Ask your first question

Try commands like:
- `what does this project do?`
- `what technologies does this project use?`
- `where is the main entry point?`
- `explain the folder structure`

Claude Code reads your project files as needed. You don't have to manually add context.

## Step 5: Make your first code change

Try a simple task:
```text
add a hello world function to the main file
```
Claude Code will find the appropriate file, show you the proposed changes, ask for your approval, and make the edit. Claude Code always asks for permission before modifying files. You can approve individual changes or enable "Accept all" mode for a session.

## Step 6: Use Git with Claude Code

Claude Code makes Git operations conversational:
```text
what files have I changed?
commit my changes with a descriptive message
create a new branch called feature/quickstart
show me the last 5 commits
help me resolve merge conflicts
```

## Step 7: Fix a bug or add a feature

Describe what you want in natural language, e.g. "add input validation to the user registration form", or fix existing issues like "there's a bug where users can submit empty forms - fix it". Claude Code will locate the relevant code, understand the context, implement a solution, and run tests if available.

## Step 8: Test out other common workflows

- **Refactor code**: "refactor the authentication module to use async/await instead of callbacks"
- **Write tests**: "write unit tests for the calculator functions"
- **Update documentation**: "update the README with installation instructions"
- **Code review**: "review my changes and suggest improvements"

## Essential commands

Shell commands (run from your terminal):

| Command | What it does | Example |
|---|---|---|
| `claude` | Start interactive mode | `claude` |
| `claude "task"` | Run a one-time task | `claude "fix the build error"` |
| `claude -p "query"` | Run one-off query, then exit | `claude -p "explain this function"` |
| `claude -c` | Continue most recent conversation in current directory | `claude -c` |
| `claude -r` | Resume a previous conversation | `claude -r` |

Session commands (run inside Claude Code):

| Command | What it does | Example |
|---|---|---|
| `/clear` | Clear conversation history | `/clear` |
| `/help` | Show available commands | `/help` |
| `/exit` or Ctrl+D | Exit Claude Code | `/exit` |

## Pro tips for beginners

- **Be specific with your requests**: instead of "fix the bug", try "fix the login bug where users see a blank screen after entering wrong credentials".
- **Use step-by-step instructions**: break complex tasks into numbered steps.
- **Let Claude explore first**: before making changes, let Claude understand your code, e.g. "analyze the database schema".
- **Save time with shortcuts**: type `/` to see all commands and skills, use Tab for command completion, press up arrow for command history, press Shift+Tab to cycle permission modes.

## What's next?

- How Claude Code works: understand the agentic loop, built-in tools, and how Claude Code interacts with your project
- Best practices: get better results with effective prompting and project setup
- Common workflows: step-by-step guides for common tasks
- Extend Claude Code: customize with CLAUDE.md, skills, hooks, MCP, and more

## Getting help

- In Claude Code: type `/help` or ask "how do I..."
- Documentation: browse other guides
- Community: join the Discord for tips and support

---

# 3. How Claude Code works

> Understand the agentic loop, built-in tools, and how Claude Code interacts with your project.

Claude Code is an agentic assistant that runs in your terminal. While it excels at coding, it can help with anything you can do from the command line: writing docs, running builds, searching files, researching topics, and more.

## The agentic loop

When you give Claude a task, it works through three phases: gather context, take action, and verify results. These phases blend together. Claude uses tools throughout, whether searching files to understand your code, editing to make changes, or running tests to check its work.

The loop adapts to what you ask. A question about your codebase might only need context gathering. A bug fix cycles through all three phases repeatedly. A refactor might involve extensive verification. Claude decides what each step requires based on what it learned from the previous step, chaining dozens of actions together and course-correcting along the way.

You're part of this loop too. You can interrupt at any point to steer Claude in a different direction, provide additional context, or ask it to try a different approach.

The agentic loop is powered by two components: models that reason and tools that act. Claude Code serves as the agentic harness around Claude: it provides the tools, context management, and execution environment that turn a language model into a capable coding agent.

### Models

Claude Code uses Claude models to understand your code and reason about tasks. Multiple models are available with different tradeoffs. Sonnet handles most coding tasks well. Opus provides stronger reasoning for complex architectural decisions. Switch with `/model` during a session or start with `claude --model <name>`.

### Tools

Tools are what make Claude Code agentic. Without tools, Claude can only respond with text. With tools, Claude can act: read your code, edit files, run commands, search the web, and interact with external services.

The built-in tools generally fall into five categories:

| Category | What Claude can do |
|---|---|
| File operations | Read files, edit code, create new files, rename and reorganize |
| Search | Find files by pattern, search content with regex, explore codebases |
| Execution | Run shell commands, start servers, run tests, use git |
| Web | Search the web, fetch documentation, look up error messages |
| Code intelligence | See type errors and warnings after edits, jump to definitions, find references |

Claude chooses which tools to use based on your prompt and what it learns along the way. When you say "fix the failing tests," Claude might run the test suite, read the error output, search for relevant source files, read them, edit them, and run the tests again to verify.

**Extending the base capabilities**: The built-in tools are the foundation. You can extend what Claude knows with skills, connect to external services with MCP, automate workflows with hooks, and offload tasks to subagents.

## What Claude can access

When you run `claude` in a directory, Claude Code gains access to your project files, your terminal, your git state, your CLAUDE.md, auto memory, and extensions you configure (MCP servers, skills, subagents, Claude in Chrome).

Because Claude sees your whole project, it can work across it. This is different from inline code assistants that only see the current file.

## Environments and interfaces

Claude Code runs in three environments:

| Environment | Where code runs | Use case |
|---|---|---|
| Local | Your machine | Default. Full access to your files, tools, and environment |
| Cloud | Anthropic-managed VMs | Offload tasks, work on repos you don't have locally |
| Remote Control | Your machine, controlled from a browser | Use the web UI while keeping everything local |

You can access Claude Code through the terminal, the desktop app, IDE extensions, claude.ai/code, Remote Control, Slack, and CI/CD pipelines. The interface determines how you see and interact with Claude, but the underlying agentic loop is identical.

## Work with sessions

Claude Code saves your conversation locally as you work. Each message, tool use, and result is written to a plaintext JSONL file under `~/.claude/projects/`, which enables rewinding, resuming, and forking sessions.

**Sessions are independent.** Each new session starts with a fresh context window. Claude can persist learnings across sessions using auto memory, and you can add your own persistent instructions in CLAUDE.md.

### Work across branches

Each Claude Code conversation is a session tied to your current directory. Claude sees your current branch's files. When you switch branches, Claude sees the new branch's files, but your conversation history stays the same.

### Resume or fork sessions

Resuming a session with `claude --continue` or `claude --resume` reopens it under the same session ID and appends new messages to the existing conversation. Forking with `--fork-session` or `/branch` copies the history into a new session ID, leaving the original unchanged.

### The context window

Claude's context window holds your conversation history, file contents, command outputs, CLAUDE.md, auto memory, loaded skills, and system instructions. As you work, context fills up. Claude compacts automatically, but instructions from early in the conversation can get lost. Put persistent rules in CLAUDE.md, and run `/context` to see what's using space.

#### When context fills up

Claude Code manages context automatically as you approach the limit. It clears older tool outputs first, then summarizes the conversation if needed. To control what's preserved during compaction, add a "Compact Instructions" section to CLAUDE.md or run `/compact` with a focus.

#### Manage context with skills and subagents

Skills load on demand: Claude sees skill descriptions at session start, but the full content only loads when a skill is used. Subagents get their own fresh context, completely separate from your main conversation, and return a summary when done.

## Stay safe with checkpoints and permissions

### Undo changes with checkpoints

Every file edit is reversible. Before Claude edits any file, it snapshots the current contents. If something goes wrong, press Esc twice to rewind to a previous state, or ask Claude to undo. Checkpoints are local to your session, separate from git, and only cover file changes.

### Control what Claude can do

Press Shift+Tab to cycle through permission modes:
- **Default**: Claude asks before file edits and shell commands
- **Auto-accept edits**: Claude edits files and runs common filesystem commands without asking
- **Plan mode**: Claude explores and proposes a plan without editing your source files
- **Auto mode**: Claude evaluates all actions with background safety checks (research preview)

You can also allow specific commands in `.claude/settings.json` so Claude doesn't ask each time.

## Work effectively with Claude Code

### Ask Claude Code for help

Claude Code can teach you how to use it. Built-in commands like `/init`, `/agents`, and `/doctor` guide you through setup.

### It's a conversation

Claude Code is conversational. You don't need perfect prompts. Start with what you want, then refine.

#### Interrupt and steer

Press Esc to stop Claude immediately, or type a correction and press Enter to send it without stopping the running tool.

### Be specific upfront

The more precise your initial prompt, the fewer corrections you'll need. Reference specific files, mention constraints, and point to example patterns.

### Give Claude something to verify against

Claude performs better when it can check its own work. Include test cases, paste screenshots of expected UI, or define the output you want.

### Explore before implementing

For complex problems, separate research from coding. Use plan mode (Shift+Tab twice) to analyze the codebase first.

### Delegate, don't dictate

Think of delegating to a capable colleague. Give context and direction, then trust Claude to figure out the details.

---

# 4. Glossary

> Definitions for Claude Code terminology.

**Agent teams**: Multiple independent Claude Code sessions coordinated by a team lead, with a shared task list and peer-to-peer messaging. Experimental, enabled with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`.

**Agentic coding**: A workflow where the AI can read files, run commands, and make changes autonomously while you watch, redirect, or step away.

**Agentic harness**: The tools, context management, and execution environment that turn a language model into a capable coding agent. Claude Code is the harness; Claude is the model inside it.

**Agentic loop**: The cycle Claude works through for every task: gather context, take action, verify results, and repeat until done.

**Artifact**: A live, interactive web page Claude Code publishes from your session to a private URL on claude.ai.

**Auto memory**: Notes Claude writes for itself based on your corrections and preferences, stored per git repository under `~/.claude/projects/`.

**Auto mode**: A permission mode where a separate classifier model reviews actions in the background, so most run without approval prompts.

**Bare mode**: A startup flag, `--bare`, that skips auto-discovery of hooks, skills, plugins, MCP servers, auto memory, and CLAUDE.md.

**Bundled skills**: Prompt-based playbooks included with Claude Code, such as `/batch`, `/code-review`, `/debug`, and `/loop`.

**Channel**: An MCP server that pushes events into your running session so Claude can react to things that happen while you're away.

**Checkpoint**: A restore point created at each prompt you send. Press Esc twice or run `/rewind` to restore code, conversation, or both.

**.claude directory**: The directory where Claude Code reads project-scoped configuration: settings, hooks, skills, subagents, rules, and auto memory.

**CLAUDE.md**: A markdown file of persistent instructions you write for Claude, loaded at the start of every session.

**Command**: A reusable instruction you invoke by typing `/name` in the prompt.

**Compaction**: Automatic summarization of your conversation when the context window approaches its limit.

**Context window**: The working memory for a session, holding conversation history, file contents, command outputs, CLAUDE.md, auto memory, loaded skills, and system instructions.

**Dispatch**: A phone-initiated task router that spawns a Claude Code session in the Desktop app.

**Effort level**: A setting that controls how much of the adaptive-reasoning thinking budget Claude uses on each turn.

**Extended thinking**: Visible step-by-step reasoning the model performs before responding.

**Hook**: A user-defined handler that executes automatically at a specific point in Claude Code's lifecycle.

**Managed settings**: Settings enforced org-wide by IT or DevOps, delivered from Anthropic's servers.

**MCP (Model Context Protocol)**: An open standard for connecting AI tools to external data sources and services.

**MCP Tool Search**: A context-saving mechanism that defers MCP tool schemas until needed.

**Non-interactive mode**: A mode that executes a single prompt and exits without a conversational session, invoked with `-p` or `--print`. Formerly called headless mode.

**Output style**: A configuration that modifies Claude's system prompt to change response behavior, tone, or format.

**Permission mode**: The baseline approval behavior for the session. Available modes are `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, and `bypassPermissions`.

**Permission rule**: A settings entry that allows, asks about, or denies a tool invocation based on the tool name and argument pattern.

**Plan mode**: A permission mode where Claude researches and proposes changes without editing your source files.

**Plugin**: A bundle of skills, hooks, subagents, and MCP servers packaged as a single installable unit.

**Project trust**: A dialog accepting a directory before Claude Code loads its configuration.

**Prompt injection**: Hostile instructions embedded in a file, web page, or tool result that attempt to redirect Claude toward actions you never asked for.

**Remote Control**: A way to continue a local Claude Code session from your phone or browser via claude.ai.

**Rules**: Modular instruction files in `.claude/rules/` that load alongside CLAUDE.md.

**Sandboxing**: OS-level filesystem and network isolation for the Bash tool.

**Session**: A conversation tied to your current directory, with its own independent context window.

**Settings layers**: The hierarchy Claude Code reads configuration from, in precedence order: managed policy, command-line arguments, local settings, project settings, then user settings.

**Skill**: A `SKILL.md` file containing instructions, knowledge, or a workflow that Claude adds to its toolkit.

**Subagent**: A specialized AI assistant that runs in its own context window with a custom system prompt, specific tool access, and independent permissions.

**Surface**: Any place you access Claude Code: the CLI, VS Code, JetBrains, Desktop, or claude.ai.

**Teleport**: A command, `/teleport`, that pulls a cloud Claude Code session into your local terminal.

**Tool**: An action Claude can take: read a file, edit code, run a shell command, search the web, spawn a subagent.

**Turn**: One complete response from Claude within a session.

**Verification loop**: How a session knows the work is actually done rather than just plausible.

**Worktree isolation**: An isolation mode that runs Claude in a separate git worktree under `.claude/worktrees/`.

### Deprecated and renamed terms

| Old term | Now called | Notes |
|---|---|---|
| Headless mode | Non-interactive mode | Same `-p` flag, same behavior |
| Custom commands | Skills | `.claude/commands/` files still work |
| Slash commands | Commands | "Slash" dropped from product copy |

---

# 5. Authentication

> Log in to Claude Code and configure authentication for individuals, teams, and organizations.

Claude Code supports multiple authentication methods depending on your setup. Individual users can log in with a Claude.ai account, while teams can use Claude for Teams or Enterprise, the Claude Console, or a cloud provider like Amazon Bedrock, Google Vertex AI, or Microsoft Foundry.

## Log in to Claude Code

After installing Claude Code, run `claude` in your terminal. On first launch, Claude Code opens a browser window for you to log in. If the browser doesn't open automatically, press `c` to copy the login URL to your clipboard.

You can authenticate with:
- **Claude Pro or Max subscription**: log in with your Claude.ai account
- **Claude for Teams or Enterprise**: log in with the Claude.ai account your team admin invited you to
- **Claude Console**: log in with your Console credentials
- **Cloud providers**: Amazon Bedrock, Google Vertex AI, or Microsoft Foundry, set required environment variables before running `claude`
- **Cloud gateway**: sign in with corporate SSO through `/login`

To log out and re-authenticate, type `/logout` at the Claude Code prompt.

## Set up team authentication

For teams and organizations, configure Claude Code access via Claude for Teams/Enterprise (recommended for most teams), Claude Console, Claude apps gateway, Amazon Bedrock, Google Vertex AI, or Microsoft Foundry.

### Claude for Teams or Enterprise

Claude for Teams and Claude for Enterprise provide the best experience for organizations using Claude Code. Team members get access to both Claude Code and Claude on the web with centralized billing and team management.

- **Claude for Teams**: self-service plan with collaboration features, admin tools, and billing management
- **Claude for Enterprise**: adds SSO, domain capture, role-based permissions, compliance API, and managed policy settings

Steps: subscribe, invite team members from the admin dashboard, then team members install Claude Code and log in with their Claude.ai accounts.

### Claude Console authentication

For organizations that prefer API-based billing, create or use a Console account, add users (bulk invite or SSO), assign roles (Claude Code role or Developer role), and have each user accept the invite, install Claude Code, and log in.

### Cloud provider authentication

For teams using Amazon Bedrock, Google Vertex AI, or Microsoft Foundry: follow provider setup, distribute environment variables and credential instructions, then users install Claude Code.

## Credential management

Claude Code securely manages your authentication credentials:

- On macOS, credentials are stored in the encrypted macOS Keychain
- On Linux, credentials are stored in `~/.claude/.credentials.json` with file mode `0600`
- On Windows, credentials are stored in `%USERPROFILE%\.claude\.credentials.json`
- Supported authentication types: Claude.ai credentials, Claude API credentials, Azure Auth, Bedrock Auth, Vertex Auth, and Claude apps gateway session tokens
- Custom credential scripts: the `apiKeyHelper` setting can run a shell script that returns an API key
- Refresh intervals: `apiKeyHelper` is called after 5 minutes or on HTTP 401 response by default

### Authentication precedence

When multiple credentials are present, Claude Code chooses one in this order:

1. Cloud provider credentials (Bedrock, Vertex, Foundry)
2. `ANTHROPIC_AUTH_TOKEN` environment variable
3. `ANTHROPIC_API_KEY` environment variable
4. `apiKeyHelper` script output
5. `CLAUDE_CODE_OAUTH_TOKEN` environment variable
6. Subscription OAuth credentials from `/login`

A signed-in Claude apps gateway session outranks all of these.

### Generate a long-lived token

For CI pipelines or scripts where interactive browser login isn't available, generate a one-year OAuth token:
```bash
claude setup-token
```
Set it as the `CLAUDE_CODE_OAUTH_TOKEN` environment variable. This token authenticates with your Claude subscription and requires a Pro, Max, Team, or Enterprise plan.

---

# 6. Advanced setup

> System requirements, platform-specific installation, version management, and uninstallation for Claude Code.

## System requirements

- **Operating system**: macOS 13.0+, Windows 10 1809+ or Windows Server 2019+, Ubuntu 20.04+, Debian 10+, Alpine Linux 3.19+
- **Hardware**: 4 GB+ RAM, x64 or ARM64 processor
- **Network**: internet connection required
- **Shell**: Bash, Zsh, PowerShell, or CMD

## Install Claude Code

Use Native Install, Homebrew, or WinGet (see Overview section above for commands). You can also install with apt, dnf, or apk on Debian, Fedora, RHEL, and Alpine.

After installation completes, open a terminal in the project you want to work in and start Claude Code with `claude`.

### Set up on Windows

| Option | Requires | Sandboxing | When to use |
|---|---|---|---|
| Native Windows | None; Git for Windows optional | Not supported | Windows-native projects and tools |
| WSL 2 | WSL 2 enabled | Supported | Linux toolchains or sandboxed command execution |
| WSL 1 | WSL 1 enabled | Not supported | If WSL 2 is unavailable |

Without Git for Windows, Claude Code runs shell commands via the PowerShell tool. With Git for Windows, Claude Code uses Git Bash for the Bash tool.

### Alpine Linux and musl-based distributions

Requires `libgcc`, `libstdc++`, and `ripgrep`, plus setting `USE_BUILTIN_RIPGREP=0`.

## Verify your installation

```bash
claude --version
claude doctor
```

## Authenticate

Claude Code requires a Pro, Max, Team, Enterprise, or Console account. The free Claude.ai plan does not include Claude Code access.

## Update Claude Code

Native installations automatically update in the background. Homebrew, WinGet, and Linux package manager installations require manual updates by default.

### Configure release channel

Control which release channel Claude Code follows with the `autoUpdatesChannel` setting: `"latest"` (default, receive new features as soon as released) or `"stable"` (about one week old, skips major regressions).

### Pin a minimum version

The `minimumVersion` setting establishes a floor so background auto-updates and `claude update` refuse to install any version below it.

### Disable auto-updates

Set `DISABLE_AUTOUPDATER` to `"1"` in your `settings.json`. To block all update paths, use `DISABLE_UPDATES` instead.

### Update manually

```bash
claude update
```

## Advanced installation options

### Install a specific version

The native installer accepts a specific version number or a release channel (`latest` or `stable`).

### Install with Linux package managers

Claude Code publishes signed apt, dnf, and apk repositories, each with `stable` and `latest` channels. All repositories are signed with the Claude Code release signing key.

### Install with npm

```bash
npm install -g @anthropic-ai/claude-code
```
Requires Node.js 18 or later. Do NOT use `sudo npm install -g`.

### Binary integrity and code signing

Each release publishes a signed `manifest.json` containing SHA256 checksums for every platform binary. macOS binaries are signed by "Anthropic PBC" and notarized by Apple; Windows binaries are signed by "Anthropic, PBC".

## Uninstall Claude Code

Remove the binary and version files for your installation method (native, Homebrew, WinGet, apt/dnf/apk, or npm). To remove Claude Code settings and cached data, delete `~/.claude`, `~/.claude.json`, and project-specific `.claude` and `.mcp.json` files. Note that removing configuration files deletes all your settings, allowed tools, MCP server configurations, and session history.

---

# 7. Get started with the desktop app

> Install Claude Code on desktop and start your first coding session

The desktop app gives you Claude Code with a graphical interface built for running multiple sessions side by side: a sidebar for managing parallel work, a drag-and-drop layout with an integrated terminal and file editor, visual diff review, live app preview, GitHub PR monitoring with auto-merge, and scheduled tasks. No terminal required.

Download for macOS (Universal build for Intel and Apple Silicon), Windows (x64), or Linux (beta, apt or .deb for Ubuntu and Debian). For Windows ARM64, download the ARM64 installer.

Claude Code requires a Pro, Max, Team, or Enterprise subscription.

The desktop app has three tabs:
- **Chat**: General conversation with no file access, similar to claude.ai
- **Cowork**: An autonomous background agent that works on tasks in a cloud VM with its own environment
- **Code**: An interactive coding assistant with direct access to your local files, where you review and approve each change in real time

## Install

1. Install and sign in: download the installer and run it, launch Claude, sign in with your Anthropic account
2. Open the Code tab: click the Code tab at the top center. If prompted to upgrade, subscribe to a paid plan first

The desktop app includes Claude Code. You don't need to install Node.js or the CLI separately.

## Start your first session

1. **Choose an environment and folder**: select Local to run Claude on your machine using your files directly, or Remote to run on Anthropic's cloud infrastructure, or SSH to connect to a remote machine
2. **Choose a model**: select a model from the dropdown next to the send button
3. **Tell Claude what to do**: type what you want Claude to do, e.g. "Find a TODO comment and fix it"
4. **Review and accept changes**: by default, the Code tab starts in Ask permissions mode, where Claude proposes changes and waits for your approval before applying them

## Now what?

- **Interrupt and steer**: click the stop button to interrupt immediately, or type a correction and press Enter
- **Give Claude more context**: type `@filename` to pull a specific file into the conversation, attach images and PDFs, or drag and drop files
- **Use skills for repeatable tasks**: type `/` or click + then Slash commands to browse built-in commands, custom skills, and plugin skills
- **Review changes before committing**: click the diff indicator to open the diff view, review modifications file by file, and comment on specific lines
- **Adjust how much control you have**: your permission mode controls the balance (Ask permissions, Auto accept edits, Plan mode)
- **Add plugins for more capabilities**: click + next to the prompt box and select Plugins
- **Arrange your workspace**: drag panes into whatever layout you want, open the terminal with Ctrl+`
- **Preview your app**: click the Preview dropdown to run your dev server directly in the desktop
- **Track your pull request**: Claude Code monitors CI check results and can automatically fix failures or merge the PR
- **Put Claude on a schedule**: set up scheduled tasks to run Claude automatically on a recurring basis
- **Scale up when you're ready**: open parallel sessions from the sidebar, each in its own Git worktree

## Coming from the CLI?

Desktop runs the same engine as the CLI with a graphical interface. You can run both simultaneously on the same project, and they share configuration (CLAUDE.md files, MCP servers, hooks, skills, and settings).

---

# 8. Get started with Claude Code on the web

> Run Claude Code in the cloud from your browser or phone. Connect a GitHub repository, submit a task, and review the PR without local setup.

Claude Code on the web is in research preview for Pro, Max, and Team users, and for Enterprise users with premium seats or Chat + Claude Code seats.

Claude Code on the web runs on Anthropic-managed cloud infrastructure instead of your machine. Submit tasks from claude.ai/code in your browser or the Claude mobile app. You'll need a GitHub repository. Claude clones it into an isolated virtual machine, makes changes, and pushes a branch for you to review.

Claude Code on the web works well for:
- **Parallel tasks**: run several independent tasks at once, each in its own session and branch
- **Repos you don't have locally**: Claude clones the repo fresh every session
- **Tasks that don't need frequent steering**: submit a well-defined task and review later
- **Code questions and exploration**: understand a codebase without a local checkout

## How sessions run

1. **Clone and prepare**: your repository is cloned to an Anthropic-managed VM, and your setup script runs if configured
2. **Configure network**: internet access is set based on your environment's access level
3. **Work**: Claude analyzes code, makes changes, runs tests, and checks its work
4. **Push the branch**: when Claude reaches a stopping point, it pushes its branch to GitHub

## Compare ways to run Claude Code

|  | On the web | Remote Control | Terminal CLI | Desktop app |
|---|---|---|---|---|
| Code runs on | Anthropic cloud VM | Your machine | Your machine | Your machine or cloud VM |
| You chat from | claude.ai or mobile app | claude.ai or mobile app | Your terminal | The Desktop UI |
| Uses your local config | No, repo only | Yes | Yes | Yes for local, no for cloud |
| Requires GitHub | Yes | No | No | Only for cloud sessions |
| Keeps running if you disconnect | Yes | While terminal stays open | No | Depends on session type |

## Connect GitHub and create an environment

1. Visit claude.ai/code and sign in with your Anthropic account
2. Install the Claude GitHub App and grant it access to your repositories
3. Create your environment: name, network access level (default Trusted), environment variables, and an optional setup script

### Connect from your terminal

If you already use the GitHub CLI (`gh`), run `/web-setup` in the Claude Code CLI. This reads your local `gh` token, links it to your Claude account, and creates a default cloud environment if you don't have one.

## Start a task

1. Select a repository and branch
2. Choose a permission mode: defaults to Accept edits, or switch to Plan mode
3. Describe the task and submit: be specific, name the file or function, paste error output if you have it

## Pre-fill sessions

You can prefill the prompt, repositories, and environment for a new session by adding query parameters (`prompt`, `prompt_url`, `repositories`, `environment`) to the claude.ai/code URL.

## Review and iterate

1. Open the diff view: a diff indicator shows lines added and removed
2. Leave inline comments: select any line in the diff and type your feedback
3. Create a pull request: select Create PR at the top of the diff view
4. Keep iterating after the PR: paste CI failure output or reviewer comments into the chat

## Next steps

- Use Claude Code on the web: the full reference
- Routines: automate work on a schedule
- CLAUDE.md: give Claude persistent instructions

---

# 9. Best practices for Claude Code

> Tips and patterns for getting the most out of Claude Code, from configuring your environment to scaling across parallel sessions.

Claude Code is an agentic coding environment. Unlike a chatbot that answers questions and waits, Claude Code can read your files, run commands, make changes, and autonomously work through problems while you watch, redirect, or step away entirely.

Most best practices are based on one constraint: Claude's context window fills up fast, and performance degrades as it fills.

## Give Claude a way to verify its work

Give Claude a check it can run: tests, a build, a screenshot to compare. Without a check it can run, "looks done" is the only signal available, and you become the verification loop. Give Claude something that produces a pass or fail, and the loop closes on its own.

| Strategy | Before | After |
|---|---|---|
| Provide verification criteria | "implement a function that validates email addresses" | "write a validateEmail function. example test cases: ... run the tests after implementing" |
| Verify UI changes visually | "make the dashboard look better" | "[paste screenshot] implement this design. take a screenshot of the result and compare it to the original" |
| Address root causes, not symptoms | "the build is failing" | "the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause" |

Once the check exists, decide how hard it gates the stop: in one prompt, across a session with `/goal`, as a deterministic Stop hook gate, or by a second opinion from a verification subagent.

## Explore first, then plan, then code

Letting Claude jump straight to coding can produce code that solves the wrong problem. Use plan mode to separate exploration from execution.

The recommended workflow has four phases:
1. **Explore**: enter plan mode, Claude reads files and answers questions without making changes
2. **Plan**: ask Claude to create a detailed implementation plan
3. **Implement**: switch out of plan mode and let Claude code, verifying against its plan
4. **Commit**: ask Claude to commit with a descriptive message and create a PR

Planning is most useful when you're uncertain about the approach, when the change modifies multiple files, or when you're unfamiliar with the code being modified. If you could describe the diff in one sentence, skip the plan.

## Provide specific context in your prompts

Claude can infer intent, but it can't read your mind. Reference specific files, mention constraints, and point to example patterns.

| Strategy | Before | After |
|---|---|---|
| Scope the task | "add tests for foo.py" | "write a test for foo.py covering the edge case where the user is logged out" |
| Point to sources | "why does ExecutionFactory have such a weird api?" | "look through ExecutionFactory's git history and summarize how its api came to be" |
| Reference existing patterns | "add a calendar widget" | "look at how existing widgets are implemented... follow the pattern" |
| Describe the symptom | "fix the login bug" | "users report that login fails after session timeout. check the auth flow..." |

### Provide rich content

Use `@` to reference files, paste screenshots/images, give URLs for documentation, pipe in data, or let Claude fetch what it needs itself.

## Configure your environment

### Write an effective CLAUDE.md

Run `/init` to generate a starter CLAUDE.md file based on your current project structure. CLAUDE.md is a special file that Claude reads at the start of every conversation. Include Bash commands, code style, and workflow rules.

Keep it concise. For each line, ask: "Would removing this cause Claude to make mistakes?" If not, cut it.

| Include | Exclude |
|---|---|
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Code style rules that differ from defaults | Standard language conventions Claude already knows |
| Testing instructions and preferred test runners | Detailed API documentation (link to docs instead) |
| Repository etiquette | Information that changes frequently |
| Architectural decisions specific to your project | Long explanations or tutorials |

CLAUDE.md files can import additional files using `@path/to/import` syntax, and can be placed in the home folder, project root, `.local.md` variant, parent directories, or child directories.

### Configure permissions

Three ways to reduce permission interruptions: Auto mode (classifier reviews commands), permission allowlists (permit specific safe tools), and sandboxing (OS-level isolation).

### Use CLI tools

Tell Claude Code to use CLI tools like `gh`, `aws`, `gcloud`, and `sentry-cli` when interacting with external services.

### Connect MCP servers

Run `claude mcp add` to connect external tools like Notion, Figma, or your database.

### Set up hooks

Use hooks for actions that must happen every time with zero exceptions. Hooks run scripts automatically at specific points in Claude's workflow, and unlike CLAUDE.md instructions, hooks are deterministic.

### Create skills

Create `SKILL.md` files in `.claude/skills/` to give Claude domain knowledge and reusable workflows.

### Create custom subagents

Define specialized assistants in `.claude/agents/` that Claude can delegate to for isolated tasks.

### Install plugins

Run `/plugin` to browse the marketplace. Plugins bundle skills, hooks, subagents, and MCP servers into a single installable unit.

## Communicate effectively

### Ask codebase questions

When onboarding to a new codebase, ask Claude the same sorts of questions you would ask another engineer.

### Let Claude interview you

For larger features, have Claude interview you first using the AskUserQuestion tool, then write a complete spec before starting implementation in a fresh session.

## Manage your session

### Course-correct early and often

The best results come from tight feedback loops. Use Esc to stop Claude mid-action, Esc+Esc or `/rewind` to restore previous state, or `/clear` to reset context between unrelated tasks.

### Manage context aggressively

Run `/clear` frequently between tasks. For more control, run `/compact <instructions>`. Use `/btw` for quick questions that don't need to stay in context.

### Use subagents for investigation

Delegate research with "use subagents to investigate X". They explore in a separate context, keeping your main conversation clean.

### Rewind with checkpoints

Every prompt you send creates a checkpoint. Double-tap Escape or run `/rewind` to open the rewind menu.

### Resume conversations

Run `claude --continue` to pick up the most recent session, or `claude --resume` to choose from a list. Give sessions descriptive names like `oauth-migration`.

## Automate and scale

### Run non-interactive mode

Use `claude -p "prompt"` in CI, pre-commit hooks, or scripts. Output formats include plain text, JSON, or streaming JSON.

### Run multiple Claude sessions

Pick the parallel approach that fits: Worktrees, Desktop app, Claude Code on the web, or Agent teams. Use a Writer/Reviewer pattern for quality-focused workflows.

### Fan out across files

For large migrations, generate a task list, write a script to loop through it calling `claude -p` for each file, and test on a few files before running at scale.

### Run autonomously with auto mode

```bash
claude --permission-mode auto -p "fix all lint errors"
```

### Add an adversarial review step

Before treating a task as done, have a subagent review the diff in a fresh context and report gaps. Run the bundled `/code-review` skill, or write your own review prompt.

## Avoid common failure patterns

- **The kitchen sink session**: context is full of irrelevant information. Fix: `/clear` between unrelated tasks
- **Correcting over and over**: context is polluted with failed approaches. Fix: after two failed corrections, `/clear` and write a better prompt
- **The over-specified CLAUDE.md**: Claude ignores half of it. Fix: ruthlessly prune
- **The trust-then-verify gap**: implementation doesn't handle edge cases. Fix: always provide verification
- **The infinite exploration**: Claude reads hundreds of files. Fix: scope investigations narrowly or use subagents

## Develop your intuition

The patterns in this guide aren't set in stone. Pay attention to what works, and over time you'll develop intuition that no guide can capture.
