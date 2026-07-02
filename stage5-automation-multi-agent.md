# 第五階段, 自動化與多代理協作

## 1. 讓 Claude 朝目標持續工作 /goal

`/goal` 指令設定一個完成條件, 之後 Claude 會在每個回合結束後自動檢查條件是否成立, 若未成立就自動開始下一回合, 不需要你每次手動催促.需要 Claude Code v2.1.139 以上版本.

適合用在有明確可驗證結果的大型工作, 例如, 把整個模組遷移到新 API 直到每個呼叫點都能編譯且測試通過, 依照設計文件實作直到所有驗收條件成立, 把大檔案拆成多個模組直到每個檔案都小於指定大小, 逐一處理已標記的 issue 待辦清單直到清空為止.

### 三種讓 session 持續運作的方式比較

| 方式 | 下一回合何時開始 | 何時停止 |
| :-- | :-- | :-- |
| `/goal` | 前一回合結束後 | 模型確認條件已達成 |
| `/loop` | 時間間隔到達後 | 你手動停止, 或 Claude 判斷工作已完成 |
| Stop hook | 前一回合結束後 | 你自己的腳本或提示詞決定 |

`/goal` 與 Stop hook 都是在每回合後觸發.`/goal` 是 session 範圍內的捷徑, 只在目前對話中生效.Stop hook 則寫在設定檔中, 適用於該範圍內的每個 session, 可以執行腳本做決定性判斷, 也可以用提示詞讓模型判斷.

auto mode 只在單一回合內自動核准工具呼叫, 並不會啟動新回合, Claude 會在判斷工作完成時自行停止.`/goal` 則是額外加上一個評估者, 在每回合後用你設定的條件檢查, 完成與否是由另一個全新的模型判斷, 而非執行工作的那個模型.兩者互補, auto mode 移除逐工具的提示, `/goal` 移除逐回合的提示.

### 使用 /goal

一個 session 同時只能有一個進行中的目標.

設定目標, 直接輸入條件, 若已有進行中的目標會被取代.

```text
/goal all tests in test/auth pass and the lint step is clean
```

設定目標後會立即以該條件為指令開始一個回合, 不需要另外送出提示詞.目標進行中時畫面會顯示 `◎ /goal active` 指示器.

撰寫有效的條件要點, 一個可衡量的結束狀態（測試結果、建置結束碼、檔案數量、佇列清空）, 一個明確的檢查方式（例如 `npm test` 結束碼為 0）, 以及過程中不應改變的限制條件.條件最長可達 4000 個字元.若要限制目標最長執行時間, 可在條件中加入回合數或時間子句, 例如 `or stop after 20 turns`.

檢查狀態, 不帶參數執行 `/goal` 可看到目前狀態, 包含條件內容、已執行時間、已評估回合數、目前 token 花費、評估者最近一次給出的理由.

清除目標, 執行 `/goal clear` 可在條件達成前移除進行中的目標, `stop` `off` `reset` `none` `cancel` 都是 `clear` 的別名.執行 `/clear` 開始新對話也會一併移除進行中的目標.

回復 session 時, 若目標仍在進行中, 用 `--resume` 或 `--continue` 回復 session 時條件會被保留, 但回合數、計時器、token 花費基準會重新歸零.已達成或已清除的目標不會被回復.

`/goal` 也可在非互動模式、桌面應用程式、以及 Remote Control 中使用.

### 評估運作方式

`/goal` 本質上是一個 session 範圍的提示詞型 Stop hook.每次 Claude 完成一回合, 條件與目前對話內容會送給你設定的小型快速模型（預設為 Haiku）判斷, 回傳是或否與簡短理由."否" 會告訴 Claude 繼續工作並把理由當作下一回合的指引,"是" 則清除目標並在對話紀錄中留下已達成的紀錄.評估者不會呼叫任何工具, 只能根據 Claude 已經在對話中呈現的內容做判斷.

### 使用限制

`/goal` 只能在已通過信任對話框的工作區中執行, 因為評估者是 hooks 系統的一部分.當 `disableAllHooks` 在任何設定層級被啟用, 或 `allowManagedHooksOnly` 在管理設定中被啟用時, `/goal` 也無法使用.

## 2. 排程執行提示詞 /loop

排程任務讓 Claude 依照時間間隔自動重新執行提示詞, 用來輪詢部署狀態、盯著 PR、確認長時間建置的結果, 或是在 session 中設定稍後提醒.若要對事件即時反應而非輪詢, 可改用 channels, 若要讓 session 持續工作到條件達成而非依時間間隔, 可改用 `/goal`.

任務是 session 範圍的, 存在於目前對話中, 開新對話就會停止.用 `--resume` 或 `--continue` 回復時, 尚未過期的任務會被還原, 包含 7 天內建立的重複性任務, 以及排定時間尚未到達的一次性任務.

### 三種排程方式比較

