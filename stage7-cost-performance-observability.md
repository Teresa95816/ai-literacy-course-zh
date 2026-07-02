# 第七階段, 成本, 效能與可觀測性

## 1. 有效管理成本

Claude Code 依 API token 消耗計費.訂閱方案（Pro、Max、Team、Enterprise）定價請見 claude.com/pricing, 每位開發者的實際花費因模型選擇、程式碼庫大小、以及使用模式（如同時執行多個實例或自動化）而有很大差異.在企業部署中, 平均每位開發者每個活躍工作日約 13 美元, 每月約 150 到 250 美元, 90% 使用者的每日花費維持在 30 美元以下.

### 追蹤你的成本

`/usage` 指令頂端的 Session 區塊顯示目前 session 的詳細 token 用量統計, 金額是依 token 數量在本機估算的數字, 可能與實際帳單有差異, 正式計費請見 Claude Console 的 Usage 頁面.在 Pro、Max、Team、Enterprise 方案上, `/usage` 也會顯示計入方案額度的用量細分, 依 skills、subagents、plugins、以及個別 MCP servers 歸類, 各自以佔總量的百分比呈現, 按 `d` 或 `w` 可切換最近 24 小時與最近 7 天.

### 為團隊管理成本

使用 Claude API 時, 可設定 workspace 支出上限, 管理者可在 Console 中檢視成本與用量報告.在 Pro 與 Max 方案上, 可用 `/usage-credits` 指令設定用量額度的每月支出上限.首次以 Claude Console 帳號驗證時, 系統會自動建立一個名為 "Claude Code" 的 workspace, 提供組織內所有 Claude Code 用量的集中成本追蹤與管理.

### 費率限制建議

依團隊規模建議每位使用者的 Token Per Minute（TPM）與 Request Per Minute（RPM）,

| 團隊規模 | 每人 TPM | 每人 RPM |
| :-- | :-- | :-- |
| 1 到 5 人 | 200k 到 300k | 5 到 7 |
| 5 到 20 人 | 100k 到 150k | 2.5 到 3.5 |
| 20 到 50 人 | 50k 到 75k | 1.25 到 1.75 |
| 50 到 100 人 | 25k 到 35k | 0.62 到 0.87 |
| 100 到 500 人 | 15k 到 20k | 0.37 到 0.47 |
| 500 人以上 | 10k 到 15k | 0.25 到 0.35 |

團隊規模越大, 每人 TPM 反而越低, 因為大型組織中同時使用 Claude Code 的使用者比例較低.這些限制是以組織層級而非個別使用者層級套用.

### agent team 的 token 成本

agent teams 會產生多個各自有獨立情境視窗的 Claude Code 實例, token 用量隨活躍 teammate 數量與各自執行時間增加.控制成本的做法包含, teammate 使用 Sonnet, 團隊維持小規模, 派送提示詞保持聚焦, 工作完成後結束 teammate.

### 降低 token 用量

token 成本隨情境大小增加, Claude Code 會自動透過 prompt caching 與自動壓縮優化成本.可採取的策略包含, 主動管理情境（切換不相關工作時用 `/clear`, 用 `/compact` 加上自訂指示告訴 Claude 要保留什麼）, 選擇合適的模型（Sonnet 處理多數編碼任務且比 Opus 便宜, Opus 保留給複雜架構決策）, 降低 MCP server 額外負擔（優先使用 CLI 工具如 gh、aws, 停用未使用的 server）, 安裝型別語言的程式碼智慧外掛, 把處理工作卸載給 hooks 與 skills（例如用 hook 過濾冗長的日誌檔案, 只回傳符合條件的行）, 把 CLAUDE.md 中的細節指示移到 skills（CLAUDE.md 應保持在 200 行以內）, 調整 extended thinking 的 effort 等級, 把冗長操作委派給 subagents, 撰寫具體的提示詞, 使用 plan mode 處理複雜任務並及早修正方向.

## 2. Claude Code 如何使用 prompt caching

prompt caching 讓 Claude Code 更快也更省成本.沒有快取時, API 每回合都要重新處理完整歷史記錄, 有快取時則重複利用已處理過的部分, 只對變動的部分做新的運算.

### 快取的組織方式

每次在 Claude Code 中送出訊息都會產生一個新的 API 請求, 模型在請求之間沒有記憶, 因此 Claude Code 會重新送出完整情境, 新內容附加在最後, 也就是說每個請求中大部分內容與前一個請求相同.API 依請求開頭（稱為 prefix）比對是否符合最近處理過的內容, 比對是完全一致的, 因此 prefix 中任何位置的改動都會導致之後的一切重新運算.

| 層級 | 內容 | 何時改變 |
| :-- | :-- | :-- |
| 系統提示詞 | 核心指示、工具定義、輸出風格 | 已載入的工具定義集合改變, 或 Claude Code 升級時 |
| 專案情境 | CLAUDE.md、auto memory、無範圍規則 | session 開始時, 或 /clear、/compact 之後 |
| 對話內容 | 你的訊息、Claude 的回應、工具結果 | 每回合 |

