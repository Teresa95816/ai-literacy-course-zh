# 第十一階段, 疑難排解與參考資料

## 1. Debug your configuration 除錯設定

當 Claude 忽略指示或設定的功能沒出現, 通常是檔案沒載入, 從非預期位置載入, 或被另一個檔案覆蓋.先執行 `/context` 檢視目前情境視窗的組成（系統提示詞, 記憶檔案, skills, MCP 工具, 對話訊息）確認 CLAUDE.md, rules, skill 描述是否存在.進一步用 `/memory`（已載入的 CLAUDE.md 與 rules）, `/skills`, `/agents`, `/hooks`, `/mcp`, `/permissions`, `/doctor`（設定診斷, 無效欄位, schema 錯誤）, `/status`（生效中的設定來源）個別檢查.

設定合併順序為 managed 永遠最優先, 其餘依 local, project, user 由近到遠覆寫, 部分設定也可被命令列旗標或環境變數覆寫.常見陷阱包含, hook 的 `matcher` 用了 JSON 陣列而非字串, 或用小寫工具名稱（比對區分大小寫）, MCP server 設定寫在 `.claude/` 底下而非 repo 根目錄的 `.mcp.json`, 子目錄 CLAUDE.md 需 Claude 用 Read 工具讀取該目錄下檔案時才會載入而非啟動時載入.用 `claude --safe-mode` 停用所有客製化（CLAUDE.md, skills, plugins, hooks, MCP, 自訂指令與代理）做乾淨測試, 若問題消失代表原因在其中之一.

## 2. Environment variables 環境變數參考

環境變數控制模型選擇, 驗證, 請求路由, 功能開關, 優先順序為環境變數高於 CLI 旗標/會話指令, 高於設定檔.主要分類包含, 驗證與 API 設定（`ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_AWS_WORKSPACE_ID`, `ANTHROPIC_FOUNDRY_API_KEY`）, 端點與路由（`ANTHROPIC_BASE_URL`, `ANTHROPIC_BEDROCK_BASE_URL`, `ANTHROPIC_VERTEX_BASE_URL`, `ANTHROPIC_FOUNDRY_BASE_URL`）, 模型設定（`ANTHROPIC_MODEL`, `ANTHROPIC_DEFAULT_HAIKU_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`, `ANTHROPIC_DEFAULT_OPUS_MODEL`）, 超時與效能（`API_TIMEOUT_MS` 預設 600000ms, `BASH_DEFAULT_TIMEOUT_MS` 預設 120000ms）.

其餘分類涵蓋 UI 與顯示（`CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN`, `CLAUDE_AX_SCREEN_READER`）, 推理與思考（`MAX_THINKING_TOKENS`, `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`）, 記憶與檢查點（`CLAUDE_CODE_DISABLE_AUTO_MEMORY`, `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING`）, 功能與工具開關（`CLAUDE_CODE_DISABLE_BUNDLED_SKILLS`, `CLAUDE_CODE_DISABLE_FAST_MODE`）, 代理與背景任務（`CLAUDE_CODE_DISABLE_AGENT_VIEW`）, 上下文與壓縮（`CLAUDE_CODE_AUTO_COMPACT_WINDOW`）, TLS 與憑證, 遙測與回饋（`DISABLE_TELEMETRY`, `DO_NOT_TRACK`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` 等同一次停用所有遙測相關項目）, 調試與日誌（`CLAUDE_CODE_DEBUG_LOG_LEVEL`）.環境變數在啟動時讀取, 修改後需重啟 Claude Code 才會生效.

## 3. Error reference 錯誤參考

依錯誤訊息分類查找對應解法.伺服器錯誤（來自推論供應商本身而非帳號或請求, 如 `API Error: 500`, `Repeated 529 Overloaded`, `Request timed out`）通常是暫時性, 稍後重試或用 `/model` 切換到負載較低的模型即可.用量限制錯誤（`You've hit your session limit`, `Credit balance is too low`, `Request rejected 429`）需檢查方案額度或帳戶餘額.驗證錯誤（`Not logged in`, `Invalid API key`, `This organization has been disabled`, `OAuth token revoked`）通常需要 `/login` 重新驗證或檢查是否有殘留的舊 `ANTHROPIC_API_KEY` 蓋過訂閱憑證.

網路錯誤（`Unable to connect to API`, SSL 憑證錯誤）需檢查代理與憑證設定.請求錯誤（`Prompt is too long`, `Request too large`, `Extra inputs are not permitted`, 模型相關限制）通常需要 `/compact` 或調整請求內容.Claude Code 對暫時性失敗（伺服器錯誤, 過載回應, 逾時, 暫時性 429, 斷線）會自動以指數退避重試最多 10 次, 可用 `CLAUDE_CODE_MAX_RETRIES`, `CLAUDE_CODE_RETRY_WATCHDOG`（CI 環境中對 429/529 無限重試）, `API_TIMEOUT_MS` 調整重試行為.