| | 雲端 routines | 桌面應用 | `/loop` |
| :-- | :-- | :-- | :-- |
| 執行位置 | Anthropic 雲端 | 你的機器 | 你的機器 |
| 需要機器開機 | 否 | 是 | 是 |
| 需要開啟的 session | 否 | 否 | 是 |
| 重啟後持續 | 是 | 是 | 未過期時, 於 resume 時還原 |
| 存取本機檔案 | 否（乾淨 clone） | 是 | 是 |
| MCP servers | 每個任務個別設定 connector | 設定檔與 connector | 繼承 session |
| 權限提示 | 無（自主執行） | 每個任務可設定 | 繼承 session |
| 自訂排程 | 透過 CLI 的 `/schedule` | 是 | 是 |
| 最小間隔 | 1 小時 | 1 分鐘 | 1 分鐘 |

雲端任務適合不依賴你機器就要可靠執行的工作, 桌面任務適合需要存取本機檔案與工具的工作, `/loop` 適合 session 中的快速輪詢.

### 用 /loop 重複執行提示詞

`/loop` 是內建的 skill, 依照你提供的內容決定行為.

| 提供內容 | 範例 | 行為 |
| :-- | :-- | :-- |
| 間隔與提示詞 | `/loop 5m check the deploy` | 固定排程執行 |
| 只有提示詞 | `/loop check the deploy` | 每次迭代由 Claude 自行選擇間隔 |
| 只有間隔或都沒有 | `/loop` | 執行內建維護提示詞, 或你自訂的 `loop.md` |

也可以把 skill 當作提示詞傳入, 例如 `/loop 20m /review-pr 1234`, 每次迭代重新執行該 skill.

固定間隔時, Claude 會把你給的間隔轉成 cron 表達式並確認排程週期, 秒數會無條件進位到最接近的分鐘, 因為 cron 的最小粒度是一分鐘.不指定間隔時, Claude 會依觀察到的狀況動態選擇一到六十分鐘的延遲, 例如建置進行中時等待較短, 沒有待辦事項時等待較長, 此時 Claude 可能會直接使用 Monitor 工具而非重新執行提示詞輪詢.

不提供提示詞時, Claude 使用內建維護提示詞, 依序處理, 對話中尚未完成的工作、目前分支 PR 的檢視留言與失敗的 CI 與合併衝突、以及在沒有其他待辦事項時進行清理（例如抓錯或簡化）.Claude 不會主動開展對話中沒有授權的新工作, 推送或刪除等不可逆操作只有在延續對話中已授權的內容時才會進行.

可以用 `loop.md` 檔案取代內建維護提示詞, 專案層級放在 `.claude/loop.md`, 使用者層級放在 `~/.claude/loop.md`, 優先使用專案層級.編輯後下一次迭代就會生效, 內容超過 25000 位元組會被截斷.

停止 loop, 在等待下次迭代時按 `Esc` 即可清除待執行的喚醒, 用自然語言排程的任務不受 `Esc` 影響.在自行選擇間隔模式下, Claude 也可以在判斷工作確實完成後自行決定不再排下一次喚醒, 藉此結束 loop.

### 設定一次性提醒

用自然語言描述即可, 例如 "remind me at 3pm to push the release branch", Claude 會排定一次性任務, 執行後自動刪除.

### 管理排程任務

可用自然語言請 Claude 列出或取消任務, 底層使用三個工具, `CronCreate` 建立新任務（接受標準五欄位 cron 表達式）, `CronList` 列出所有已排程任務, `CronDelete` 依 ID 取消任務.每個任務有 8 個字元的 ID, 一個 session 最多可持有 50 個排程任務.

排程器每秒檢查一次到期任務並以低優先權排入佇列, 只會在你的回合之間觸發, 不會打斷 Claude 正在進行的回應.所有時間都以你的本機時區解讀.

為避免所有 session 在同一時刻同時打 API, 排程器會加入確定性的偏移量, 重複性任務最多延後 30 分鐘（間隔小於一小時者則延後至多間隔的一半）, 整點或半點排定的一次性任務最多提早 90 秒.

重複性任務會在建立後 7 天自動過期, 會先執行最後一次再自動刪除, 藉此限制被遺忘的 loop 能運行多久.若需要更長效的排程, 應使用 routines 或桌面排程任務.

### cron 表達式參考

| 範例 | 意義 |
| :-- | :-- |
| `*/5 * * * *` | 每 5 分鐘 |
| `0 * * * *` | 每小時整點 |
| `0 9 * * *` | 每天本機時間上午 9 點 |
| `0 9 * * 1-5` | 平日上午 9 點 |
| `30 14 15 3 *` | 3 月 15 日下午 2 點 30 分 |

設定 `CLAUDE_CODE_DISABLE_CRON=1` 可完全停用排程器.

session 範圍的排程有其限制, 只有在 Claude Code 執行且閒置時才會觸發, 錯過的觸發不會補跑, 開新對話會清除所有 session 範圍的任務.

## 3. 在 Claude Code Desktop 排程重複性任務

