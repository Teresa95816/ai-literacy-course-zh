# 第四階段, 擴充能力, Skills, Hooks, 子代理與 MCP

學習如何客製化與擴充 Claude 的能力.

---

# 1. Extend Claude Code

> Understand when to use CLAUDE.md, Skills, subagents, hooks, MCP, and plugins.

擴充功能插入代理迴圈的不同部分 : CLAUDE.md 加入持久性 context, Skills 加入可重複使用的知識與可呼叫的工作流程, MCP 連接外部服務, Subagents 在隔離的 context 中執行, Hooks 於生命週期事件觸發, Plugins 打包並分發以上功能.

## 依目標選擇功能

| 功能 | 做什麼 | 何時使用 |
|---|---|---|
| CLAUDE.md | 每次對話都載入的持久性 context | 專案慣例, "一律要做 X" 的規則 |
| Skill | Claude 可使用的指示, 知識與工作流程 | 可重複使用的內容, 參考文件, 可重複的任務 |
| Subagent | 隔離的執行 context, 回傳摘要結果 | context 隔離, 平行任務, 專門的工作者 |
| Agent teams | 協調多個獨立的 Claude Code session | 平行研究, 新功能開發, 用不同假設除錯 |
| MCP | 連接外部服務 | 外部資料或動作 |
| Hook | 由事件觸發的腳本, HTTP 請求, 提示詞或子代理 | 必須在每次符合的事件發生時執行的自動化 |

Plugins 是封裝層, 把 skills, hooks, subagents 與 MCP 伺服器打包成單一可安裝單元.

## 逐步建立你的設定

| 觸發時機 | 該新增什麼 |
|---|---|
| Claude 把慣例或指令弄錯兩次 | 加進 CLAUDE.md |
| 你一直輸入同樣的提示詞來開始任務 | 存成使用者可呼叫的 skill |
| 你第三次把同樣的作業程序貼進對話 | 寫成 skill |
| 你一直從 Claude 看不到的瀏覽器分頁複製資料 | 把該系統接成 MCP 伺服器 |
| Claude 讀很多檔案才能找到符號定義 | 為你的語言安裝 code intelligence plugin |
| 旁支任務灌爆你的對話, 且內容不會再被參考 | 透過 subagent 執行 |
| 你希望某件事每次都自動發生, 不需要詢問 | 寫一個 hook |
| 第二個儲存庫也需要同樣的設定 | 打包成 plugin |

## 比較容易混淆的功能

**Skill vs Subagent** : Skill 是可重複使用的內容, 載入任何 context; Subagent 是隔離的工作者, 獨立於主對話執行. Skill 可以是參考內容或任務內容; Subagent 適合會讀很多檔案的任務.

**CLAUDE.md vs Skill** : CLAUDE.md 每個 session 自動載入; Skill 依需求載入, 可用 `/<name>` 觸發工作流程.

**Hook vs Skill** : Hook 保證在其事件上觸發, 適合每次都要一樣執行的動作（如編輯後檢查, 封鎖不安全指令）; Skill 讓 Claude 自行判斷如何套用步驟.

**MCP vs Skill** : MCP 提供工具與資料存取; Skill 提供如何有效使用這些工具的知識.

## 理解 context 成本

| 功能 | 何時載入 | 載入什麼 | context 成本 |
|---|---|---|---|
| CLAUDE.md | Session 開始時 | 完整內容 | 每次請求 |
| Skills | Session 開始 + 使用時 | 開始時載入描述, 使用時載入完整內容 | 低（描述部分) |
| MCP 伺服器 | Session 開始時 | 工具名稱, 完整 schema 依需求載入 | 低, 直到工具被使用 |
| Subagents | 產生時 | 全新 context, 含指定的 skills | 與主 session 隔離 |
| Hooks | 觸發時 | 無（在外部執行) | 零, 除非 hook 回傳額外 context |

