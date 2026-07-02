# 第九階段, Agent SDK, 開發者建置自訂代理

## 1. Agent SDK 總覽

Agent SDK 讓你以 Claude Code 作為函式庫建置正式環境的 AI 代理, 提供與 Claude Code 相同的工具、代理迴圈、情境管理能力, 可用 Python 與 TypeScript 程式化操作.內建工具包含 Read、Write、Edit、Bash、Monitor、Glob、Grep、WebSearch、WebFetch、AskUserQuestion, 讓代理不需要你自己實作工具執行邏輯即可開始工作.

安裝, TypeScript 用 `npm install @anthropic-ai/claude-agent-sdk`, Python 用 `pip install claude-agent-sdk`（需要 Python 3.10 以上）.設定 `ANTHROPIC_API_KEY` 環境變數即可驗證, 也支援 Amazon Bedrock、Claude Platform on AWS、Google Vertex AI、Microsoft Azure 等第三方供應商.

核心能力涵蓋內建工具、hooks（在代理生命週期的關鍵點執行自訂程式碼）、subagents（產生專門的代理處理聚焦的子任務）、MCP（連接外部系統如資料庫、瀏覽器、API）、permissions（控制代理能使用哪些工具）、sessions（維持跨多輪交流的情境, 可回復或分支）.

Agent SDK 與 Client SDK 的差異在於, Client SDK 給你直接的 API 存取, 你自己實作工具執行迴圈, Agent SDK 則讓 Claude 自主處理工具.與 Claude Code CLI 相比, 兩者能力相同但介面不同, CLI 適合互動式開發與一次性任務, SDK 適合 CI/CD 與正式環境自動化.與 Managed Agents（託管的 REST API）相比, Agent SDK 在你自己的行程中執行代理迴圈, Managed Agents 則由 Anthropic 代管沙盒與基礎架構.

## 2. Quickstart 快速入門

這份快速入門引導建置一個能找出並修復程式錯誤的代理.前置需求為 Node.js 18 以上或 Python 3.10 以上, 以及一個 Anthropic 帳號.

建立專案目錄、安裝 SDK 套件、設定 API 金鑰後, 建立一個含有兩個錯誤的 `utils.py`（除以零與呼叫 None 物件屬性）.接著撰寫代理腳本,

```python
async for message in query(
    prompt="Review utils.py for bugs that would cause crashes. Fix any issues you find.",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Glob"],
        permission_mode="acceptEdits",
    ),
):
    ...
```

`query` 是建立代理迴圈的主要進入點, 回傳非同步疊代器.`prompt` 是你要 Claude 做的事.`options` 用來設定, 例如 `allowedTools` 預先核准特定工具, `permissionMode: "acceptEdits"` 自動核准檔案變更.

關鍵概念, 工具組合決定代理能做什麼（唯讀分析、分析並修改、完整自動化）.權限模式決定需要多少人工監督, `acceptEdits` 自動核准編輯與常見檔案系統指令, `plan` 只執行唯讀工具, `dontAsk` 拒絕不在允許清單中的任何操作, `auto`（僅 TypeScript）用模型分類器核准或拒絕每個工具呼叫, `bypassPermissions` 完全不詢問, `default` 需要 `canUseTool` 回呼處理核准.

## 3. Agent loop 代理迴圈如何運作

Agent SDK 讓你在自己的應用程式中嵌入 Claude Code 的自主代理迴圈.啟動代理後, SDK 執行與 Claude Code 相同的執行迴圈, Claude 評估提示詞、呼叫工具採取行動、接收結果、重複直到任務完成.

迴圈概覽, 接收提示詞（SDK 產出 `SystemMessage`, subtype 為 init）, 評估並回應（Claude 可能回應文字、要求工具呼叫、或兩者皆有）, 執行工具（SDK 執行每個要求的工具並收集結果）, 重複（步驟持續直到 Claude 產出沒有工具呼叫的回應）, 回傳結果（SDK 產出最終的 `AssistantMessage` 加上含有最終文字、token 用量、成本、session ID 的 `ResultMessage`）.

一個回合（turn）是迴圈內一次來回, 可用 `max_turns` 限制工具使用回合數, `max_budget_usd` 依支出上限停止.effort 選項控制 Claude 投入多少推理, low 用於檔案查詢, medium 用於例行編輯, high 用於重構除錯, xhigh 與 max 用於需要深度分析的多步驟問題.