排程任務會在你選定的時間與頻率自動開啟新 session, 適合每天的程式碼檢視、依賴套件檢查、或每天早上彙整行事曆與收件匣的簡報.桌面應用的 Routines 頁面可以建立本機排程任務, 也可以建立遠端 routines, 本機任務在你的機器上執行, 可直接存取檔案與工具, 但只在應用程式開啟且電腦未睡眠時才會觸發, 遠端 routines 則在 Anthropic 管理的雲端基礎架構上執行, 即使電腦關機也能運作, 也可以由 API 呼叫或 GitHub 事件觸發.

### 建立排程任務

點選側邊欄的 Routines, 點選 New routine 並選擇 Local, 設定名稱、描述、指示內容（寫法與在提示框輸入訊息相同）、以及排程.名稱會轉成小寫連字號形式作為磁碟上的資料夾名稱, 必須在你的任務中唯一.儲存前需要指定工作資料夾, 若尚未信任該資料夾, 桌面應用會先提示信任.

也可以直接在任何 session 中用自然語言描述, 例如 "set up a daily code review that runs every morning at 9am" 會建立重複性任務, "remind me at 3pm tomorrow to check the deploy" 則建立一次性任務.

### 排程選項

Manual 只在你點選 Run now 時執行, Hourly 每小時執行一次, Daily 有時間選擇器（預設上午 9 點）, Weekdays 與 Daily 相同但跳過週末, Weekly 有時間與星期選擇器.其他選擇器沒提供的間隔（例如每 15 分鐘、每月第一天）可直接請 Claude 用自然語言設定.

### 運作方式

排程任務在應用程式開啟時每分鐘檢查一次排程, 到期時會開啟全新的 session, 與你手動開啟的 session 互不影響.每個任務在排定時間後會有數分鐘的延遲以分散 API 流量.觸發時會有桌面通知, 並在側邊欄的 Scheduled 區塊中出現新 session.只有在應用程式執行且電腦未睡眠時才會運作, 若電腦睡眠跨過排定時間, 該次執行會被跳過, 可在設定中開啟 Keep computer awake 避免閒置睡眠.

### 錯過的執行

應用程式啟動或電腦喚醒時, 桌面應用會檢查過去 7 天內是否有任務錯過執行, 若有, 只會針對最近一次錯過的時間執行一次補跑, 更早的都會被捨棄, 並顯示通知.撰寫提示詞時要考慮到這點, 例如可以加上守門條件, "若現在已過下午 5 點, 就跳過檢視只回報摘要".

### 權限與管理

每個任務有自己的權限模式, `~/.claude/settings.json` 中的 allow 規則同樣適用.若任務以 Ask 模式執行且需要未授權的工具, 該次執行會停滯直到你核准.建議建立後先點 Run now 觀察權限提示並選擇永遠允許, 之後的執行就會自動核准相同工具.

任務詳細頁面可以, 立即執行 Run now, 在 Active 與 Paused 間切換狀態, 編輯指示與排程, 檢視每次執行歷史（包含被跳過的原因）, 檢視並撤銷已允許的權限, 刪除任務（可選擇是否一併刪除磁碟上的 `SKILL.md` 檔案）.任務也可以在 session 中透過 `update_scheduled_task` 這個 MCP 工具自行修改排程或提示詞.任務的提示詞存放於 `~/.claude/scheduled-tasks/<task-name>/SKILL.md`, 排程、資料夾、模型等設定不在此檔案中.

## 4. 用 routines 自動化工作

routine 是一份已儲存的 Claude Code 設定, 包含提示詞、一個或多個儲存庫、以及一組 connector, 打包後自動執行.routines 在 Anthropic 管理的雲端基礎架構上執行, 即使筆電關機也能持續運作.目前為研究預覽階段.

每個 routine 可以附加一種或多種觸發方式, Scheduled 依固定週期（每小時、每晚、每週）或在指定的未來時間執行一次, API 透過對專屬端點送出 HTTP POST 並附帶 bearer token 觸發, GitHub 則依儲存庫事件（例如 pull request 或 release）自動執行.同一個 routine 可以合併多種觸發方式.

routines 適用於 Pro、Max、Team、Enterprise 方案並需開啟 Claude Code on the web, 可在 claude.ai/code/routines 或 CLI 的 `/schedule` 建立與管理.Team 與 Enterprise 的 Owner 可在管理後台的 Routines 開關中為所有成員停用.

### 應用案例

積壓工作維護, 排程觸發於每個工作夜晚, 透過 connector 讀取自上次執行後新開的 issue, 套用標籤、依程式碼區域指派負責人, 並將摘要發布到 Slack.警示分流, 監控工具在錯誤超過閾值時呼叫 routine 的 API 端點, routine 拉取堆疊追蹤, 與近期 commit 關聯, 開出附有修復建議的草稿 PR.客製化程式碼檢視, GitHub 觸發於 `pull_request.opened`, 依團隊自訂檢查清單留下行內註解.部署驗證, CD pipeline 在每次正式環境部署後呼叫 API 端點進行驗證.文件漂移偵測, 每週掃描已合併的 PR, 標記過時文件並開出更新 PR.函式庫移植, GitHub 觸發於合併的 PR, 將變更移植到另一個語言的對應函式庫.