CLAUDE.md 建議控制在 200 行以內. 對於只想讓自己觸發, 不想讓 Claude 自動載入的 skill, 設定 `disable-model-invocation: true` 可將 context 成本降到零.

---

# 2. Extend Claude with skills

> Create, manage, and share skills to extend Claude's capabilities in Claude Code.

建立一個含有指示的 `SKILL.md` 檔案, Claude 就會把它加進工具箱. Claude 在相關時會使用 skill, 你也可以直接用 `/skill-name` 呼叫.

## 內建 skills

Claude Code 內含一組隨附的 skills, 在每個 session 中都可使用（除非用 `disableBundledSkills` 停用), 包括 `/code-review`, `/batch`, `/debug`, `/loop`, `/claude-api`.

## Skill 存放位置

| 位置 | 路徑 | 適用範圍 |
|---|---|---|
| Enterprise | 見 managed settings | 組織內所有使用者 |
| 個人 | `~/.claude/skills/<skill-name>/SKILL.md` | 你所有的專案 |
| 專案 | `.claude/skills/<skill-name>/SKILL.md` | 僅此專案 |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` | Plugin 啟用之處 |

## Frontmatter 參考

| 欄位 | 說明 |
|---|---|
| `description` | Skill 的用途與使用時機, Claude 用此判斷何時套用 |
| `disable-model-invocation` | 設為 true 只讓你能呼叫, Claude 不會自動觸發 |
| `user-invocable` | 設為 false 從 / 選單隱藏, 只有 Claude 能呼叫 |
| `allowed-tools` | Skill 啟用時 Claude 不需詢問即可使用的工具 |
| `context` | 設為 fork 在被分岔的 subagent context 中執行 |
| `paths` | 限制此 skill 只在符合的檔案類型時自動啟用 |

## 控制誰能呼叫 skill

- `disable-model-invocation: true` : 只有你能呼叫（適合有副作用的工作流程, 如 /deploy）
- `user-invocable: false` : 只有 Claude 能呼叫（適合背景知識, 如 legacy-system-context）

## 動態注入 context

`` !`<command>` `` 語法會在 skill 內容送到 Claude 之前先執行 shell 指令, 指令輸出會取代該處的文字. 例如把 `git diff HEAD` 的結果直接嵌入 skill 提示中.

## 在 subagent 中執行 skill

加上 `context: fork` 讓 skill 在隔離環境中執行, skill 內容變成驅動 subagent 的提示詞.

---

# 3. Create custom subagents

> Create and use specialized AI subagents in Claude Code for task-specific workflows and improved context management.

Subagent 是專門處理特定任務類型的 AI 助理, 各自在獨立的 context window 中執行, 有自訂系統提示, 特定工具存取權與獨立權限.

## 內建 subagents

| Subagent | 模型 | 用途 |
|---|---|---|
| Explore | Haiku | 快速唯讀, 搜尋與分析程式碼庫 |
| Plan | 繼承主對話 | Plan mode 中的研究代理 |
| general-purpose | 繼承主對話 | 需要探索與行動的複雜多步驟任務 |

## 建立第一個 subagent

執行 `/agents` 開啟子代理介面, 選擇 Personal 儲存位置, 用 Generate with Claude 產生, 選擇工具（唯讀審閱者只保留唯讀工具), 選擇模型與顏色, 設定記憶範圍後儲存.

## Subagent 儲存範圍

| 位置 | 範圍 | 優先權 |
|---|---|---|
| Managed settings | 組織全體 | 1（最高） |
| `--agents` CLI 旗標 | 目前 session | 2 |
| `.claude/agents/` | 目前專案 | 3 |
| `~/.claude/agents/` | 你所有的專案 | 4 |
| Plugin 的 `agents/` 目錄 | Plugin 啟用之處 | 5（最低） |

## 重要 frontmatter 欄位

`name`, `description`（必填), `tools`, `disallowedTools`, `model`, `permissionMode`, `skills`（預先載入的 skills), `mcpServers`, `memory`（持久記憶範圍：user/project/local), `isolation: worktree`（獨立 git worktree）.

## 明確呼叫 subagent

用自然語言提及名稱, 用 `@` 提及選取, 或用 `claude --agent <name>` 讓整個 session 都採用該 subagent 的系統提示, 工具限制與模型.

## Fork 目前對話

Fork 是繼承整個對話歷史的 subagent, 而非從零開始. 用 `/fork` 加上指示啟動, fork 只有最終結果回到主對話, 中間工具呼叫不會污染主 context.

---

# 4. Automate actions with hooks

> Run shell commands automatically when Claude Code edits files, finishes tasks, or needs input.

Hooks 是使用者定義的 shell 指令, 在 Claude Code 生命週期的特定時間點執行, 提供確定性的控制, 確保某些動作一定會發生.

## 常見自動化範例

- 取得完成通知（Notification hook)
- 編輯後自動格式化程式碼（PostToolUse + Edit|Write matcher)
- 封鎖對受保護檔案的編輯（PreToolUse, exit code 2)
- 壓縮後重新注入 context（SessionStart, matcher: compact)
- 稽核設定變更（ConfigChange event)
- 目錄變更時重新載入環境（CwdChanged + SessionStart)
- 自動核准特定權限提示（PermissionRequest, 回傳 JSON allow）

## Hook 如何運作

事件觸發時, 所有符合的 hooks 平行執行. Hook 透過 stdin 接收 JSON, 透過 stdout, stderr 與結束代碼告知 Claude Code 下一步.

- **結束代碼 0** : 沒有異議, 動作正常進行
- **結束代碼 2** : 動作被封鎖, stderr 內容回饋給 Claude
- **其他結束代碼** : 動作繼續進行, 但顯示 hook 錯誤提示

## 用 matcher 篩選

`matcher` 依事件類型篩選不同欄位, 例如 `PreToolUse`/`PostToolUse` 依工具名稱, `SessionStart` 依啟動方式（startup/resume/clear/compact）.

## Prompt 型與 Agent 型 hooks

`type: "prompt"` 送出提示詞給模型做是非判斷; `type: "agent"` 產生可讀檔案, 搜尋程式碼的 subagent 來驗證條件, 逾時較長（60 秒, 最多 50 輪工具使用）.

---

# 5. Hooks reference

> Reference for Claude Code hook events, configuration schema, JSON input/output formats, exit codes, async hooks, HTTP hooks, prompt hooks, and MCP tool hooks.

## 完整事件清單

| 事件 | 觸發時機 |
|---|---|
| `SessionStart` | Session 開始或恢復時 |
| `UserPromptSubmit` | 送出提示詞, Claude 處理前 |
| `PreToolUse` | 工具呼叫執行前, 可封鎖 |
| `PermissionRequest` | 權限對話框出現時 |
| `PostToolUse` | 工具呼叫成功後 |
| `Notification` | Claude Code 送出通知時 |
| `SubagentStart` / `SubagentStop` | 子代理產生 / 結束時 |
| `Stop` | Claude 完成回應時 |
| `PreCompact` / `PostCompact` | Context 壓縮前 / 後 |
| `SessionEnd` | Session 結束時 |

## Hook 類型

| 類型 | 說明 |
|---|---|
| `command` | 執行 shell 指令 |
| `http` | 將事件資料 POST 到 URL |
| `mcp_tool` | 呼叫已連接的 MCP 伺服器工具 |
| `prompt` | 單輪 LLM 評估 |
| `agent` | 多輪驗證, 附帶工具存取（實驗性） |

## 設定位置與範圍

| 位置 | 範圍 | 可分享 |
|---|---|---|
| `~/.claude/settings.json` | 你所有的專案 | 否, 僅限本機 |
| `.claude/settings.json` | 單一專案 | 是, 可提交到 repo |
| `.claude/settings.local.json` | 單一專案 | 否, 自動加入 gitignore |
| Managed policy settings | 組織全體 | 是, 管理員控制 |

## 常見限制

`PostToolUse` 無法撤銷已執行的動作; `PermissionRequest` 在非互動模式（`-p`）不會觸發; `Stop` hook 連續封鎖 8 次會被覆寫, 恢復正常結束.

---

# 6. Connect to MCP servers

> Add an MCP server to Claude Code, verify the connection, and find the configuration on disk.

## 新增並驗證伺服器（以文件伺服器為例）

```bash
claude mcp add --transport http claude-code-docs https://code.claude.com/docs/mcp
claude mcp list
```

狀態指標 : `✓ Connected`（就緒), `! Needs authentication`（需要瀏覽器登入), `✗ Failed to connect`（連線失敗）.

## 設定存放位置

| Scope | 檔案 | 適用對象 |
|---|---|---|
| `local`（預設） | `~/.claude.json`, 該專案項目下 | 只有你, 只有此專案 |
| `project` | 專案根目錄的 `.mcp.json` | 所有複製此專案的人 |
| `user` | `~/.claude.json`, 頂層 mcpServers 鍵 | 只有你, 所有專案 |

## 新增本機（stdio）伺服器

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

## 需要登入的伺服器

新增後狀態顯示 `! Needs authentication`, 在 session 內執行 `/mcp` 選擇伺服器並完成瀏覽器登入.

## 其他介面連接方式

Desktop app 的 Connectors UI, VS Code 的 `claude mcp add`, Claude Code on the web 讀取儲存庫的 `.mcp.json`.

---

# 7. Connect Claude Code to tools via MCP

> Learn how to connect Claude Code to your tools with the Model Context Protocol.

## 你可以用 MCP 做的事

實作議題追蹤系統中描述的功能, 分析監控資料, 查詢資料庫, 整合設計稿, 自動化工作流程, 或讓 MCP 伺服器當作 channel 推送訊息進 session.

## 安裝方式

| 方式 | 指令範例 |
|---|---|
| 遠端 HTTP（建議） | `claude mcp add --transport http notion https://mcp.notion.com/mcp` |
| 遠端 SSE（已棄用） | `claude mcp add --transport sse asana https://mcp.asana.com/sse` |
| 本機 stdio | `claude mcp add --env KEY=val --transport stdio airtable -- npx -y airtable-mcp-server` |
| 遠端 WebSocket | `claude mcp add-json events-server '{"type":"ws","url":"wss://..."}'` |

