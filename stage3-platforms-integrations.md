# 第三階段, 使用介面與平台整合

在不同環境（編輯器, 桌面, 瀏覽器, 行動裝置）中使用 Claude Code.

---

# 1. Platforms and integrations

> Choose where to run Claude Code and what to connect it to.

Claude Code runs the same underlying engine everywhere, but each surface is tuned for a different way of working.

## Where to run Claude Code

| Platform | Best for | What you get |
|---|---|---|
| CLI | Terminal workflows, scripting, remote servers | Full feature set, Agent SDK, computer use on macOS (Pro/Max), third-party providers |
| Desktop | Visual review, parallel sessions, managed setup | Diff viewer, app preview, computer use and Dispatch on Pro/Max |
| VS Code | Working inside VS Code without switching to a terminal | Inline diffs, integrated terminal, file context |
| JetBrains | Working inside IntelliJ, PyCharm, WebStorm | Diff viewer, selection sharing, terminal session |
| Web | Long-running tasks that don't need much steering | Anthropic-managed cloud, continues after disconnect |
| Mobile | Starting and monitoring tasks while away | Cloud sessions, Remote Control, Dispatch to Desktop on Pro/Max |

The CLI is the most complete surface for terminal-native work: scripting and the Agent SDK are CLI-only. You can mix surfaces on the same project; configuration, project memory, and MCP servers are shared across local surfaces.

## Connect your tools

| Integration | What it does | Use it for |
|---|---|---|
| Chrome | Controls your browser with your logged-in sessions | Testing web apps, filling forms, automating sites without an API |
| GitHub Actions | Runs Claude in your CI pipeline | Automated PR reviews, issue triage, scheduled maintenance |
| GitLab CI/CD | Same as GitHub Actions for GitLab | CI-driven automation on GitLab |
| Code Review | Reviews every PR automatically | Catching bugs before human review |
| Slack | Responds to @Claude mentions in your channels | Turning bug reports into pull requests from team chat |

## Work when you are away from your terminal

| Option | Trigger | Claude runs on | Best for |
|---|---|---|---|
| Dispatch | Message a task from the Claude mobile app | Your machine (Desktop) | Delegating work while away, minimal setup |
| Remote Control | Drive a running session from claude.ai/code or mobile | Your machine (CLI or VS Code) | Steering in-progress work from another device |
| Channels | Push events from a chat app or your own server | Your machine (CLI) | Reacting to external events like CI failures |
| Slack | Mention @Claude in a team channel | Anthropic cloud | PRs and reviews from team chat |
| Scheduled tasks | Set a schedule | CLI, Desktop, or cloud | Recurring automation like daily reviews |

---

# 2. Use Claude Code in VS Code

> Install and configure the Claude Code extension for VS Code.

The VS Code extension provides a native graphical interface integrated directly into your IDE: review and edit plans before accepting, auto-accept edits, @-mention files with line ranges, access conversation history, and open multiple conversations in tabs.

## Prerequisites

VS Code 1.98.0 or higher, and any paid Claude subscription (Pro, Max, Team, Enterprise) or Claude Console account; no API key required.

## Install the extension

Click "Install for VS Code" or "Install for Cursor", or press Cmd/Ctrl+Shift+X, search "Claude Code", and click Install.

## Get started

1. **Open the Claude Code panel**: click the Spark icon in the Editor Toolbar, or use the Activity Bar, Command Palette, or Status Bar
2. **Sign in**: click Sign in and complete authorization in your browser
3. **Send a prompt**: Claude automatically sees your selected text; press Option+K/Alt+K to insert an @-mention reference
4. **Review changes**: Claude shows a side-by-side diff and asks for permission before editing

## Use the prompt box

- **Permission modes**: click the mode indicator to switch between normal, Plan mode, and auto-accept mode
- **Command menu**: click `/` to attach files, switch models, toggle extended thinking, view `/usage`, start Remote Control
- **Context indicator**: shows context window usage
- **Multi-line input**: Shift+Enter adds a new line

### Reference files and folders

Type `@` followed by a file or folder name; supports fuzzy matching. Selected editor text is automatically visible to Claude; press Option+K/Alt+K to insert `@app.ts#5-10`.