### 建立 routine

可從 claude.ai/code/routines 網頁、桌面應用（選擇 Remote, 選擇 Local 則會建立本機的桌面排程任務）、或 CLI 建立, 三者共用同一組雲端帳號資料.routines 以完整的雲端 session 自主執行, 沒有權限模式選擇器, 執行期間不會出現核准提示, 可以執行 shell 指令、使用已提交到儲存庫的 skills、以及呼叫任何包含的 connector.

routine 屬於個人帳號, 不與團隊成員共用, 並計入你帳號每日的執行額度.透過你連結的 GitHub 身分或 connector 所做的一切都會顯示為你本人, commit 與 PR 會帶有你的 GitHub 使用者名稱.

建立時需要, 命名並撰寫提示詞（提示詞必須自成一體, 因為是自主執行）, 選擇一個或多個 GitHub 儲存庫（每次執行都會從預設分支重新 clone, Claude 只能建立 `claude/` 開頭的分支, 除非啟用允許不受限推送）, 選擇雲端環境（控制網路存取層級、環境變數、安裝腳本）, 選擇觸發方式, 檢視 connector 與權限分頁.

### 設定觸發方式

排程觸發可選預設頻率或指定未來單一時間點的一次性執行, 一次性執行後會自動停用, 不計入每日 routine 執行上限.API 觸發需為 routine 產生專屬 URL 與 token, token 只顯示一次, 需妥善保存, 呼叫方式是對 `/fire` 端點送出 POST 並在 `Authorization: Bearer` 標頭帶入 token, 請求主體可帶入 `text` 欄位傳遞情境資訊.GitHub 觸發需先安裝 Claude GitHub App, 可訂閱 pull request 或 release 事件, 並可用作者、標題、內文、分支、標籤、是否草稿、是否已合併等欄位篩選.

### 管理與限制

routine 詳細頁面可立即執行、暫停或恢復排程、編輯設定、刪除.綠色執行狀態只代表 session 正常啟動並結束, 不代表提示詞中的任務真的成功, 應開啟該次執行閱讀對話紀錄確認.

connector 預設全部包含最新連結的 MCP connector, 可移除不需要的以限縮存取範圍.環境的網路存取分為 Trusted（預設允許清單）與 Full（無限制）, 也可自訂允許網域.

routines 依帳號用量計算訂閱用量, 並額外設有每日執行次數上限, 一次性執行不計入此上限.

## 5. 用 channels 將事件推送進執行中的 session

channel 是一個 MCP server, 可以把訊息、警示、webhook 推送進正在執行的 Claude Code session, 讓 Claude 在你不在終端機前時也能對外部事件做出反應.channel 可以是雙向的, Claude 讀取事件後透過同一個 channel 回覆, 就像聊天橋接器一樣.事件只會在 session 開啟時送達, 若要全天候運作需在背景程序或持續開啟的終端機中執行 Claude.channel 需要 Claude Code v2.1.80 以上版本, 需要透過 claude.ai 或 Console API key 驗證, 不支援 Bedrock、Vertex AI、Microsoft Foundry, 目前為研究預覽階段.

channel 以外掛形式安裝並用你自己的憑證設定, 研究預覽階段內建支援 Telegram、Discord、iMessage.

### 已支援的 channel

Telegram, 在 BotFather 建立機器人取得 token, 執行 `/plugin install telegram@claude-plugins-official`, 執行 `/telegram:configure <token>`, 以 `claude --channels plugin:telegram@claude-plugins-official` 重啟, 傳訊息給機器人取得配對碼, 執行 `/telegram:access pair <code>` 完成配對, 再執行 `/telegram:access policy allowlist` 鎖定只允許自己傳訊息.

Discord, 在開發者後台建立應用程式與機器人, 開啟 Message Content Intent, 邀請機器人加入伺服器並授予必要權限, 安裝外掛並設定 token, 重啟並私訊機器人取得配對碼完成配對.

iMessage, 直接讀取本機 Messages 資料庫並透過 AppleScript 送出回覆, 僅支援 macOS, 不需要機器人 token 或外部服務, 需授予完整磁碟存取權限, 傳訊息給自己即可略過存取控制, 其他聯絡人需以 `/imessage:access allow <handle>` 加入允許清單.

### 快速入門與安全性

fakechat 是官方提供的示範 channel, 在本機開啟聊天介面, 不需驗證任何東西.安全性方面, 每個已核准的 channel 外掛都維護一份寄件者允許清單, 只有清單內的 ID 能推送訊息.你也能以 `--channels` 控制每個 session 啟用哪些 server, 組織可透過 `channelsEnabled` 這個管理設定控制可用性.

### 與其他功能比較