## 安裝範圍

| Scope | 載入於 | 與團隊共享 | 儲存於 |
|---|---|---|---|
| Local | 僅目前專案 | 否 | `~/.claude.json` |
| Project | 僅目前專案 | 是, 透過版本控制 | 專案根目錄的 `.mcp.json` |
| User | 所有你的專案 | 否 | `~/.claude.json` |

## 範圍優先順序

Local > Project > User > Plugin 提供的伺服器 > claude.ai connectors. 同名時取最高優先權來源, 欄位不會跨範圍合併.

## MCP tool search

預設啟用, 讓 context 使用量維持低點. 只有工具名稱與伺服器指示在 session 開始時載入, 完整 schema 依需求延後載入.

## OAuth 驗證

執行 `/mcp` 或 `claude mcp login <name>` 完成瀏覽器登入流程. 也支援固定 callback port 或預先設定的 OAuth 憑證.

---

# 8. Output styles

> Adapt Claude Code for uses beyond software engineering

Output styles 改變 Claude 如何回應, 而非它知道什麼. 它們修改系統提示以設定角色, 語氣與輸出格式.

## 內建 output styles

| Style | 說明 |
|---|---|
| Default | 既有系統提示, 為高效完成軟體工程任務而設計 |
| Proactive | Claude 立即執行, 做出合理假設而非暫停等待常規決策 |
| Explanatory | 在協助完成任務之間提供教育性的 "Insights" |
| Learning | 協作式邊做邊學模式, 會用 TODO(human) 標記讓你自己實作部分程式碼 |