模型與 effort 等級也是快取鍵的一部分但不在提示詞文字中, 每個模型有各自的快取, 切換模型會讓下一個請求完全不命中快取.

### 會讓快取失效的動作

切換模型, 改變 effort 等級, 開啟 fast mode（成本只發生一次, 之後保留 header 即可維持快取）, 連接或斷開 MCP server（若工具定義被延遲載入則不受影響, 若載入到 prefix 中則會失效）, 啟用或停用 plugin（提供 MCP servers 的 plugin 依相同規則, 其餘元件類型只會附加內容不影響快取）, 對整個工具加入拒絕規則, 壓縮對話（`/compact` 會取代訊息歷史, 因此使對話層失效, 但系統提示詞層與專案情境層可重複使用）, 升級 Claude Code 版本.

### 會保留快取的動作

編輯儲存庫中的檔案, session 中途編輯 CLAUDE.md（編輯不會失效但也不會生效, 需等下次 /clear、/compact 或重啟）, 改變輸出風格, 改變權限模式, 呼叫 skills 與 commands, 執行 `/recap`, 回退對話（`/rewind`）, 派送 subagent（父層快取不受影響）.

### 快取存續時間

快取的 prefix 會在閒置一段時間後過期, 每次命中快取的請求都會重設計時器.API 提供兩種存續時間, 五分鐘 TTL 與一小時 TTL（快取寫入費率較高）.在 Claude 訂閱方案上, Claude Code 會自動要求一小時 TTL, 因為用量已包含在方案中.在 API 金鑰或第三方供應商上, 預設維持較便宜的五分鐘 TTL, 可設定 `ENABLE_PROMPT_CACHING_1H=1` 選擇加入一小時 TTL.

### 檢查快取效能

API 每次回應都會回報兩個 token 計數, `cache_creation_input_tokens`（本回合寫入快取的 token, 依快取寫入費率計費）與 `cache_read_input_tokens`（本回合從快取讀取的 token, 費率約為標準輸入費率的 10%）.

## 3. 探索情境視窗

這篇文件是官方網站上的一個互動式視覺化模擬, 展示 Claude Code 的情境視窗在一次 session 中如何逐步填滿, 包含哪些內容會自動載入、每次檔案讀取要付出的成本、以及規則與 hooks 何時觸發.以下摘要其書面說明.

### 時間軸展示的內容

在你輸入任何內容之前, CLAUDE.md、auto memory、MCP 工具名稱、以及 skill 描述都已載入情境.隨著 Claude 工作, 每次檔案讀取都會增加情境, 路徑範圍的規則會隨符合的檔案自動載入, PostToolUse hook 會在每次編輯後觸發.後續提示詞中, subagent 會在自己獨立的情境視窗中處理研究工作, 因此大量檔案讀取不會進入你的主情境, 只有摘要與一小段中繼資料會回傳.最後, `/compact` 會用結構化摘要取代對話, 大部分啟動內容會自動重新載入.

### 壓縮後保留的內容

| 機制 | 壓縮後狀態 |
| :-- | :-- |
| 系統提示詞與輸出風格 | 不變, 不屬於訊息歷史 |
| 專案根目錄 CLAUDE.md 與無範圍規則 | 從磁碟重新注入 |
| Auto memory | 從磁碟重新注入 |
| 帶有 paths 前置資料的規則 | 遺失, 直到再次讀取符合的檔案 |
| 子目錄中的巢狀 CLAUDE.md | 遺失, 直到再次讀取該子目錄中的檔案 |
| 已呼叫的 skill 內容 | 重新注入, 每個 skill 上限 5000 token, 總計上限 25000 token |
| Hooks | 不適用, hooks 以程式碼執行, 不屬於情境 |

### 情境填滿時

Claude Code 會在接近上限時自動壓縮, 你也可以主動處理, 用帶焦點的方式壓縮（`/compact focus on the auth bug fix`）, 切換不相關工作時清除（`/clear`）, 把大量讀取委派給 subagent.若需要更大的視窗而非更小的對話, Fable 5、Sonnet 5、Opus 4.6 以上、以及 Sonnet 4.6 支援 100 萬 token 的情境視窗.執行 `/context` 可查看目前實際情境用量的即時分類, 執行 `/memory` 可檢查啟動時載入了哪些 CLAUDE.md 與 auto memory 檔案.

## 4. Monitoring 監控

透過 OpenTelemetry（OTel）匯出遙測資料, 追蹤組織內的 Claude Code 用量、成本、與工具活動.Claude Code 以標準 metrics 協定匯出指標, 以 logs/events 協定匯出事件, 並可選擇性地以 traces 協定匯出分散式追蹤.