| 功能 | 作用 | 適合情境 |
| :-- | :-- | :-- |
| Claude Code on the web | 在全新雲端沙盒中執行任務 | 委派自成一體且稍後查看的非同步工作 |
| Claude in Slack | 從頻道或討論串的 @Claude 提及啟動 web session | 直接從團隊對話情境啟動任務 |
| 標準 MCP server | Claude 於任務中查詢, 不會主動推送進 session | 讓 Claude 依需求存取或查詢系統 |
| Remote Control | 從 claude.ai 或行動應用程式操控本機 session | 離開座位時仍能操控進行中的 session |

channels 補上的正是把非 Claude 來源的事件推送進已在執行的本機 session 這個空缺.

## 6. Channels 參考文件

這篇是給想要自行打造 channel 的開發者的技術參考文件, 內容涵蓋 MCP server 的能力宣告、通知格式、雙向回覆工具、寄件者過濾、以及權限中繼機制, 原文附有大量 TypeScript 程式碼範例（如何用 `@modelcontextprotocol/sdk` 建立一個監聽 HTTP webhook 並推送進 Claude session 的最小伺服器）, 內容較長且偏工程實作細節, 以下摘要重點.

channel 是一個在與 Claude Code 同一台機器上執行的 MCP server, 由 Claude Code 以子行程方式啟動並透過 stdio 溝通.聊天平台（Telegram、Discord）由外掛在本機輪詢平台 API 取得新訊息, webhook 類型（CI、監控）則由伺服器監聽本機 HTTP 埠, 外部系統對該埠送出 POST.

建立 channel 的最低需求是, 宣告 `claude/channel` 這個 capability 讓 Claude Code 註冊通知監聽器, 在事件發生時送出 `notifications/claude/channel` 通知, 以及以 stdio transport 連線.通知格式包含 `content`（事件內容, 會成為 `<channel>` 標籤的內容本文）以及可選的 `meta`（每個鍵值對會成為標籤上的屬性, 用於路由用途如聊天室 ID）.

若要做成雙向 channel（像聊天橋接器）, 需在 capabilities 中加入 `tools: {}` 並註冊一個 `reply` 工具, 讓 Claude 可以呼叫它把訊息送回外部平台, 同時要在 `instructions` 中告訴 Claude 何時、如何呼叫這個工具.

任何未經驗證的 channel 都是提示注入的破口, 因此必須在推送任何通知前先核對寄件者身分（依寄件者 ID 而非聊天室 ID 過濾, 否則群組聊天中任何人都能冒充）.

進階功能是權限中繼, 讓雙向 channel 可以把終端機的權限核准對話框同時轉發到你的聊天平台上, 你可以在終端機或手機上任一處回覆, 先到的回覆會被採用.這需要額外宣告 `claude/channel/permission` capability, 處理 Claude Code 送來的 `permission_request` 通知（內含五個小寫字母組成的 `request_id`、`tool_name`、`description`、`input_preview`）, 並在收到符合格式的回覆（如 `yes <id>` 或 `no <id>`）時送出 `notifications/claude/channel/permission` 裁定通知, 而非把它當作一般聊天內容轉發.只有在 channel 已確實驗證寄件者身分的情況下才應該啟用此功能, 因為任何能透過該 channel 回覆的人都能核准或拒絕你 session 中的工具使用.

研究預覽階段中, 自訂 channel 必須使用 `--dangerously-load-development-channels` 旗標繞過官方核准清單進行本機測試.完成後可包裝成 plugin 發布到 marketplace 供他人以 `/plugin install` 安裝.

## 7. 平行執行多個代理

subagents、agent view、agent teams、dynamic workflows 是四種不同的平行處理方式, 差異在於你是否要親自留在每個對話中、是否要交辦後稍後查看、或是要 Claude 幫你協調一組工作者.

| 方式 | 提供的能力 | 適用情境 |
| :-- | :-- | :-- |
| Subagents | session 內的委派工作者, 在自己的情境中處理支線任務並回傳摘要 | 支線任務會用大量搜尋結果、日誌、檔案內容淹沒主對話, 而這些內容之後不會再用到 |
| Agent view（研究預覽） | 一個畫面用來派送與監看背景執行的 session, 以 `claude agents` 開啟 | 有多個獨立任務要交辦、想一眼看到狀態、只在需要時才介入 |
| Agent teams（實驗性, 預設停用） | 多個協同運作的 session, 共用任務清單並互相傳遞訊息, 由一個 lead 管理 | 想讓 Claude 把專案拆成幾份、分派下去、並讓工作者彼此保持同步 |
| Dynamic workflows | 由腳本執行大量 subagents 並交叉驗證彼此結果的機制 | 工作規模超出單一回合能協調的範圍, 或需要多方交叉驗證結果 |

每種方式中的工作者都是 Claude session, 若要串接不同的工具, 應以 MCP server 的形式讓 Claude 存取.

worktrees 讓每個 session 各自有獨立的 git 工作目錄, 避免平行 session 互相覆寫檔案, agent view 會自動把每個派送的 session 移入獨立的 worktree.`/batch` 是一個 skill, 會把一個大型變更拆成 5 到 30 個以 worktree 隔離的 subagents, 各自開出 pull request.