## 切換 output style

執行 `/config` 選擇 Output style, 或直接編輯設定檔中的 `outputStyle` 欄位. 變更於 `/clear` 或新 session 後生效.

## 建立自訂 output style

在 `~/.claude/output-styles` 或 `.claude/output-styles` 建立 Markdown 檔, frontmatter 決定是否保留 Claude Code 內建的軟體工程指示（`keep-coding-instructions: true`）.

---

# 9. Create plugins

> Create custom plugins to extend Claude Code with skills, agents, hooks, and MCP servers.

## Standalone 設定 vs Plugins

| 方式 | Skill 名稱 | 最適合 |
|---|---|---|
| Standalone（`.claude/` 目錄） | `/hello` | 個人工作流程, 單一專案客製化, 快速實驗 |
| Plugins | `/plugin-name:hello` | 與團隊分享, 分發給社群, 版本化發布, 跨專案重複使用 |

## 快速入門

1. 建立 plugin 目錄與 `.claude-plugin/plugin.json` manifest
2. 在 `skills/hello/SKILL.md` 加入 skill
3. 用 `claude --plugin-dir ./my-first-plugin` 測試, 呼叫 `/my-first-plugin:hello`

## Plugin 目錄結構

| 目錄 | 用途 |
|---|---|
| `.claude-plugin/` | 只放 `plugin.json` manifest |
| `skills/` | Skills 目錄結構 |
| `agents/` | 自訂 subagent 定義 |
| `hooks/` | 事件處理器（`hooks.json`） |
| `.mcp.json` | MCP 伺服器設定 |
| `.lsp.json` | LSP 伺服器設定（程式碼智慧） |
| `monitors/` | 背景監控設定 |