情境視窗是 Claude 在 session 期間可用的全部資訊總量, 不會在回合之間重置.系統提示詞、CLAUDE.md、工具定義、對話歷史都會累積.保持不變的內容（系統提示詞、工具定義、CLAUDE.md）會自動套用 prompt caching.當情境視窗接近上限時, SDK 會自動壓縮對話, 摘要較舊的歷史以釋放空間.

## 4. 在 SDK 中使用 Claude Code 功能

Agent SDK 建構於與 Claude Code 相同的基礎上, 因此 SDK 代理可存取相同的檔案系統型功能, 專案指示（CLAUDE.md 與規則）、skills、hooks 等.省略 `settingSources` 時, `query()` 讀取與 CLI 相同的檔案系統設定, 傳入 `settingSources: []` 則限制代理只用你程式化設定的內容.

`settingSources` 有三種來源, `"project"` 載入專案 CLAUDE.md、`.claude/rules/*.md`、專案 skills、專案 hooks、專案 settings.json, `"user"` 載入使用者層級 CLAUDE.md 與 skills（`~/.claude/`）, `"local"` 載入 CLAUDE.local.md 與本機 settings.json.省略 `settingSources` 等同於 `["user", "project", "local"]`.

有幾項輸入不受 `settingSources` 控制, 管理政策設定、`~/.claude.json` 全域設定、auto memory、claude.ai MCP connectors 都會照常讀取, 除非個別設定停用.對於多租戶隔離, 務必設定 `settingSources: []` 加上 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`.

## 5. Streaming Input 串流輸入模式

Agent SDK 支援兩種輸入模式, 串流輸入模式（預設且建議）是持久互動式 session, 單一訊息輸入則是使用 session 狀態與回復的一次性查詢.

串流輸入模式的優點包含, 圖片上傳（可直接附加圖片到訊息做視覺分析）, 佇列訊息（可送出多個依序處理的訊息並能中斷）, 完整工具整合、即時回饋、自然的情境持續性.適合建置豐富的互動體驗.

單一訊息輸入較簡單但較受限, 適合一次性回應、不需要圖片附件或執行期中控制方法、需要在無狀態環境（如 lambda 函式）中運作的情境.不支援直接圖片附件、動態訊息佇列、即時中斷、自然的多輪對話.

## 6. Stream responses 即時串流回應

預設情況下, Agent SDK 在 Claude 完成每個回應後才產出完整的 `AssistantMessage`.要接收文字與工具呼叫逐步產生時的增量更新, 需在選項中設定 `include_partial_messages`（Python）或 `includePartialMessages`（TypeScript）為 true, 這會讓 SDK 產出包含原始 API 事件的 `StreamEvent` 訊息.

常見的事件類型包含 `message_start`（新訊息開始）、`content_block_start`（新內容區塊開始, 文字或工具使用）、`content_block_delta`（內容的增量更新）、`content_block_stop`（內容區塊結束）、`message_delta`、`message_stop`.要顯示文字生成過程, 尋找 `delta.type` 為 `text_delta` 的 `content_block_delta` 事件.工具呼叫也會逐步串流, 可追蹤工具何時開始、接收其輸入、以及何時完成.

## 7. Work with sessions 使用 session

session 是 SDK 在代理工作時累積的對話歷史, 包含你的提示詞、每次工具呼叫、每個工具結果、每個回應, SDK 會自動寫入磁碟供之後回到該 session.

繼續（continue）與回復（resume）都會接續既有的 session 並加入新內容, 差別在於如何找到該 session, continue 找到目前目錄中最近的 session, resume 需要具體的 session ID.分支（fork）則不同, 它建立一個以原始歷史複本開始的新 session, 原始 session 保持不變, 適合在保留回頭選項的同時嘗試不同方向.

Python 的 `ClaudeSDKClient` 會在呼叫之間內部處理 session ID, TypeScript 則傳入 `continue: true` 讓 SDK 自動挑選目前目錄中最近的 session.要回復特定 session, 從結果訊息的 `session_id` 欄位擷取 ID 後傳給 `resume`.要跨主機回復 session, 需要移動 session 檔案或搭配 `SessionStore` 轉接器使用共用儲存.

## 8. Persist sessions to external storage 持久化 session 至外部儲存

預設情況下, SDK 把 session 逐字稿寫入本機檔案系統的 `~/.claude/projects/` 下的 JSONL 檔案.`SessionStore` 轉接器讓你把這些逐字稿鏡射到自己的後端, 例如 S3、Redis、或資料庫, 讓在一台主機建立的 session 能在另一台主機回復.

常見使用情境包含多主機部署（無伺服器函式、自動擴展工作者不共用檔案系統）、持久性（本機容器是短暫的）、合規與稽核（把逐字稿存放在你已治理的儲存中）.

`SessionStore` 是一個物件, 有兩個必要方法 `append` 與 `load`, 三個選用方法 `listSessions`、`delete`、`listSubkeys`.SDK 提供 `InMemorySessionStore` 供開發與測試使用, 也在 TypeScript SDK 儲存庫提供 S3、Redis、Postgres 的參考轉接器實作.儲存是鏡射而非取代, Claude Code 子行程一律先寫入本機磁碟, 鏡射寫入是盡力而為的, 若失敗會重試至多三次並在耗盡後放棄, 本機逐字稿依然完整不受影響.

## 9. Give Claude custom tools 自訂工具

自訂工具讓你擴充 Agent SDK, 定義自己的函式供 Claude 呼叫.透過 SDK 的行程內 MCP server, 可讓 Claude 存取資料庫、外部 API、領域專屬邏輯、或應用程式需要的任何能力.

一個工具由四個部分定義, 名稱（Claude 用來呼叫工具的唯一識別碼）、描述（工具做什麼, Claude 讀取以決定何時呼叫）、輸入結構描述（Claude 必須提供的引數, TypeScript 用 Zod schema, Python 用型別字典或完整 JSON Schema）、處理常式（Claude 呼叫工具時執行的非同步函式, 必須回傳含 content 陣列的物件）.

定義工具後, 用 `createSdkMcpServer`（TypeScript）或 `create_sdk_mcp_server`（Python）包裝成伺服器, 在你的應用程式行程內執行, 而非獨立行程.MCP 工具名稱遵循 `mcp__{server_name}__{tool_name}` 格式, 需列在 `allowedTools` 中才能不經提示執行.

工具結果可回傳文字、圖片、音訊、resource、resource_link 區塊.處理常式若擲出未捕捉的例外會停止代理迴圈, 若捕捉錯誤並回傳 `isError: true` 則代理迴圈會繼續, Claude 會把錯誤視為資料並可能重試或改用其他工具.

## 10. Connect to external tools with MCP 連接 MCP servers

Model Context Protocol（MCP）是連接 AI 代理與外部工具、資料來源的開放標準.透過 MCP, 代理可查詢資料庫、整合 Slack 與 GitHub 等 API、連接其他服務, 不需要撰寫自訂工具實作.

MCP servers 可以是本機行程（stdio 傳輸）、透過 HTTP 連線、或直接在 SDK 應用程式中執行（SDK MCP servers）.MCP 工具需要明確權限才能被 Claude 使用, 用 `allowedTools` 加上萬用字元 `mcp__servername__*` 可核准伺服器提供的所有工具.

當設定大量 MCP 工具時, 工具定義可能消耗大量情境視窗空間.MCP tool search 預設啟用, 保留工具定義不進情境, 只在 Claude 需要時載入相關的 3 到 5 個工具.傳輸類型方面, 若文件給的是要執行的指令則用 stdio, 若給的是 URL 則用 HTTP 或 SSE, 若自行在程式碼中建置工具則用 SDK MCP server.

## 11. Scale to many tools with tool search 用工具搜尋擴展到大量工具

tool search 讓代理能處理數百或數千個工具, 依需求動態發現與載入, 而非把所有工具定義預先載入情境視窗.這解決兩個挑戰, 情境效率（50 個工具可能用掉 1 到 2 萬個 token）與工具選擇準確度（超過 30 到 50 個工具時準確度會下降）.

tool search 預設啟用.啟用時, 工具定義會保留在情境視窗之外, 代理收到可用工具的摘要, 在任務需要目前未載入的能力時進行搜尋, 找到最相關的 3 到 5 個工具載入情境, 之後的回合持續可用.可透過 `ENABLE_TOOL_SEARCH` 環境變數調整, 設為 `auto` 時依工具定義佔情境視窗的比例決定是否啟用, `auto:N` 可自訂百分比閾值.工具少於約 10 個時, 預先載入全部通常更快.支援除 Haiku 以外的所有 Claude 模型.

## 12. Agent Skills in the SDK SDK 中的 Agent Skills

Agent Skills 讓 Claude 具備 Claude 會自主呼叫的專門能力, 打包為含有指示、描述、選用支援資源的 `SKILL.md` 檔案.與 subagents（可程式化定義）不同, skills 必須建立為檔案系統產物, SDK 沒有提供程式化的 skill 註冊 API.

skills 透過 `settingSources` 從檔案系統發現, 在 `query()` 上設定 `skills` 選項控制哪些 skill 對這個 session 可用, 省略時已發現的 skills 會啟用且 Skill 工具可用, 傳入 `"all"` 啟用所有已發現的 skill, 傳入名稱清單只啟用特定的, 傳入 `[]` 全部停用.

skill 位置分為專案 skills（`.claude/skills/`, 透過 git 與團隊共享）、使用者 skills（`~/.claude/skills/`, 跨專案的個人 skill）、plugin skills（隨已安裝的 Claude Code plugin 打包）.SKILL.md 中的 `allowed-tools` 前置資料只在直接使用 Claude Code CLI 時有效, 透過 SDK 使用 skills 時不適用, 應改用主要的 `allowedTools` 選項控制工具存取.

## 13. Subagents in the SDK SDK 中的子代理

subagents 是主代理可以產生的獨立代理實例, 用來處理聚焦的子任務.可以用三種方式建立 subagents, 程式化（在 `query()` 選項的 `agents` 參數）、以檔案系統為基礎（`.claude/agents/` 目錄下的 markdown 檔案）、內建的通用型 subagent（Claude 隨時可透過 Agent 工具呼叫, 不需自行定義）.

subagents 的優點包含, 情境隔離（每個 subagent 在自己全新的對話中執行, 只有最終訊息回傳給父代理）, 平行化（多個 subagent 可同時執行, 讓獨立子任務在最慢那個的時間內完成而非加總）, 專門的指示與知識（每個 subagent 可有量身打造的系統提示詞）, 工具限制（subagent 可限制在特定工具, 降低意外操作的風險）.

`AgentDefinition` 設定包含 `description`（必要, 說明何時使用此代理）、`prompt`（必要, 代理的系統提示詞）、`tools`（選用, 允許的工具清單）、`disallowedTools`、`model`、`skills`、`memory`、`mcpServers`、`maxTurns`、`background`、`effort`、`permissionMode`.subagent 的情境視窗全新開始, 但不是空的, 唯一從父層到 subagent 的管道是 Agent 工具的提示詞字串.

## 14. Slash Commands in the SDK SDK 中的斜線指令

斜線指令提供以 `/` 開頭的特殊指令控制 Claude Code session.這些指令可透過 SDK 送出執行壓縮情境、列出情境用量、或呼叫自訂指令等動作.只有不需要互動式終端機即可運作的指令才能透過 SDK 派送, `system/init` 訊息會列出你 session 中可用的指令.

透過在提示詞字串中包含斜線指令即可送出, 就像一般文字一樣.常見的內建指令包含 `/compact`（壓縮對話歷史, 摘要較舊訊息同時保留重要情境）與 `/clear`（重置對話為空情境）.

除了內建指令, 也可以建立自己的自訂斜線指令, 定義為特定目錄中的 markdown 檔案, 與 subagents 的設定方式類似.建議格式是 `.claude/skills/<name>/SKILL.md`, 支援相同的斜線指令呼叫（`/name`）加上 Claude 自主呼叫, `.claude/commands/` 目錄則是舊格式.自訂指令支援用 `$0`、`$1` 等佔位符處理動態引數, 也可用 `!` 前綴執行 bash 指令並納入輸出, 用 `@` 前綴引用檔案內容.

## 15. Plugins in the SDK SDK 中的外掛

plugins 讓你用可跨專案共享的自訂功能擴充 Claude Code.透過 Agent SDK, 可以程式化從本機目錄載入 plugins, 為代理 session 加入 skills、agents、hooks、MCP servers.

透過在選項設定中提供本機檔案系統路徑來載入 plugins, `type` 欄位必須是 `"local"`, 這是 SDK 唯一接受的值.要使用透過 marketplace 或遠端儲存庫散布的 plugin, 需先下載再提供本機目錄路徑.路徑應指向 plugin 的根目錄, 也就是 `skills/`、`agents/`、`hooks/`、`commands/`（舊格式）、或 `.claude-plugin/` 的父目錄.

plugin 成功載入後會出現在系統初始化訊息中, 可檢查 `message.plugins`、`message.skills`、`message.slash_commands` 驗證.plugin 提供的 skills 會自動以 plugin 名稱加上命名空間避免衝突, 直接呼叫時送出 `/plugin-name:skill-name`.

## 16. Modifying system prompts 修改系統提示詞

系統提示詞定義 Claude 的行為、能力、回應風格.Agent SDK 有三個起始點, 最小預設值（未設定 systemPrompt 時, SDK 使用涵蓋工具呼叫但省略 Claude Code 編碼指引的最小提示詞）、`claude_code` 預設值（Claude Code CLI 使用的完整系統提示詞）、自訂字串（你自己撰寫的提示詞, SDK 只送出你提供的內容）.

決定起始點的關鍵因素是你的代理與 Claude Code 有多相似, 一個在儲存庫中運作、有人類觀看串流輸出並引導工作的編碼代理.建置類似 Claude Code 的 CLI 或 IDE 工具用 `claude_code` 預設值, 相同工具加上產品專屬規則用預設值加 `append`, 不同介面 身分 或權限模型的代理用自訂提示詞字串, 沒有代理人格的薄工具呼叫迴圈則省略 systemPrompt 選項.

CLAUDE.md 檔案給 Claude 持久的專案情境與指示, SDK 把其內容注入對話而非系統提示詞, 因此可搭配任何系統提示詞設定運作.output styles 是儲存的設定, 修改 Claude 的系統提示詞, 以 markdown 檔案儲存, 可跨 session 與專案重複使用.

## 17. Configure permissions 設定權限

Agent SDK 提供權限控制管理 Claude 如何使用工具.Claude 要求工具時, SDK 依此順序檢查權限, hooks（可直接拒絕呼叫或放行）、拒絕規則（來自 disallowed_tools 與 settings.json, 即使在 bypassPermissions 模式下也會封鎖）、詢問規則（符合時會落到 canUseTool 回呼確認）、權限模式（bypassPermissions 核准到達此步驟的一切）、允許規則、canUseTool 回呼（若前述都未解決, 呼叫此回呼做決定）.

`allowed_tools` 只影響核准, 未列出的工具仍然可用並落到權限模式.`disallowed_tools` 依是否指名工具或限定模式內的樣式而有不同行為, 裸名稱如 `Bash` 會把工具定義整個從請求中移除, 限定的規則如 `Bash(rm *)` 則保留工具可用但拒絕符合的呼叫.

SDK 支援的權限模式包含 `default`（無自動核准）、`dontAsk`（拒絕而非提示）、`acceptEdits`（自動接受檔案編輯）、`bypassPermissions`（略過權限檢查, 使用時要謹慎）、`plan`（規劃模式, 探索並規劃但不編輯原始檔案）、`auto`（僅 TypeScript, 模型分類核准）.

## 18. Intercept and control agent behavior with hooks SDK 中的 hooks

hooks 是在代理事件發生時執行你程式碼的回呼函式, 例如工具被呼叫、session 開始、或執行停止.透過 hooks 可以在危險操作執行前封鎖它們、記錄並稽核每次工具呼叫、轉換輸入輸出、要求人工核准敏感動作、追蹤 session 生命週期.

可用的 hook 事件包含 `PreToolUse`（工具呼叫請求, 可封鎖或修改）、`PostToolUse`（工具執行結果）、`UserPromptSubmit`（使用者提示詞送出）、`Stop`（代理執行停止）、`SubagentStart`/`SubagentStop`、`PreCompact`（對話壓縮請求）、`PermissionRequest`, 部分事件僅 TypeScript SDK 支援（如 `SessionStart`、`SessionEnd`、`Notification`）.

每個 hook 回呼收到輸入資料、工具使用 ID、情境三個引數, 回傳一個物件告訴代理如何處理, 允許操作、封鎖操作、修改輸入、或注入情境給對話.`hookSpecificOutput` 中的 `permissionDecision` 可設為 `allow`、`deny`、`ask`、`defer`.hooks 在你的應用程式行程中執行, 不在代理的情境視窗內, 因此不消耗情境.

## 19. Rewind file changes with checkpointing SDK 中的檔案回退

檔案 checkpointing 追蹤透過 Write、Edit、NotebookEdit 工具在代理 session 期間所做的檔案修改, 讓你能把檔案回退到任何先前的狀態.只有透過這三個工具做的變更會被追蹤, 透過 Bash 指令（如 `echo > file.txt`）做的變更不會被捕捉.

啟用 checkpointing 後, SDK 會在透過這些工具修改檔案前建立備份.回應串流中的使用者訊息會包含可作為回復點的 checkpoint UUID.檔案回退只會回復磁碟上的檔案到先前狀態, 不會回退對話本身, 對話歷史與情境在呼叫 rewindFiles() 後仍維持完整.

使用流程為, 在選項中啟用 `enable_file_checkpointing=True` 並設定 `extra_args={"replay-user-messages": None}` 以在回應串流中取得 checkpoint UUID, 從第一則使用者訊息擷取 checkpoint UUID, 稍後要回退時, 以空提示詞回復該 session 並呼叫 `rewind_files()`.限制包含, 只追蹤 Write/Edit/NotebookEdit 工具、checkpoint 綁定於建立它的 session、只回退檔案內容不含目錄的建立或刪除、只支援本機檔案.

## 20. Get structured output from agents 取得結構化輸出

結構化輸出讓你定義想要從代理取得的資料確切形狀.代理仍可使用任何需要的工具完成任務, 你依然能在最後取得符合結構描述的驗證 JSON.定義一個 JSON Schema, SDK 會依此驗證輸出, 不符合時重新提示.

透過 `outputFormat`（TypeScript）或 `output_format`（Python）選項傳入 schema, 代理完成後, 結果訊息會包含 `structured_output` 欄位, 內含符合結構描述的驗證資料.要取得完整型別安全, 可用 Zod（TypeScript）或 Pydantic（Python）定義結構描述並產生 JSON Schema.

當代理無法產出符合結構描述的有效 JSON 時可能失敗, 通常發生在結構描述對任務而言過於複雜、任務本身模糊不清、或代理達到重試上限.結果訊息的 `subtype` 為 `error_max_structured_output_retries` 時表示這種失敗, 應檢查此欄位以區分驗證失敗與模型回退撤回輸出的情況.

## 21. Todo Lists 待辦清單追蹤

待辦清單追蹤提供有結構的方式管理任務並向使用者顯示進度.SDK 內建的待辦功能協助組織複雜工作流程並讓使用者了解任務進展.自 TypeScript Agent SDK 0.3.142 與 Claude Code v2.1.142 起, session 改用結構化的 Task 工具 `TaskCreate`、`TaskUpdate`、`TaskGet`、`TaskList` 取代 `TodoWrite`.

待辦生命週期為, 建立為 pending（識別出任務時）、啟動為 in_progress（開始工作時）、完成（任務成功結束時）、移除（群組中所有任務都完成時）.SDK 會為需要 3 個以上明確動作的複雜多步驟任務、使用者提供的任務清單、非瑣碎操作、使用者明確要求時自動建立待辦.

Task 工具把單一 `TodoWrite` 呼叫拆成 `TaskCreate`（每個新項目）與 `TaskUpdate`（每次狀態變更）, 監控程式碼仍檢視助理串流中的 `tool_use` 區塊, 但改為依任務 ID 維護一個對照表而非在每次呼叫時取代整個清單.

## 22. Handle approvals and user input 處理核准與使用者輸入

Claude 在兩種情況下會要求使用者輸入, 需要工具使用權限（例如刪除檔案或執行指令前）、以及有澄清性問題時（透過 `AskUserQuestion` 工具）.兩者都會觸發你的 `canUseTool` 回呼, 暫停執行直到你回傳回應.

在選項中傳入 `canUseTool` 回呼, 收到工具名稱與輸入引數.回呼在兩種情況下觸發, 工具需要核准（Claude 想使用未被權限規則或模式自動核准的工具）, Claude 提出問題（Claude 呼叫 `AskUserQuestion` 工具）.重要的是, 自動核准的工具永遠不會觸發此回呼, 若需要套用到每次工具呼叫的邏輯, 應使用 `PreToolUse` hook.

你的回呼回傳允許（`PermissionResultAllow`, 傳入原始或修改過的輸入）或拒絕（`PermissionResultDeny`, 附上說明訊息）.除了允許或拒絕, 還可修改工具輸入、核准並記住（回傳建議的權限規則避免下次再詢問）、拒絕並建議替代方案、或完全重新導向（用串流輸入送出全新指示）.

## 23. Track cost and usage 追蹤成本與用量

Agent SDK 為每次與 Claude 的互動提供詳細的 token 用量資訊.`total_cost_usd` 與 `costUSD` 欄位是客戶端估算值, 並非權威的計費資料, 適合用於開發階段的洞察與大略預算, 正式計費應使用 Usage and Cost API 或 Claude Console 的 Usage 頁面.

理解成本追蹤需要掌握 SDK 如何劃分用量資料的範圍, `query()` 呼叫（一次呼叫可能涉及多個步驟）、步驟（一次請求回應循環）、session（用 resume 選項串連的一系列 query() 呼叫）.結果訊息包含 `total_cost_usd`, 是該次呼叫中所有步驟的累計估算成本.

當 Claude 平行呼叫多個工具時, 該回合中的所有訊息共用同一個 ID, 需依 ID 去除重複才能避免重複計算.結果訊息的 `modelUsage` 提供依模型分類的 token 計數與成本細目, 適合在使用多個模型時查看 token 流向.

## 24. Observability with OpenTelemetry SDK 的可觀測性

Agent SDK 可以把追蹤、指標、事件匯出為 OpenTelemetry 資料到任何接受 OTLP 的後端, 如 Honeycomb、Datadog、Grafana、Langfuse、或自架的收集器.SDK 執行 Claude Code CLI 作為子行程並透過本機管道溝通, CLI 內建 OpenTelemetry 監測工具, SDK 本身不產生遙測資料, 而是把設定傳給 CLI 行程.

CLI 匯出三種獨立的 OpenTelemetry 訊號, 各有自己的啟用開關與匯出器.指標（token、成本、session 等計數器）用 `OTEL_METRICS_EXPORTER` 啟用, 日誌事件（每個提示詞、API 請求、API 錯誤、工具結果的結構化紀錄）用 `OTEL_LOGS_EXPORTER` 啟用, 追蹤（每次互動、模型請求、工具呼叫、hook 的 span, beta 階段）需要 `OTEL_TRACES_EXPORTER` 加上 `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`.

追蹤功能讓你看到代理執行過程最詳細的檢視, `claude_code.interaction` 包裹代理迴圈的單一回合, `claude_code.llm_request` 包裹每次呼叫 Claude API, `claude_code.tool` 包裹每次工具呼叫.SDK 會自動把 W3C trace context 傳播進 CLI 子行程, 讓代理執行過程出現在你應用程式的追蹤中而非獨立的根節點.

## 25. Securely deploying AI agents 安全部署 AI 代理

Claude Code 與 Agent SDK 能執行程式碼、存取檔案、代表你與外部服務互動.這種彈性正是它們有用之處, 但也代表其行為可能受到處理內容（檔案、網頁、使用者輸入）的影響, 有時稱為提示注入.

安全部署代理不需要特殊的基礎架構, 適用於執行任何半信任程式碼的相同原則同樣適用, 隔離、最小權限、縱深防禦.Claude Code 內建的安全功能包含權限系統、bash 指令的 AST 剖析比對權限規則、網頁搜尋摘要化、沙盒模式.

對於需要更多強化的部署, 安全性邊界原則建議把敏感資源（如憑證）放在包含代理的邊界之外, 例如透過代理外部執行的 proxy 注入 API 金鑰而非直接暴露給代理.隔離技術比較, sandbox runtime（輕量, 低效能負擔）、containers（設定決定隔離強度）、gVisor（優異隔離, 中高效能負擔）、VMs（優異隔離, 高效能負擔）.建議的核心模式是代理透過掛載的 Unix socket 與外部代理程式溝通, 即使代理被入侵也無法任意外洩資料.

## 26. Hosting the Agent SDK 部署 Agent SDK

Agent SDK 產生並監督一個 `claude` CLI 子行程, 該子行程擁有 shell、工作目錄、以及磁碟上的 session 檔案.每個 session 對應一個子行程, 執行 N 個並行 session 代表 N 個子行程.三種狀態預設存在本機磁碟, session 逐字稿、CLAUDE.md 記憶檔案、工作目錄產物, 都不會在容器重啟、縮減規模、或移到不同節點時存活.

四種 session 模式涵蓋不同的生命週期需求, 一次性 session（為每個使用者任務建立容器並在完成後銷毀）、長期執行 session（執行持久的容器實例服務持續工作）、混合 session（一次性容器在啟動時從 SessionStore 補水並持久化更新回去）、多代理容器（一個容器內執行多個 SDK 子行程做緊密協作）.

正式環境考量包含, session 與狀態持久化（用 SessionStore 轉接器鏡射逐字稿到耐用儲存）、可觀測性（在容器或編排器層級設定 OTEL 環境變數）、驗證與密鑰（子行程從環境讀取 ANTHROPIC_API_KEY, 或透過 proxy 注入）、擴展與並行（每台主機的並行數受限於其 RAM 能容納多少子行程）、多租戶隔離（傳入 `settingSources: []` 加上每租戶工作目錄與 CLAUDE_CONFIG_DIR）.

## 27. Migrate to Claude Agent SDK 遷移到 Claude Agent SDK

Claude Code SDK 已改名為 Claude Agent SDK, 文件也已重新組織, 反映 SDK 除了編碼任務之外更廣泛的能力.套件名稱從 `@anthropic-ai/claude-code`（TS）與 `claude-code-sdk`（Python）分別改為 `@anthropic-ai/claude-agent-sdk` 與 `claude-agent-sdk`.

遷移步驟為解除安裝舊套件、安裝新套件、更新 import 陳述式、更新 package.json 或 requirements 中的相依套件名稱.Python 的型別 `ClaudeCodeOptions` 已改名為 `ClaudeAgentOptions`.

主要重大變更包含, 系統提示詞不再預設使用 Claude Code 的版本（v0.1.0 起預設用最小系統提示詞, 要取得舊行為需明確設定 `systemPrompt: { type: "preset", preset: "claude_code" }`）.設定來源預設值曾短暫變更後又還原, 目前省略 `settingSources` 等同載入使用者、專案、本機層級設定, 與 CLI 行為一致.

## 28. Agent SDK reference, Python Python SDK 參考

這是 Python Agent SDK 的完整 API 參考文件, 內容涵蓋兩種互動方式的選擇.`query()` 函式每次呼叫預設建立新 session, 適合一次性任務、獨立且不需要先前交流情境的任務、簡單的自動化腳本.`ClaudeSDKClient` 則沿用同一個 session, 支援中斷、hooks、自訂工具、自動延續對話, 適合需要 Claude 記住情境的持續對話情境.

文件詳細列出 `ClaudeAgentOptions` 的完整欄位（模型、權限模式、允許與拒絕工具清單、hooks、agents、mcp_servers、system_prompt、setting_sources、resume、fork_session、max_turns、max_budget_usd、effort、output_format、session_store、env、cwd 等）, 以及訊息型別（`SystemMessage`、`AssistantMessage`、`UserMessage`、`StreamEvent`、`ResultMessage`）、工具輸入輸出型別、`AgentDefinition`、`ToolAnnotations`、`HookMatcher`、`PermissionResultAllow`/`PermissionResultDeny` 等資料結構的完整定義.由於是逐項列舉的 API 參考格式, 本課程只摘要其分類架構, 詳細簽章請查閱官方文件.

## 29. Agent SDK reference, TypeScript TypeScript SDK 參考

這是 TypeScript Agent SDK 的完整 API 參考文件.SDK 會為你的平台以選用相依套件形式綁定原生 Claude Code 執行檔（例如 `@anthropic-ai/claude-agent-sdk-darwin-arm64`）, 因此不需要另外安裝 Claude Code.若用 `bun build --compile` 編譯成單一執行檔, 需要用 `extractFromBunfs()` 把平台執行檔抽取到實際路徑並傳給 `pathToClaudeCodeExecutable`.

文件涵蓋 `query()` 函式簽章與回傳的 query 物件（含 `streamInput()`、`setPermissionMode()`、`rewindFiles()`、`interrupt()` 等方法）、完整的 `Options` 型別定義、`SDKMessage` 聯合型別家族（`SDKSystemMessage`、`SDKAssistantMessage`、`SDKUserMessage`、`SDKResultMessage`、`SDKPartialAssistantMessage` 等）、`tool()` 與 `createSdkMcpServer()` 輔助函式、`AgentDefinition`、`HookCallback` 與各 hook 輸入輸出型別、`PermissionUpdate`、`SessionStore` 介面等.與 Python 參考相同, 由於是逐項列舉格式, 本課程摘要其分類架構, 詳細型別定義請查閱官方文件.

## 30. TypeScript SDK V2 session API 已移除

V2 session API 已不再支援, TypeScript Agent SDK 0.3.142 移除了 `unstable_v2_createSession`、`unstable_v2_resumeSession`、`unstable_v2_prompt`, 以及 `SDKSession` 與 `SDKSessionOptions` 型別.

V2 曾是實驗性的 session API, 免除了非同步產生器與 yield 協調的需求, 改用每回合各自的 `send()`/`stream()` 循環, API 介面簡化為三個概念, `createSession()`/`resumeSession()` 開始或延續對話、`session.send()` 送出訊息、`session.stream()` 取得回應.

要遷移, 應改用 `query()` API 加上其接受的 session 選項, 傳入 `AsyncIterable<SDKUserMessage>` 做多輪對話, 或用 `options.resume` 延續已儲存的 session.這頁文件僅保留供維護 Agent SDK 0.2.x 以下版本程式碼的人參考.