選擇方式時可考量, 誰來協調工作（單一對話內由 Claude 委派用 subagents, 你交辦後稍後查看用 agent view, Claude 規劃、指派並監督一組工作者用 agent teams, 由腳本主導計畫而非 Claude 逐回合判斷用 dynamic workflows）, 工作者是否需要彼此溝通, 以及工作是否會碰到相同檔案（需用 worktrees 隔離, agent teams 不會自動隔離工作者, 需自行分割各自負責的檔案範圍）.

檢視進行中工作的方式, 背景 session 用 `claude agents` 開啟 agent view, 目前 session 中的 subagents 用 `/agents`, 目前 session 背景中的任何項目用 `/tasks`, dynamic workflows 用 `/workflows`.

## 8. 用 agent view 管理多個代理

agent view 以 `claude agents` 開啟, 是所有背景 session 的單一畫面, 顯示每個 session 正在做什麼、哪些需要你輸入.可以派送新 session、一眼掌握狀態, 只在需要時才介入.每個背景 session 都是完整的 Claude Code 對話, 在沒有連接終端機的情況下持續運作, 你可以隨時開啟、回覆、離開.目前為研究預覽階段, 需要 Claude Code v2.1.139 以上版本.

### 快速入門

依序為, 開啟 agent view（執行 `claude agents`, 隨時可按 Esc 回到 shell）, 派送 session（輸入描述任務的提示詞並按 Enter, 每個提示詞都會開啟一個新的背景 session）, 窺看與回覆（用方向鍵選取列並按空白鍵開啟窺看面板, 顯示最新輸出或正在等待的問題, 輸入回覆並按 Enter 送出）, 附加與分離（按 Enter 或右方向鍵附加進入完整對話, 在空白提示列按左方向鍵分離回到列表）, 把既有 session 帶進來（在 session 內執行 `/bg`, 或在空白提示列按左方向鍵把目前 session 背景化並開啟 agent view）.

### 監看 session

每列開頭的圖示顏色與動畫顯示狀態, Working（動畫）、Needs input（黃色, 正在等待你的具體問題或權限決定）、Idle（淡化）、Completed（綠色）、Failed（紅色）、Stopped（灰色）.圖示形狀則另外顯示底層行程是否存活.

每列的一行摘要由 Haiku 等級的模型產生, 讓你不用開啟完整對話紀錄就知道這個 session 在做什麼、需要什麼、或產出了什麼.當 session 開啟一個 pull request 時, 列的右側會出現 `PR #1234` 標籤, 依 PR 狀態（等待檢查或審查、檢查通過、已合併、草稿或已關閉）著色.

窺看面板（按 Space）顯示 session 需要什麼、最新輸出、以及開啟的 pull request, 大多數時候不需要再開啟完整對話紀錄.附加（按 Enter 或右方向鍵）會取代 agent view 進入完整互動 session, 附加時 Claude 會先簡短說明你不在的這段時間發生了什麼.分離不會停止背景 session, 按左方向鍵、Ctrl+Z、`/exit`、連按兩次 Ctrl+C 或 Ctrl+D 都能離開而不影響其運作, 要真正結束 session 需在其中執行 `/stop`.

整理列表, 可按 Ctrl+S 切換依目錄分組, 按 Ctrl+T 釘選 session 於頂端並保持行程運作, 按 Shift+方向鍵重新排序, 按 Ctrl+R 重新命名, 按 Ctrl+X 停止 session（兩秒內再按一次即刪除）.

### 派送新代理

可從 agent view 輸入框、既有 session 內執行 `/background`（別名 `/bg`）、或直接從 shell 用 `claude --bg "<prompt>"` 派送.派送時可用 `<agent-name> <prompt>` 或 `@<agent-name>` 指定以某個自訂 subagent 作為主要代理執行, 用 `@<repo>` 指定執行的儲存庫, 用 `! <command>` 以背景 shell 工作而非 Claude session 執行.

每個背景 session 在編輯檔案前會先移入獨立的 git worktree（位於 `.claude/worktrees/` 下）, 讓平行的 session 各自寫入自己的副本而不互相干擾.刪除 session 時會一併移除 Claude 建立的 worktree, 包含其中尚未提交的變更, 因此刪除前應先推送或提交想保留的工作.

### 從 shell 管理 session

| 指令 | 用途 |
| :-- | :-- |
| `claude agents` | 開啟 agent view |
| `claude agents --json` | 以 JSON 陣列列印目前所有 session |
| `claude attach <id>` | 在此終端機附加至某個 session |
| `claude logs <id>` | 印出該 session 最近的輸出 |
| `claude stop <id>` | 停止 session（也接受 `claude kill`） |
| `claude respawn <id>` | 保留對話內容重新啟動 session |
| `claude rm <id>` | 從列表中移除 session |

### 運作方式與限制