## 測試與分享

用 `--plugin-dir` 本機測試, 修改後執行 `/reload-plugins` 套用變更. 準備分享時建立 marketplace, 或提交到官方 / 社群 marketplace 審核.

---

# 10. Plugins reference

> Complete technical reference for Claude Code plugin system, including schemas, CLI commands, and component specifications.

## 元件參考摘要

| 元件 | 位置 | 格式 |
|---|---|---|
| Skills | `skills/` 或 `commands/` | 目錄含 SKILL.md, 或單一根目錄 SKILL.md |
| Agents | `agents/` | Markdown 檔含 frontmatter |
| Hooks | `hooks/hooks.json` | JSON 事件對應設定 |
| MCP servers | `.mcp.json` | 標準 MCP 伺服器設定 |
| LSP servers | `.lsp.json` | 語言伺服器設定 |
| Monitors | `monitors/monitors.json` | 背景監控指令陣列 |
| Themes | `themes/` | 色彩主題 JSON |

## 環境變數

`${CLAUDE_PLUGIN_ROOT}`（plugin 安裝目錄的絕對路徑), `${CLAUDE_PLUGIN_DATA}`（持久資料目錄, 跨更新保留), `${CLAUDE_PROJECT_DIR}`（專案根目錄）.

## 常用 CLI 指令

| 指令 | 用途 |
|---|---|
| `claude plugin init <name>` | 在 `~/.claude/skills/` 建立新 plugin 骨架 |
| `claude plugin install <plugin>` | 從 marketplace 安裝 plugin |
| `claude plugin uninstall <plugin>` | 移除已安裝的 plugin |
| `claude plugin enable/disable <plugin>` | 啟用 / 停用 plugin |
| `claude plugin list` | 列出已安裝的 plugins |
| `claude plugin details <name>` | 顯示元件清單與預估 token 成本 |
| `claude plugin validate` | 驗證 manifest 與元件檔案 |

## 安裝範圍