## 4. Tools reference 內建工具參考

完整列出 Claude Code 可用的內建工具, 工具名稱是 permission 規則, subagent 工具清單, hook matcher 中使用的確切字串.核心操作工具包含 `Read`（讀檔, 無需權限）, `Write`（建立或覆寫檔案, 需權限）, `Edit`（針對性修改, 需權限）, `Glob`（依樣式尋找檔案）, `Grep`（搜尋檔案內容）, `Bash`（執行 shell 指令, 需權限）, `PowerShell`（原生執行 PowerShell 指令, 需權限）, `NotebookEdit`（修改 Jupyter notebook 儲存格, 需權限）.

代理與任務管理工具包含 `Agent`（產生 subagent）, `TaskCreate`/`TaskGet`/`TaskList`/`TaskUpdate`（結構化任務清單, 取代舊版 `TodoWrite`）, `SendMessage`（傳訊給 agent team 隊友或回復 subagent）, `Workflow`（執行協調多個 subagent 的動態工作流程）.網路與外部工具包含 `WebFetch`（擷取 URL 內容, 需權限）, `WebSearch`（網頁搜尋, 需權限）, `ListMcpResourcesTool` 與 `ReadMcpResourceTool`（MCP 資源）, `ToolSearch`（在啟用 tool search 時搜尋並載入延遲載入的工具）.其他工具包含 `Skill`（在主對話中執行 skill）, `LSP`（透過語言伺服器提供程式碼智慧）, `Monitor`（背景執行指令並將每行輸出回饋給 Claude）, `Artifact`（發佈為 claude.ai 上的互動頁面, 需 Team/Enterprise 方案）, `AskUserQuestion`（提出選擇題釐清需求）.規則格式統一為 `ToolName(specifier)`, 例如 `Bash(npm run *)`, `Read(~/secrets/**)`, `WebFetch(domain:example.com)`.

## 5. Troubleshoot installation and login 安裝與登入疑難排解

安裝失敗依症狀對應解法, `command not found` 通常是安裝目錄不在 PATH 中（macOS/Linux 為 `~/.local/bin`）, 需加入 shell 設定檔.安裝腳本回傳 HTML 而非 shell script 時, 代表該區域不支援或網路問題, 可改用 Homebrew（`brew install --cask claude-code`）或 WinGet（`winget install Anthropic.ClaudeCode`）.TLS/SSL 連線錯誤通常需更新系統 CA 憑證或設定 `NODE_EXTRA_CA_CERTS` 指向企業憑證.低記憶體 Linux 主機安裝時被 OOM killer 終止（`Killed`）需加入至少 2GB swap 空間, Claude Code 需要至少 4GB 可用 RAM.

登入問題方面, `/logout` 後重新 `/login` 可解決多數驗證失敗.`OAuth error, Invalid code` 通常是登入碼過期或複製不完整.殘留的舊 `ANTHROPIC_API_KEY` 環境變數會蓋過訂閱憑證造成 `This organization has been disabled` 等錯誤, 需 `unset ANTHROPIC_API_KEY`.WSL2, SSH, 或容器環境中瀏覽器可能開在不同主機, 登入後貼上顯示的登入碼即可完成.Bedrock, Vertex, Foundry 憑證載入失敗通常是雲端 CLI 尚未在目前 shell 完成驗證（`aws sts get-caller-identity`, `gcloud auth application-default login`, `az login`）.

## 6. Troubleshooting 效能與穩定性疑難排解

處理效能, 穩定性, 搜尋相關問題.高 CPU 或記憶體用量可用 `/compact` 縮小情境, 用 `claude --safe-mode` 排除是否為 plugin, MCP, hook 造成, 記憶體持續偏高可用 `/heapdump` 產出 JS heap 快照到桌面協助診斷.自動壓縮出現 thrashing 錯誤（壓縮後立刻又被大量輸出填滿）代表某個檔案或工具輸出反覆撐爆情境視窗, 可請 Claude 分段讀取大檔案, 用有針對性的 `/compact` 說明, 或把工作移到獨立情境視窗的 subagent.

指令卡住或無回應時按 Ctrl+C 嘗試取消, 若無效可關閉終端機後用 `claude --resume` 在同目錄接續 session, 不會遺失對話.編輯器內建終端機出現亂碼字元通常是 GPU 渲染問題, 執行 `/terminal-setup` 關閉 GPU 加速.搜尋功能異常（Search, @file, skills 找不到檔案）通常是內建 `ripgrep` 二進位檔無法在系統上執行, 需安裝系統對應套件並設定 `USE_BUILTIN_RIPGREP=0`.遇到未列出的問題, 執行 `/doctor` 做一次性完整診斷, 或用 `/feedback` 直接回報.