背景 session 由每位使用者各自的一個 supervisor 行程負責統籌, 與終端機及 agent view 分離, 會在你第一次背景化 session 或開啟 agent view 時自動啟動.session 結束且沒有附加超過約一小時後, supervisor 會停止其行程以釋放資源, 已釘選的 session 則不受此限制.

研究預覽階段的限制包含, 背景 session 會依同樣速率消耗你的訂閱用量, 平行執行十個代理大約會以十倍速度消耗額度, session 皆在本機執行, 睡眠時會被保留但關機會停止, Claude 建立的 worktree 會隨 session 刪除一併移除.

## 9. 協調 Claude Code session 團隊 agent teams

agent teams 讓你協調多個一起運作的 Claude Code 實例, 一個 session 擔任 team lead, 負責協調工作、指派任務、彙整結果, 其餘 teammates 各自在獨立的情境視窗中工作並直接互相溝通.與 subagents 不同的是, 你也可以不透過 lead 直接與個別 teammate 互動.

目前為實驗性功能且預設停用, 需在 `settings.json` 中加入 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` 環境變數才會啟用, 未設定時不會在 session 啟動時建立任何團隊, Claude 也不會主動提議或派送 teammate.

### 何時使用

最適合的情境是, 研究與檢視（多個 teammate 同時從不同角度調查, 再互相分享與挑戰彼此的發現）, 新模組或新功能（各自獨立負責一塊, 互不干擾）, 帶有競爭假說的除錯（teammate 平行測試不同理論, 更快收斂到答案）, 跨層級協調（前端、後端、測試各由不同 teammate 負責）.agent teams 會增加協調開銷並消耗遠多於單一 session 的 token, 對於循序性任務、同檔案編輯、或依賴關係眾多的工作, 單一 session 或 subagents 更有效率.

### 與 subagents 比較

| | Subagents | Agent teams |
| :-- | :-- | :-- |
| 情境 | 自己的情境視窗, 結果回傳給呼叫者 | 自己的情境視窗, 完全獨立 |
| 溝通 | 只把結果回報給主代理 | teammate 之間直接互傳訊息 |
| 協調 | 由主代理管理所有工作 | 共用任務清單並自行協調 |
| 最適合 | 只在意結果的專注型任務 | 需要討論與協作的複雜工作 |
| Token 成本 | 較低（結果會摘要回主情境） | 較高（每個 teammate 都是獨立的 Claude 實例） |

### 啟用與使用

啟用後, 用自然語言描述任務與想要的 teammate 角色, Claude 會派送並依你的提示協調工作.例如可以要求 "Spawn three teammates to explore this from different angles, 一個負責 UX, 一個負責技術架構, 一個扮演唱反調的角色".

顯示模式分為 in-process（所有 teammate 都在你的主終端機內, 用上下方向鍵在代理面板中選取, 按 Enter 檢視並輸入訊息直接對話）與 split panes（每個 teammate 各自有獨立面板, 需要 tmux 或 iTerm2）.預設為 in-process.

可指定 teammate 數量與模型, teammate 預設不會繼承 lead 的 `/model` 選擇, 需在 `/config` 中設定 Default teammate model.對於複雜或高風險任務, 可要求 teammate 在動手實作前先規劃, lead 審核計畫後核准或退回修正意見.

共用任務清單協調全隊工作, 任務有 pending、in progress、completed 三種狀態並可設定相依關係, lead 可明確指派任務, teammate 完成手上工作後也可自行認領下一個未被指派且未被阻塞的任務.

結束某個 teammate 時, 直接以名稱要求 "Ask the researcher teammate to shut down", lead 會送出結束請求, teammate 可核准或附上理由拒絕.

可用 hooks 強制品質關卡, `TeammateIdle` 在 teammate 即將閒置時觸發, `TaskCreated` 在建立任務時觸發, `TaskCompleted` 在標記任務完成時觸發, 皆可用結束碼 2 阻擋並附上回饋意見.

### 架構與限制

團隊由 team lead（主 session）、teammates（獨立的 Claude Code 實例）、共用任務清單、以及訊息信箱四個部分組成, 團隊設定與任務清單以 session 衍生的名稱儲存在本機的 `~/.claude/teams/` 與 `~/.claude/tasks/` 下.

teammate 啟動時會載入同樣的專案情境（CLAUDE.md、MCP servers、skills）以及 lead 給的派送提示詞, 但不會繼承 lead 的對話歷史.teammate 一開始會沿用 lead 的權限設定, 之後可個別調整模式, 但無法在派送當下就個別設定.

實驗性功能目前的限制包含, in-process teammate 不支援 session 回復（`/resume` 與 `/rewind` 無法還原）, 任務狀態有時會延遲更新, teammate 無法自行派送下一層的 teammate, 一個 session 只能有一個團隊, lead 身分固定無法轉移, split panes 模式不支援 VS Code 內建終端機、Windows Terminal、Ghostty.

實務建議是, 給 teammate 足夠的任務情境（因為不會繼承對話歷史）, 團隊規模從 3 到 5 個 teammate 開始, 每個 teammate 分配 5 到 6 個任務量最能維持生產力, 任務大小要拿捏得宜（太小協調開銷超過效益, 太大則工作太久沒有查核點）, 避免兩個 teammate 編輯同一份檔案, 並持續監看與適時介入導正方向.

## 10. 以程式化方式執行 Claude Code headless

Agent SDK 提供與 Claude Code 相同的工具、代理迴圈與情境管理能力, 可作為 CLI 用在腳本與 CI/CD, 或作為 Python 與 TypeScript 套件做完整的程式化控制.要以非互動模式執行, 加上 `-p` 旗標並附上提示詞即可, 例如, `claude -p "Find and fix the bug in auth.py" --allowedTools "Read,Edit,Bash"`.

加上 `--bare` 可跳過自動探索 hooks、skills、plugins、MCP servers、auto memory、CLAUDE.md, 藉此縮短啟動時間, 適合需要在任何機器上都得到相同結果的 CI 與腳本情境, 只有明確傳入的旗標才會生效.`--bare` 模式下 Claude 仍可使用 Bash 以及檔案讀寫工具, 若需要額外情境可用對應旗標補上, 例如系統提示詞用 `--append-system-prompt`, 設定用 `--settings`, MCP servers 用 `--mcp-config`.

若 `claude -p` 執行期間啟動了背景 Bash 任務（例如開發伺服器）, 該行程會在 Claude 回傳最終結果且 stdin 關閉後大約五秒被終止, 背景 subagents 與 workflows 則不受此五秒寬限期限制, `claude -p` 會等待其完成（預設上限十分鐘, 可用環境變數調整）.

常見用法示例, 把資料透過管線送進 Claude 並導出結果, `cat build-error.txt | claude -p '...' > output.txt`, 將 Claude 加進建置腳本作為專案專屬的 linter 或 reviewer.

用 `--output-format` 控制回應格式, `text`（預設, 純文字）、`json`（結構化 JSON, 含 session ID 與中繼資料）、`stream-json`（換行分隔的 JSON, 適合即時串流）.搭配 `--json-schema` 可要求輸出符合指定的 JSON Schema.搭配 `--output-format stream-json --verbose --include-partial-messages` 可即時接收逐字元的串流輸出.

用 `--allowedTools` 讓 Claude 在不詢問的情況下使用特定工具, 例如 `--allowedTools "Bash,Read,Edit"`.也可用 `--permission-mode` 設定整個 session 的權限基準, `dontAsk` 會拒絕不在允許規則中的任何操作, 適合鎖死的 CI 執行環境, `acceptEdits` 讓 Claude 可以直接寫入檔案並自動核准常見的檔案系統指令.

用 `--continue` 延續最近的對話, 或用 `--resume <session_id>` 延續特定對話, 適合分段執行多個後續提示詞的情境.

## 11. 用 ultraplan 在雲端規劃

ultraplan 把規劃工作從你本機的 CLI 交給一個在 Claude Code on the web 上以 plan mode 執行的雲端 session, Claude 在雲端起草計畫的同時, 你可以繼續在終端機做其他事.計畫準備好後, 你在瀏覽器中開啟, 針對特定段落留言、要求修改、並選擇要在哪裡執行.目前為研究預覽階段, 需要 Claude Code v2.1.91 以上版本, 需要 Claude Code on the web 帳號與 GitHub 儲存庫, 在 Bedrock、Vertex AI、Microsoft Foundry 上不可用.

從本機 CLI 啟動 ultraplan 有三種方式, 執行 `/ultraplan` 指令並附上提示詞, 在一般提示詞中的任何位置包含 ultraplan 這個關鍵字, 或是在本機 plan mode 完成計畫後的核准對話框中選擇以 Ultraplan on Claude Code on the web 進一步修改.

啟動後, CLI 的提示輸入框會顯示狀態指示器, `◇ ultraplan` 表示 Claude 正在研究程式碼並起草計畫, `◇ ultraplan needs your input` 表示 Claude 有澄清性問題待回答, `◆ ultraplan ready` 表示計畫已可供檢視.執行 `/tasks` 並選取 ultraplan 項目可開啟詳細檢視, 內含 session 連結、代理活動、以及停止 ultraplan 的選項.

計畫就緒後在瀏覽器中檢視, 支援, 行內留言（反白任一段落留言請 Claude 處理）、表情符號反應（標示認同或疑慮）、大綱側邊欄（在各段落間跳轉）.要求 Claude 處理留言後會提出修改後的版本, 可反覆迭代直到滿意為止.

選擇執行位置時, 在瀏覽器選擇核准計畫並在網頁開始撰寫, 由同一個雲端 session 實作, 完成後在網頁介面檢視差異並建立 pull request, 或選擇核准計畫並傳回終端機, 讓計畫回到你本機的終端機以完整的本機環境權限實作, 此時終端機會顯示一個對話框, 提供在此實作（將計畫注入目前對話並延續）、開新 session（清空目前對話, 只以該計畫作為情境重新開始）、取消（把計畫存成檔案但不執行, 並印出檔案路徑）三個選項.