| Scope | 設定檔 | 用途 |
|---|---|---|
| `user`（預設） | `~/.claude/settings.json` | 個人 plugin, 跨所有專案 |
| `project` | `.claude/settings.json` | 團隊 plugin, 透過版本控制共享 |
| `local` | `.claude/settings.local.json` | 專案專屬, 已加入 gitignore |
| `managed` | Managed settings | 管理員部署的 plugin（唯讀） |

---

# 11. Create and distribute a plugin marketplace

> Build and host plugin marketplaces to distribute Claude Code extensions across teams and communities.

Marketplace 是讓你分發 plugin 給他人的目錄清單, 提供集中發現, 版本追蹤與自動更新.

## 建立步驟

1. 建立 plugins（skills, agents, hooks, MCP 伺服器）
2. 建立 `.claude-plugin/marketplace.json` 目錄檔
3. 推送到 GitHub 或其他 git 主機
4. 使用者用 `/plugin marketplace add` 新增並安裝個別 plugin

## marketplace.json 必要欄位

`name`（識別字), `owner`（維護者資訊), `plugins`（plugin 清單, 每個至少要有 `name` 與 `source`）.

## Plugin 來源類型

| 來源 | 說明 |
|---|---|
| 相對路徑 | 同一儲存庫內的本機目錄, 如 `"./my-plugin"` |
| `github` | GitHub 儲存庫, 可指定 ref 或 sha |
| `url` | 任意 git URL |
| `git-subdir` | Git 儲存庫內的子目錄, 適合大型 monorepo |
| `npm` | 透過 npm install 安裝的套件 |

## 版本管理

Claude Code 依序解析版本 : `plugin.json` 中的 `version` → marketplace 條目中的 `version` → git commit SHA. 設定明確版本後, 每次發布都要手動遞增才能推送更新給使用者.

## 受管理的 marketplace 限制

管理員可用 `strictKnownMarketplaces` 限制使用者能新增的 marketplace 來源, 空陣列表示完全鎖定.

---

# 12. Discover and install prebuilt plugins through marketplaces

> Find and install plugins from marketplaces to extend Claude Code with new skills, agents, and capabilities.

使用 marketplace 分兩步驟 : 先新增目錄（不會安裝任何東西), 再瀏覽並安裝個別 plugin.

## 官方 Anthropic marketplace

`claude-plugins-official` 於啟動 Claude Code 時自動可用. 執行 `/plugin` 前往 Discover 分頁瀏覽.

### 主要類別

| 類別 | 範例 plugin |
|---|---|
| 程式碼智慧（LSP） | `pyright-lsp`, `typescript-lsp`, `rust-analyzer-lsp` 等, 需要對應語言伺服器執行檔 |
| 外部整合 | `github`, `linear`, `notion`, `figma`, `slack`, `sentry` 等預先設定的 MCP 伺服器 |
| 自動安全審查 | `security-guidance`, 審查每次變更的常見漏洞 |
| 開發工作流程 | `commit-commands`, `pr-review-toolkit`, `agent-sdk-dev`, `plugin-dev` |
| Output styles | `explanatory-output-style`, `learning-output-style` |

## 社群 marketplace

`anthropics/claude-plugins-community` 收錄通過自動驗證與安全篩選的第三方 plugin, 需手動新增 `/plugin marketplace add anthropics/claude-plugins-community`.

## 安裝 plugin

```text
/plugin install plugin-name@marketplace-name
```

選擇安裝範圍 : User（跨所有專案), Project（與協作者共享), Local（僅此儲存庫）.

## 安全性

Plugin 與 marketplace 是高度信任的元件, 可用你的使用者權限執行任意程式碼. 只從你信任的來源安裝.

---

# 13. Constrain plugin dependency versions

> Declare version constraints on plugin dependencies so your plugin keeps working when an upstream plugin ships a breaking change.

Plugin 可以透過 `plugin.json` 中的 `dependencies` 陣列依賴其他 plugin. 預設情況下, 依賴會追蹤最新版本, 版本限制讓你能將依賴固定在測試過的版本範圍.