### Resume past conversations

Click Session history at the top of the panel to search or browse by time. Cloud sessions from Claude Code on the web appear under the Remote tab if signed in with a Claude.ai subscription.

## Customize your workflow

Drag the Claude panel to the secondary sidebar, primary sidebar, or editor area. Use Open in New Tab / New Window for parallel conversations. Switch to terminal-style interface via the Use Terminal setting.

## Manage plugins

Type `/plugins` to open the plugin dialog with Plugins and Marketplaces tabs. Choose install scope: for you (user), for this project, or locally.

## Automate browser tasks with Chrome

Type `@browser` followed by an instruction to control Chrome from VS Code, requires the Claude in Chrome extension.

## Key VS Code commands and shortcuts

| Command | Shortcut | Description |
|---|---|---|
| Focus Input | Cmd/Ctrl+Esc | Toggle focus between editor and Claude |
| Open in New Tab | Cmd/Ctrl+Shift+Esc | Open a new conversation as an editor tab |
| Reopen Closed Session | Cmd/Ctrl+Shift+T | Reopen the most recently closed Claude session tab |
| Insert @-Mention Reference | Option/Alt+K | Insert a reference to the current file and selection |

## VS Code extension vs. Claude Code CLI

| Feature | CLI | VS Code Extension |
|---|---|---|
| Commands and skills | All | Subset |
| MCP server config | Yes | Partial |
| Checkpoints | Yes | Yes |
| `!` bash shortcut | Yes | No |
| Tab completion | Yes | No |

### Rewind with checkpoints

Hover over any message to reveal the rewind button: Fork conversation from here, Rewind code to here, or Fork conversation and rewind code.

### Run CLI in VS Code

Open the integrated terminal and run `claude`; requires the standalone CLI install since the extension bundles a private copy only for its chat panel.

## Configure settings

Extension settings live in VS Code preferences; shared Claude Code settings live in `~/.claude/settings.json`. Key settings include `useTerminal`, `initialPermissionMode`, `preferredLocation`, `autosave`, and `allowDangerouslySkipPermissions`.

## Security and privacy

Enable VS Code Restricted Mode for untrusted workspaces; use manual approval mode instead of auto-accept; review changes before accepting.

---

# 3. JetBrains IDEs

> Use Claude Code with JetBrains IDEs including IntelliJ, PyCharm, WebStorm, and more.

Supported IDEs include IntelliJ IDEA, PyCharm, Android Studio, WebStorm, PhpStorm, and GoLand.

## Features

- **Quick launch**: Cmd/Ctrl+Esc opens Claude Code directly from your editor
- **Diff viewing**: code changes display in the IDE diff viewer
- **Selection context**: current selection or tab is automatically shared with Claude Code
- **File reference shortcuts**: Cmd+Option+K / Alt+Ctrl+K inserts file references like `@src/auth.ts#L1-99`
- **Diagnostic sharing**: lint and syntax errors are automatically shared with Claude

## Installation

1. Install the Claude Code CLI following the quickstart
2. Install the Claude Code plugin from the JetBrains Marketplace and restart your IDE

Claude Code works with any paid Claude subscription or Claude Console account.

## Usage

Run `claude` from your IDE's integrated terminal to activate all integration features. From external terminals, use `/ide` to connect.

## Configuration

Set the diff tool to `auto` (IDE) or `terminal` via `/config`. Plugin settings under Settings → Tools → Claude Code [Beta] include Claude command path, Option+Enter for multi-line prompts (macOS), and automatic updates.

## Special configurations

**Remote development**: install the plugin on the remote host, not the local client.

**WSL configuration**: if "No available IDEs detected" appears, it's usually WSL2's NAT networking or Windows Firewall blocking the connection. Fix by allowing WSL2 traffic through Windows Firewall, or switch WSL2 to mirrored networking (Windows 11 22H2+).

## Security considerations

In `acceptEdits` mode, Claude may modify IDE configuration files that can auto-execute. Consider manual approval mode and being aware of which files Claude can modify.

---

# 4. Desktop application

> Get more out of Claude Code Desktop: parallel sessions, drag-and-drop panes, computer use, Dispatch, visual diff review, app previews, PR monitoring, and enterprise configuration.