### 快速設定

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer your-token"
claude
```

管理者可透過受管理設定檔為所有使用者集中設定 OpenTelemetry, 環境變數具有高優先權, 使用者無法覆寫.

### 常用設定變數

`CLAUDE_CODE_ENABLE_TELEMETRY` 啟用遙測收集（必要）, `OTEL_METRICS_EXPORTER` 指標匯出器類型（console、otlp、prometheus、none）, `OTEL_LOGS_EXPORTER` 日誌事件匯出器類型, `OTEL_EXPORTER_OTLP_ENDPOINT` OTLP 收集端點, `OTEL_LOG_USER_PROMPTS` 啟用記錄使用者提示詞內容（預設停用）, `OTEL_LOG_TOOL_DETAILS` 啟用記錄工具參數與輸入引數.

### 分散式追蹤 traces（beta）

分散式追蹤會匯出 span, 把每個使用者提示詞連結到它觸發的 API 請求與工具執行.預設關閉, 需同時設定 `CLAUDE_CODE_ENABLE_TELEMETRY=1` 與 `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`, 並設定 `OTEL_TRACES_EXPORTER`.每個使用者提示詞會開啟一個 `claude_code.interaction` 根 span, API 呼叫、工具呼叫、hook 執行都記錄為其子 span.

### 可用的指標

| 指標名稱 | 說明 | 單位 |
| :-- | :-- | :-- |
| claude_code.session.count | CLI session 啟動次數 | count |
| claude_code.lines_of_code.count | 修改的程式碼行數 | count |
| claude_code.pull_request.count | 建立的 pull request 數量 | count |
| claude_code.commit.count | 建立的 git commit 數量 | count |
| claude_code.cost.usage | Claude Code session 的成本 | USD |
| claude_code.token.usage | 使用的 token 數量 | tokens |
| claude_code.code_edit_tool.decision | 程式碼編輯工具的權限決定次數 | count |
| claude_code.active_time.total | 總主動使用時間 | s |

### 事件

Claude Code 透過 OpenTelemetry logs/events 匯出多種事件, 包含使用者提示詞事件、助理回應事件、工具結果事件、API 請求事件、API 錯誤事件、API 拒絕事件、以及工具決定事件.每個提示詞產生的 `prompt.id` 可用來把所有相關事件串連回同一個觸發提示詞, 便於追蹤與稽核.

## 5. 用分析儀表板追蹤團隊用量

Claude Code 提供分析儀表板, 協助組織了解開發者使用模式、追蹤貢獻指標、並衡量 Claude Code 對工程效率的影響.

| 方案 | 儀表板網址 | 涵蓋內容 |
| :-- | :-- | :-- |
| Claude for Teams / Enterprise | claude.ai/analytics/claude-code | 用量指標、含 GitHub 整合的貢獻指標、排行榜、資料匯出 |
| API（Claude Console） | platform.claude.com/claude-code | 用量指標、支出追蹤、團隊洞察 |

### Team 與 Enterprise 分析

管理者與 Owner 可檢視儀表板, 包含用量指標（接受的程式碼行數、建議接受率、每日活躍使用者與 session 數）, 貢獻指標（透過 GitHub 整合追蹤 Claude Code 協助完成的 PR 與程式碼行數）, 排行榜（依 Claude Code 使用量排名的前幾名貢獻者）, 資料匯出（下載貢獻資料為 CSV）.

啟用貢獻指標需要 GitHub 管理者安裝 Claude GitHub App, 並由 Claude Owner 在管理設定中啟用 Claude Code analytics 與 GitHub analytics.對於已啟用 Zero Data Retention 的組織, 貢獻指標不可用, 儀表板只顯示用量指標.

### 摘要指標

Claude Code 協助的 PR 數量、Claude Code 協助的程式碼行數（只計算有效行, 排除空行與僅有括號或瑣碎標點的行）、Claude Code 協助的 PR 百分比、建議接受率、使用者已接受的程式碼行數.這些指標刻意保守, 只計算高信心確定 Claude Code 有參與的行數與 PR.

### PR 歸屬判定

已合併的 PR 中, 若包含至少一行在 Claude Code session 中撰寫的程式碼, 會被標記為 "with Claude Code".判定流程為, 從 PR 差異中擷取新增的行, 找出在符合時間窗（PR 合併日期前 21 天到後 2 天）內編輯過相符檔案的 Claude Code session, 用多種策略比對 PR 行與 Claude Code 輸出.鎖定檔案、產生的程式碼、建置目錄、測試固定資料等會自動排除在分析之外.

### API 客戶的分析

使用 Claude Console 的 API 客戶可在 platform.claude.com/claude-code 存取分析, 需要 UsageView 權限.Console 儀表板顯示已接受的程式碼行數、建議接受率、每日活躍使用者與 session 數的活動圖表、以及每日 API 成本的支出圖表.