## 宣告版本限制

```json
{
  "name": "deploy-kit",
  "dependencies": [
    "audit-logger",
    { "name": "secrets-vault", "version": "~2.1.0" }
  ]
}
```

`version` 欄位接受 semver 範圍語法, 如 `~2.1.0`, `^2.0`, `>=1.4`.

## 標記發布版本

版本限制依據 marketplace 儲存庫上的 git tag 解析, 命名慣例為 `{plugin-name}--v{version}`, 執行 `claude plugin tag --push` 建立.

## 常見錯誤

| 錯誤 | 意義 |
|---|---|
| `dependency-unsatisfied` | 宣告的依賴未安裝或已停用 |
| `range-conflict` | 多個版本需求無法合併 |
| `dependency-version-unsatisfied` | 已安裝的依賴版本超出此 plugin 宣告的範圍 |
| `no-matching-tag` | 依賴的儲存庫沒有符合範圍的 tag |

## 清理孤兒依賴

```bash
claude plugin prune
```
移除不再被任何已安裝 plugin 需要的自動安裝依賴.

---

# 14. Recommend your plugin from your CLI

> Emit a one-line marker from your CLI so Claude Code prompts users to install your official plugin.

若你維護的 CLI 或 SDK 在官方 Anthropic marketplace 有對應的 plugin, 你的工具可以提示 Claude Code 使用者安裝該 plugin.

## 運作方式

當 Claude Code 透過 Bash 或 PowerShell 工具執行指令時, 會設定 `CLAUDECODE=1` 環境變數. 你的 CLI 偵測到此變數後, 寫入一行標記到 stderr :

```text
<claude-code-hint v="1" type="plugin" value="example-cli@claude-plugins-official" />
```

Claude Code 會掃描並移除此標記行, 再顯示安裝提示給使用者. Claude Code 永遠不會自動安裝, 使用者必須確認.

## 選擇放置點

| 位置 | 為何有效 |
|---|---|
| `--help` 輸出 | Claude 探索陌生 CLI 時常執行 help |
| 未知子指令錯誤 | 正好是 Claude 對介面感到困惑的時刻 |
| 登入或驗證成功 | 使用者已處於設定情境 |
| 首次執行的歡迎訊息 | 自然的入門時刻 |

## 提示頻率限制

每個 plugin 只提示一次（無論使用者如何回答), 每個 session 跨所有 CLI 最多顯示一次提示.

---

# 15. Recommend plugins for your org

> Add a relevance block to marketplace plugin entries so Claude Code suggests them when a user's work matches.

若你營運組織的 plugin marketplace, 可以根據使用者正在進行的工作, 建議特定的 plugin.

## 加入 relevance 區塊

```json
{
  "name": "terraform-helpers",
  "relevance": {
    "topic": "Terraform",
    "signals": {
      "cli": ["terraform"],
      "filesRead": ["**/*.tf"]
    }
  }
}
```

## 訊號欄位參考

| 欄位 | 說明 |
|---|---|
| `cwd` | 符合工作目錄的 glob 樣式, 唯一能在 session 開始時就符合的訊號 |
| `cli` | 本 session 執行過的 shell 指令名稱 |
| `hosts` | Bash 指令中出現過的網域主機名稱 |
| `filesRead` | 符合 Claude 讀取過的檔案路徑的 glob 樣式 |
| `manifestDeps` | 套件清單檔案（如 package.json）中宣告的相依套件 |

## 啟用建議

Marketplace 宣告的建議是選擇性加入的（opt-in), 管理員必須在 managed settings 的 `pluginSuggestionMarketplaces` 中將 marketplace 加入允許清單.

## 使用者會看到什麼

符合訊號時, 顯示為 spinner 下方的提示文字, session 開始時的一行通知, 或 `/plugin` Discover 分頁置頂並標註符合原因.