The Claude Desktop app has three tabs: Chat, Cowork (Dispatch and longer agentic work), and Code (software development, this page's reference).

## Start a session

Configure four things: **Environment** (Local/Remote/SSH), **Project folder**, **Model**, and **Permission mode**. Type your task and press Enter.

## Choose a permission mode

| Mode | Settings key | Behavior |
|---|---|---|
| Ask permissions | `default` | Claude asks before editing files or running commands |
| Auto accept edits | `acceptEdits` | Auto-accepts file edits and common filesystem commands |
| Plan mode | `plan` | Claude explores and proposes a plan without editing source |
| Auto | `auto` | Executes all actions with background safety checks |
| Bypass permissions | `bypassPermissions` | Runs without permission prompts; sandboxed environments only |

Cloud sessions support Accept edits, Plan mode, and Auto mode only.

## Preview your app

Claude starts a dev server and opens an embedded browser to verify changes: takes screenshots, inspects the DOM, clicks elements, fills forms, and fixes issues automatically (auto-verify).

## Review changes with diff view

A diff stats indicator like `+12 -1` opens the diff viewer. Click any line to comment; press Cmd/Ctrl+Enter to submit comments. Click **Review code** to have Claude self-evaluate the diff for high-signal issues.

## Monitor pull request status

Auto-fix attempts to fix failing CI checks; Auto-merge merges the PR once checks pass (squash merge). Requires the GitHub CLI (`gh`) installed and authenticated.

## Arrange your workspace

Panes include chat, diff, preview, terminal, file, plan, tasks, and subagent. Drag to reposition or resize; press Cmd/Ctrl+\ to close the focused pane.

## Let Claude use your computer

Computer use is a research preview on macOS and Windows (Pro/Max only), off by default. Enable in Settings → General, then grant Accessibility and Screen Recording permissions on macOS.

### App permission tiers

| Tier | What Claude can do | Applies to |
|---|---|---|
| View only | See the app in screenshots | Browsers, trading platforms |
| Click only | Click and scroll, not type | Terminals, IDEs |
| Full control | Click, type, drag, keyboard shortcuts | Everything else |

## Manage sessions

Click + New session or Cmd/Ctrl+N to work on multiple tasks in parallel; each session gets its own Git worktree. Cloud sessions (Remote) continue even if you close the app. SSH sessions run on a remote machine you manage.

## Coming from the CLI

Desktop runs the same engine with a graphical interface; both share CLAUDE.md, MCP servers, hooks, skills, and settings. Run `/desktop` in the terminal to move a CLI session into Desktop.

### Feature comparison highlights

| Feature | CLI | Desktop |
|---|---|---|
| Permission modes | All including `dontAsk` | Ask, Auto accept, Plan, Auto, Bypass |
| Session isolation | `--worktree` flag | Automatic worktrees |
| Computer use | Enable via `/mcp`, macOS only | App and screen control, macOS and Windows |
| Scripting and automation | `--print`, Agent SDK | Not available |
| Agent teams | Available in CLI | Not available; use dynamic workflows instead |

---

# 5. Claude Desktop on Linux (beta)

> Install and update the Claude desktop app on Ubuntu and Debian.

Linux support is in beta; Chat, Cowork, and Code tabs are all available.

## Requirements

Ubuntu 22.04+ or Debian 12+, x86_64 or arm64.

## Install

1. Add Anthropic's apt repository (download signing key, register repository)
2. `sudo apt update && sudo apt install claude-desktop`
3. Launch **Claude** from your application launcher or run `claude-desktop`, sign in

Alternatively, download the `.deb` package and install with `sudo apt install ./claude-desktop_*.deb`, though this won't auto-update through apt.

## Update

```bash
sudo apt update && sudo apt upgrade
```

## Uninstall

```bash
sudo apt remove claude-desktop
sudo rm /etc/apt/sources.list.d/claude-desktop.list
```

## What's not in the Linux beta yet

- **Computer Use**: not available on Linux
- **Dictation**: voice input not available; use voice dictation in the CLI instead
- **Quick Entry global hotkey**: works on X11; native Wayland requires the desktop environment's GlobalShortcuts portal
- **Fedora and RHEL**: only Debian-based distributions supported today

---

# 6. Use Claude Code with Chrome (beta)

> Connect Claude Code to your Chrome browser to test web apps, debug with console logs, automate form filling, and extract data from web pages.

Claude opens new tabs for browser tasks and shares your browser's login state. Browser actions run in a visible Chrome window in real time; Claude pauses for login pages or CAPTCHAs.

Works with Google Chrome and Microsoft Edge only; not Brave, Arc, or WSL.

## Capabilities

Live debugging, design verification, web app testing, authenticated web apps (Gmail, Notion, Google Docs), data extraction, task automation, and session recording as GIFs.

## Prerequisites

Chrome or Edge, Claude in Chrome extension 1.0.36+, Claude Code v2.0.73+, and a direct Anthropic plan (Pro, Max, Team, Enterprise).

## Get started in the CLI

```bash
claude --chrome
```

Or run `/chrome` within an existing session. Then ask Claude to use the browser, e.g. "Go to code.claude.com/docs, click the search box, type 'hooks'".

Run `/chrome` and select "Enabled by default" to avoid passing the flag each session (uses more context since browser tools are always loaded).

## Example workflows

Testing a local web app, debugging with console logs, automating form filling from a spreadsheet, drafting content in Google Docs, extracting structured data, multi-site workflows, and recording demo GIFs.

## Troubleshooting

Common errors: "Browser extension is not connected" (restart Chrome and Claude Code, run `/chrome`), "Extension not detected" (install/enable in chrome://extensions), "No tab available" (ask Claude to create a new tab), "Receiving end does not exist" (extension service worker went idle, reconnect).

---

# 7. Let Claude use your computer from the CLI

> Enable computer use in the Claude Code CLI so Claude can open apps, click, type, and see your screen on macOS.

Computer use is a research preview on macOS requiring a Pro or Max plan, Claude Code v2.1.85+, and an interactive session (not available with `-p`).

## What you can do

Build and validate native apps, end-to-end UI testing, debug visual and layout issues, and drive GUI-only tools like design tools or the iOS Simulator.

## When computer use applies

Claude tries the most precise tool first: MCP server, then Bash, then Claude in Chrome, then computer use as the broadest and slowest option, reserved for native apps and tools without an API.

## Enable computer use

1. Run `/mcp`, find `computer-use` in the server list (disabled by default)
2. Select and choose **Enable** (persists per project)
3. Grant macOS permissions: Accessibility (click, type, scroll) and Screen Recording (see the screen)

## Approve apps per session

The first time Claude needs a specific app, a prompt shows which apps, extra permissions, and how many other apps will be hidden. Choose **Allow for this session** or **Deny**. Apps with broad reach (Terminal, Finder, System Settings) show extra warnings.

## How Claude works on your screen

Computer use holds a machine-wide lock from first action until the session exits (not just task completion). Other visible apps are hidden while Claude works; your terminal stays visible and excluded from screenshots. Screenshots are automatically downscaled. Press `Esc` anywhere to abort immediately.

## Safety and the trust boundary

Per-app approval, sentinel warnings for high-reach apps, terminal excluded from screenshots, global escape key, and a lock file limiting one session at a time.

---

# 8. Use Claude Code on the web

> Configure cloud environments, setup scripts, network access, and Docker in Anthropic's sandbox.

Claude Code on the web runs tasks on Anthropic-managed cloud infrastructure at claude.ai/code. Sessions persist even if you close your browser.

## GitHub authentication options

| Method | How it works | Best for |
|---|---|---|
| GitHub App | Authorize during web onboarding | Browser onboarding, teams wanting Auto-fix |
| `/web-setup` | Sync local `gh` CLI token | Individual developers who already use `gh` |

## The cloud environment

Available in cloud sessions: your repo's CLAUDE.md, `.claude/settings.json` hooks, `.mcp.json` MCP servers, `.claude/rules/`, skills/agents/commands, plugins declared in settings, and server-managed settings. NOT available: your user `~/.claude/` files, MCP servers added with `claude mcp add`, static API tokens, or interactive auth like AWS SSO.

## Installed tools

Python, Node.js (20/21/22), Ruby, PHP, Java, Go, Rust, C/C++, Docker, PostgreSQL, Redis, and common utilities pre-installed.

## Resource limits

Approximately 4 vCPUs, 16 GB RAM, 30 GB disk.

## Setup scripts

A Bash script that runs when a new cloud session starts, before Claude Code launches, running as root on Ubuntu 24.04. Keep runtime under roughly five minutes; the environment is cached after the first successful run.

## Network access

| Level | Outbound connections |
|---|---|
| None | No outbound network access |
| Trusted | Allowlisted domains only: package registries, GitHub, cloud SDKs |
| Full | Any domain |
| Custom | Your own allowlist, optionally including defaults |

## Move tasks between web and terminal

```bash
claude --remote "Fix the authentication bug in src/auth/login.ts"
```
Creates a new cloud session. Use `--teleport` to pull a cloud session into your terminal, or `/teleport` inside an existing session.

## Auto-fix pull requests

Claude watches a PR and automatically responds to CI failures and review comments, requires the Claude GitHub App installed. Turn on via the CI status bar, `/autofix-pr`, or by telling Claude to watch a PR.

## Security and isolation

Isolated VMs per session, network access controls, credential protection through a secure proxy with scoped credentials.

---

# 9. Continue local sessions from any device with Remote Control

> Continue a local Claude Code session from your phone, tablet, or any browser using Remote Control.

Remote Control connects claude.ai/code or the Claude mobile app to a session running on your machine; Claude keeps running locally, nothing moves to the cloud.

## Requirements

Pro, Max, Team, or Enterprise plan (API keys not supported). Not available on Bedrock, Vertex AI, or Foundry.

## Start a Remote Control session

| Mode | Command |
|---|---|
| Server mode | `claude remote-control` |
| Interactive session | `claude --remote-control` or `--rc` |
| From an existing session | `/remote-control` or `/rc` |
| VS Code | Type `/remote-control` in the prompt box |

Server mode flags include `--name`, `--spawn <same-dir/worktree/session>`, `--capacity <N>`, and `--sandbox`.

## Connect from another device

Open the session URL in any browser, scan the QR code, or find the session by name in the session list.

## Trusted Devices

An organization-wide setting (Team/Enterprise) requiring device enrollment and a recent sign-in (within 18 hours) before a member can view or steer Remote Control sessions. Uses Face ID, Touch ID, Windows Hello, or a passkey for biometric step-up.

## Remote Control vs Claude Code on the web

Remote Control executes on your machine, keeping local MCP servers and tools available. Claude Code on the web executes in Anthropic-managed cloud infrastructure. Use Remote Control to continue local work from another device; use the web to kick off tasks without local setup.

## Mobile push notifications

Claude decides when to push, typically when a long task finishes or needs a decision. Enable via `/config`: "Push when Claude decides" and "Push when actions required".

---

# 10. Claude Code in Slack

> Delegate coding tasks directly from your Slack workspace.

When you mention @Claude with a coding task, Claude detects the intent and creates a Claude Code session on the web under your own account, using your connected repositories and plan limits.

## Use cases

Bug investigation and fixes, quick code reviews, collaborative debugging using thread context, and parallel task execution.

## Prerequisites

Pro/Max/Team/Enterprise plan with Claude Code access, Claude Code on the web enabled, a connected GitHub account, and Slack authentication linked to your Claude account.

## Setting up

1. A workspace admin installs the Claude app from the Slack App Marketplace
2. Connect your individual Claude account via the App Home tab
3. Configure Claude Code on the web at claude.ai/code and connect GitHub
4. Choose a routing mode: **Code only** or **Code + Chat** (intelligently routes between coding and general questions)
5. Invite Claude to channels with `/invite @Claude` (not automatic, and does not work in DMs)

## How it works

Claude automatically detects coding intent from your @mention, gathers context from the thread or channel, creates a session on claude.ai/code, posts progress updates, and @mentions you with a summary and action buttons (View Session, Create PR) when done.

## Access and permissions

Sessions run under your own account and count against your individual usage limits. Channel-based access control: Claude only responds where explicitly invited.

---

# 11. Voice dictation

> Speak your prompts in the Claude Code CLI with hold-to-record or tap-to-record voice dictation.

Your speech is transcribed live into the prompt input; you can mix voice and typing in the same message. Requires a Claude.ai account (not available via API key, Bedrock, Vertex, or Foundry), a local microphone (not remote environments), and an organization without HIPAA compliance enabled.

## Enable voice dictation

```text
/voice
```

| Command | Effect |
|---|---|
| `/voice` | Toggle on or off, keep current mode |
| `/voice hold` | Enable in hold mode |
| `/voice tap` | Enable in tap mode |
| `/voice off` | Disable |

## Hold to record

Push-to-talk: hold Space to start recording, release to stop and finalize. There's a brief warmup for key-repeat detection. Set `"autoSubmit": true` to send automatically on release if the transcript is at least three words.

## Tap to record and send

Tap once to start (only when input is empty), speak, tap again to stop. Auto-submits if the transcript is at least three words long; no warmup needed.

## Change the dictation language

Uses the same `language` setting that controls Claude's response language, supporting 20 languages including English, Japanese, Spanish, French, German, and more.

## Rebind the dictation key

Bound to `voice:pushToTalk` in the Chat context, defaults to Space; rebind in `~/.claude/keybindings.json`.

---

# 12. Launch sessions from links

> Open a Claude Code terminal session from a URL.

A deep link is a `claude-cli://` URL that opens Claude Code in a new terminal window, carrying a working directory and a prompt to pre-fill. The prompt is populated but not sent until you press Enter.

## Build a link

```text
claude-cli://open
```

| Parameter | Description |
|---|---|
| `q` | Text to pre-fill in the prompt box, URL-encoded, max 5,000 characters |
| `cwd` | Absolute path to use as the working directory |
| `repo` | A GitHub owner/name slug resolved to a local clone Claude Code has seen before |

Example:
```text
claude-cli://open?repo=acme/payments&q=Investigate%20the%20failed%20deploy
```

`cwd` takes precedence over `repo` if both are passed. Use `cwd` for standardized paths (devcontainers); use `repo` when everyone clones to a different location.

## What a launched session shows

A deep link never executes anything on its own; the prompt is inert until you press Enter. A warning line reads "Prompt from an external link" until sent or cleared.

## Registration and supported platforms

Claude Code registers the `claude-cli://` handler with your OS the first time you start an interactive session, at user-level locations only. To prevent registration, set `disableDeepLinkRegistration` to `"disable"` in settings.json.

---

# 13. Share session output as artifacts

> Artifacts turn Claude Code's work into live, interactive pages at a private URL you can share inside your organization.

An artifact is a live, interactive web page Claude Code publishes from your session to a private URL on claude.ai, updating in place as the session continues. Requires Team or Enterprise plan and sign-in with `/login`.

## When to use an artifact

Walk a reviewer through a PR with annotated diffs, render a dashboard, lay out design options side by side, keep an investigation timeline, or send a teammate a link instead of pasting output.

An artifact is a capture of work, not an application: one self-contained page with no backend.

## Create an artifact

Ask in plain language, e.g. "Make an artifact that walks through this PR with the diff annotated inline." Claude writes an HTML or Markdown file, asks for permission, then publishes. Press `Ctrl+]` to reopen the most recent artifact.

## Update and share

Ask Claude to revise the page; each publish becomes a new version. Use the **Share** control in the page header to grant access to specific people or everyone in your organization. Sharing stops at your organization boundary.

## Page constraints

| Constraint | Effect |
|---|---|
| No external requests | CSP blocks external scripts, styles, fonts, images, fetch, XHR, WebSocket |
| No backend | Cannot store form data, authenticate viewers, or call an API at view time |
| Single page | Relative links don't resolve; uses in-page anchors instead |
| Source file types | Must be `.html`, `.htm`, or `.md` |
| Rendered size | Must be 16 MiB or smaller |

## Availability

Requires Team or Enterprise plan, sign-in via `/login` (not API key or cloud provider credential), Anthropic API model provider, and no CMEK/HIPAA/Zero Data Retention policy conflicts.

## Disable artifacts

Set `"disableArtifact": true` in settings, `CLAUDE_CODE_DISABLE_ARTIFACT=1`, or add `Artifact` to `permissions.deny`.
