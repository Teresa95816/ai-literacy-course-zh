# 第六階段, 程式碼品質, 安全與合規

## 1. Checkpointing

Claude Code 會自動追蹤 Claude 所做的檔案編輯, 讓你可以快速復原變更並回到先前的狀態.

### checkpoint 如何運作

在與 Claude 協作時, checkpointing 會在每次編輯前自動擷取程式碼狀態, 這個安全網讓你可以放心進行大規模的變更, 因為隨時能回到先前的程式碼狀態.每個使用者提示詞都會建立一個新的 checkpoint, checkpoint 會跨 session 保留, 並在 30 天後隨 session 一起自動清除（可設定）.

### rewind 與 summarize

執行 `/rewind`, 或在提示輸入框空白時連按兩次 `Esc`, 可開啟 rewind 選單.選單會列出這個 session 中送出的每個提示詞, 選取想操作的時間點後可選擇以下動作,還原程式碼與對話（同時回復兩者）,還原對話（回到該訊息但保留目前程式碼）,還原程式碼（回復檔案變更但保留對話）,從此處摘要（把這個時間點之後的對話壓縮成摘要, 釋放情境視窗空間）,摘要至此處（把這個時間點之前的對話壓縮成摘要, 保留之後的訊息）.

還原選項會回復狀態（復原程式碼變更、對話歷史、或兩者）, 摘要選項則是把部分對話壓縮成 AI 生成的摘要而不改動磁碟上的檔案.兩種摘要方式中, 原始訊息都會保留在 session 逐字稿中, 因此 Claude 仍可在需要時參考細節.這與 `/compact` 類似, 但更為精準, 你可以選擇要壓縮所選訊息的哪一側.

若要保留原始 session 完整不動, 另闢分支嘗試不同做法, 應改用 fork（`claude --continue --fork-session`）.

### 常見應用情境

探索不同做法而不失去起點, 從造成錯誤的變更中快速復原, 在功能上反覆迭代並知道隨時可回到可運作的狀態, 從除錯過程的中間點開始摘要以釋放情境空間並保留最初的指示.

### 限制

checkpointing 不會追蹤由 bash 指令修改的檔案, 例如 `rm file.txt`、`mv old.txt new.txt` 這類操作無法透過 rewind 復原, 只有透過 Claude 檔案編輯工具直接進行的編輯才會被追蹤.checkpointing 也只追蹤目前 session 中已編輯過的檔案, 你在 Claude Code 之外手動修改的檔案、或其他並行 session 的編輯通常不會被捕捉.checkpoint 是為快速的 session 層級復原而設計, 不能取代版本控制, 應繼續用 Git 做永久的版本歷史與協作, checkpoint 是 "本機復原", Git 才是 "永久歷史".

## 2. Code Review

Code Review 會分析你的 GitHub pull request, 並在發現問題的程式碼行上以行內留言方式回報.一組專門的代理會在你完整程式碼庫的情境中檢視變更, 尋找邏輯錯誤、安全漏洞、邊界情況錯誤、以及細微的迴歸問題.目前為研究預覽階段, 適用於 Team 與 Enterprise 訂閱方案, 在啟用 Zero Data Retention 的組織中不可用.

發現的問題會依嚴重程度標記, 不會核准或封鎖你的 PR, 因此既有的審核流程不受影響.可以透過在儲存庫加入 `CLAUDE.md` 或 `REVIEW.md` 檔案來調整 Claude 標記的內容.

### 運作方式

Owner 為組織啟用 Code Review 後, 檢視會依儲存庫設定的行為在 PR 開啟時、每次推送時、或手動要求時觸發, 留言 `@claude review` 可在任何模式下對 PR 啟動檢視.檢視執行時, 多個代理在 Anthropic 基礎架構上平行分析差異與周邊程式碼, 每個代理尋找不同類別的問題, 接著一個驗證步驟會對照實際程式碼行為核對候選項目以過濾誤判.結果會去除重複、依嚴重程度排序, 並以行內留言張貼在發現問題的具體行數上, 審核內容開頭附上摘要.

### 嚴重程度標記

| 標記 | 嚴重程度 | 意義 |
| :-- | :-- | :-- |
| 紅色 | Important | 合併前應修復的錯誤 |
| 黃色 | Nit | 值得修但不阻擋合併的小問題 |
| 紫色 | Pre-existing | 程式碼庫中既有但非此 PR 引入的錯誤 |

每則留言在 GitHub 介面上已附上 好 與 不好 兩個反應按鈕, 供一鍵評分, Anthropic 會在 PR 合併後收集反應次數用以調校審核模型, 但反應本身不會觸發重新檢視.回覆行內留言不會促使 Claude 回應或更新 PR, 要處理發現的問題需修好程式碼並推送, 若要在不推送的情況下要求全新檢視, 可留言 `@claude review once`.

### 設定 Code Review

Owner 為整個組織啟用一次並選擇要納入的儲存庫, 步驟為, 在管理後台開啟 Code Review 區塊, 點選 Setup 開始安裝 GitHub App 流程, 安裝 Claude GitHub App（需要 Contents 讀寫、Issues 讀寫、Pull requests 讀寫權限）, 選擇要啟用的儲存庫, 為每個儲存庫設定審核觸發時機（PR 建立後一次、每次推送後、或僅手動）.

### 手動觸發檢視

`@claude review` 啟動檢視並讓 PR 訂閱後續推送觸發的檢視, `@claude review once` 只啟動單次檢視而不訂閱.兩者都必須以頂層 PR 留言方式張貼, 且需要儲存庫的擁有者、成員或協作者權限.

### 客製化檢視

Code Review 會讀取 `CLAUDE.md`, 把新引入的違規視為 nit 等級的發現.`REVIEW.md` 則是專屬於檢視行為的檔案, 內容會以最高優先權注入審核流程中每個代理的系統提示詞, 可調整的重點包含, 重新定義嚴重程度分級標準, 限制單次檢視張貼的 nit 留言數量上限, 列出應完全略過的路徑或分支模式（如產生的程式碼、鎖定檔案）, 加入儲存庫專屬的檢查規則, 要求某類發現需附上證據才能張貼, 定義重新檢視時的收斂行為, 以及要求摘要開頭附上一行式的統計數字.

### 計費

Code Review 依 token 用量計費, 每次檢視平均費用 15 到 25 美元, 依 PR 大小、程式碼庫複雜度、以及需要驗證的問題數量而變動, 透過用量額度另外計費, 不計入方案內含用量.

### 疑難排解

檢視執行是盡力而為的, 失敗的執行不會阻擋 PR, 但也不會自動重試.若檢視遭遇內部錯誤或超過時間限制, 可留言 `@claude review once` 重新執行.達到組織每月支出上限時, Code Review 會在 PR 上留言說明檢視被跳過, 下個計費週期會自動恢復.

### 在本機檢視差異

`/code-review` 指令可在終端機中檢視差異而不需安裝 GitHub App, 預設涵蓋分支相對於上游的 commit 加上工作目錄中未提交的變更.加上 `--comment` 可把發現張貼為行內 PR 留言, 加上 `--fix` 則在檢視後把發現套用到工作目錄.`/code-review ultra --fix` 會在雲端執行更深入的 ultrareview, 並在結果回傳 session 後套用到工作目錄.

## 3. 用 ultrareview 找出錯誤

ultrareview 是在 Claude Code on the web 基礎架構上執行的深度程式碼檢視.執行 `/code-review ultra` 時, Claude Code 會在遠端沙盒中啟動一組審核代理, 尋找你分支或 pull request 中的錯誤.目前為研究預覽階段, 需要 Claude Code v2.1.86 以上版本, 指令現稱為 `/code-review ultra`, `/ultrareview` 仍保留為別名.

相較於本機的 `/code-review` 或 `/review`, ultrareview 提供, 更高的訊噪比（每個回報的發現都會獨立重現並驗證, 結果聚焦真正的錯誤而非風格建議）, 更廣的涵蓋範圍（更大規模的審核代理群平行探索變更）, 不佔用本機資源（審核完全在遠端沙盒執行）.需要以 Claude.ai 帳號驗證, 在 Bedrock、Vertex AI、Microsoft Foundry 上不可用, 對已啟用 Zero Data Retention 的組織也不可用.

### 從 CLI 執行

```text
/code-review ultra
```

不帶參數時, ultrareview 檢視目前分支與預設分支之間的差異, 包含工作目錄中未提交與已暫存的變更.傳入 PR 編號則改為檢視該 pull request, 此模式下遠端沙盒會直接從主機 clone PR, 而非打包本機工作目錄.

### 計費與免費次數

| 方案 | 內含免費次數 | 超出後 |
| :-- | :-- | :-- |
| Pro | 3 次免費 | 以用量額度計費 |
| Max | 3 次免費 | 以用量額度計費 |
| Team 與 Enterprise | 無 | 以用量額度計費 |

免費次數是每個帳號一次性配額, 不會重新整理.用完後, 每次檢視通常費用為 5 到 20 美元, 依變更規模而定.

### 追蹤執行中的檢視

一次檢視通常需要 5 到 10 分鐘, 以背景任務方式執行, 可用 `/tasks` 查看執行中與已完成的檢視、開啟詳細檢視、或停止進行中的檢視.完成後, 已驗證的發現會以通知方式出現在 session 中.

### 非互動執行

`claude ultrareview` 子指令可在 CI 或腳本中啟動 ultrareview, 會阻塞直到遠端檢視完成, 把發現印到 stdout, 成功時結束碼為 0, 失敗為 1.

### 與 /code-review、/review 比較

| | /code-review | /review \<pr\> | /code-review ultra |
| :-- | :-- | :-- | :-- |
| 目標 | 你的工作差異 | 一個 GitHub pull request | 你的工作差異或一個 pull request |
| 執行位置 | session 內本機執行 | session 內本機執行 | 遠端雲端沙盒 |
| 深度 | 依 effort 參數調整 | 中等的 /code-review 引擎 | 多代理群並獨立驗證 |
| 耗時 | 數秒到數分鐘 | 數分鐘 | 約 5 到 10 分鐘 |
| 適合 | 邊做邊要快速回饋 | 核准前檢視同事的 PR | 合併重大變更前的深度把關 |

## 4. 安全性

Claude Code 建置時將安全性視為核心, 依循 Anthropic 完整的安全性計畫開發.

### 以權限為基礎的架構

Claude Code 預設採用嚴格的唯讀權限, 需要額外動作（編輯檔案、執行測試、執行指令）時會明確請求權限.內建的唯讀指令集（如 `ls`、`cat`、`git status`）不需提示即可執行.

### 內建保護機制

沙盒化的 bash 工具（隔離檔案系統與網路, 用 `/sandbox` 定義可自主運作的邊界）, 寫入權限限制（Claude Code 只能寫入啟動時所在的資料夾及其子資料夾, 無法在未經明確許可下修改上層目錄）, 提示疲勞緩解（支援針對使用者、程式碼庫、或組織允許常用的安全指令）, Accept Edits 模式（自動核准檔案編輯及固定的一組檔案系統 Bash 指令, 如 `mkdir`、`touch`、`rm`、`mv`、`cp`、`sed`）.

### 防範提示注入

提示注入是攻擊者試圖透過插入惡意文字來覆寫或操控 AI 助理指示的技術, Claude Code 具備數項防護, 權限系統（敏感操作需要明確核准）, 情境感知分析（分析完整請求以偵測潛在有害指示）, 輸入淨化, 網路指令核准（`curl`、`wget` 這類從網路取得內容的指令預設不會自動核准）.

其他防護包含, 隔離的情境視窗（WebFetch 使用獨立的情境視窗以避免注入潛在惡意提示）, 信任驗證（首次執行程式碼庫與新的 MCP server 需要信任驗證）, 指令注入偵測（可疑的 bash 指令即使先前已加入允許清單, 仍需手動核准）, 安全的憑證儲存（API 金鑰與 token 在 macOS 上存於 Keychain, 在 Windows 與 Linux 上以檔案權限保護）.

### 雲端執行安全性

使用 Claude Code on the web 時, 額外的安全控制包含, 隔離的虛擬機器（每個雲端 session 在獨立的、Anthropic 管理的 VM 中執行）, 網路存取控制, 憑證保護（驗證透過安全代理處理, 使用沙盒內範圍受限的憑證）, 分支限制（git push 操作限於目前工作分支）, 稽核日誌, 自動清理.

### 安全性最佳實務

處理敏感程式碼時, 應核准前檢視所有建議的變更, 對敏感儲存庫使用專案專屬的權限設定, 考慮使用 dev containers 做額外隔離, 定期用 `/permissions` 稽核權限設定.團隊安全性方面, 應用管理設定強制執行組織標準, 透過版本控制分享已核准的權限設定, 訓練團隊成員安全最佳實務, 透過 OpenTelemetry 監控使用情況.

## 5. 在 Claude 撰寫程式碼時抓出安全問題 security-guidance 外掛

security guidance 外掛讓 Claude 在工作過程中審視自己所做的程式碼變更是否有常見漏洞, 並在同一個 session 中修復發現的問題, 涵蓋的問題類型包含注入攻擊、不安全的反序列化、以及不安全的 DOM API, 在程式碼進入 pull request 之前就先處理, 減少落到人工審核者身上的安全檢視負擔.安裝後外掛會自動運作, 不需要另外呼叫任何指令.

這個外掛是 Code Review（在 pull request 上執行）的 session 內搭檔, 外掛減少送進 PR 的問題量, Code Review 則抓住漏網之魚.

### 前置需求

Claude Code CLI v2.1.144 以上版本, `PATH` 中需有 Python 3.8 以上版本, 以及一個 git 儲存庫（回合結束與 commit 層級的檢視需要對照 git 狀態, 在非儲存庫目錄中會靜默略過, 逐一編輯的模式檢查則到處都能運作）.

### 安裝

```text
/plugin install security-guidance@claude-plugins-official
```

安裝時選擇使用者範圍, 讓外掛在此機器上每個新的本機 session 中都會載入, 接著執行 `/reload-plugins` 套用變更.若要在雲端 session 與共享儲存庫中啟用, 需在專案已提交的 `.claude/settings.json` 中宣告.

### 三層檢查

每次檔案編輯（快速的模式比對, 尋找危險呼叫, 不呼叫模型, 不計成本）, 每回合結束（背景執行的模型審核, 檢視該回合改動的一切, 抓出授權繞過、不安全的直接物件參照、注入、伺服器端請求偽造、弱加密等問題）, 每次 Claude 執行的 commit 或 push（更深入的代理式審核, 會閱讀周邊程式碼以判斷發現是否真實）.

### 新增自訂規則

建立 `.claude/claude-security-guidance.md` 描述你的威脅模型與審核檢查清單, 供模型審核層載入為額外情境.建立 `.claude/security-patterns.yaml` 可為逐一編輯的模式檢查加入正規表達式或子字串規則.

### 計費與停用

逐一編輯的模式檢查不呼叫模型, 不計成本, 回合結束與 commit 層級的審核則各自消耗額外的模型用量.可個別停用各層（`ENABLE_PATTERN_RULES=0`、`ENABLE_STOP_REVIEW=0`、`ENABLE_COMMIT_REVIEW=0`）, 或以 `SECURITY_GUIDANCE_DISABLE=1` 完全停用外掛.

## 6. 資料使用

### 資料訓練政策

消費者使用者（Free、Pro、Max 方案）可選擇是否允許資料用於改進未來的 Claude 模型.商業使用者（Team、Enterprise 方案、API、第三方平台、Claude Gov）維持既有政策, Anthropic 不會用透過商業條款傳送給 Claude Code 的程式碼或提示詞訓練生成式模型, 除非客戶已選擇提供資料供模型改進（例如 Development Partner Program）.

### /feedback 指令

若選擇用 `/feedback` 指令傳送意見回饋, 可能會用來改進產品與服務, 透過 `/feedback` 分享的逐字稿會保留 5 年.

### session 品質調查

"Claude 這次 session 表現如何?" 的提示只會記錄你的評分, 不會收集或儲存任何對話逐字稿.評分提示之後可能出現另一個獨立的後續問題, 詢問是否可以查看你的 session 逐字稿以協助改進 Claude Code, 除非你明確選擇是, 否則不會上傳任何內容.

### 資料保留

消費者使用者, 允許資料用於模型改進者保留 5 年, 不允許者保留 30 天.商業使用者標準保留期為 30 天, 符合資格的帳號可申請 Zero Data Retention.本機快取方面, Claude Code 客戶端會把 session 逐字稿以明文形式存於 `~/.claude/projects/` 下, 預設保留 30 天, 可用 `cleanupPeriodDays` 調整.

### 遙測服務

Claude Code 會連線至 Anthropic 記錄延遲、可靠性、使用模式等操作性指標, 不包含任何程式碼或檔案路徑, 設定 `DISABLE_TELEMETRY` 可退出遙測.也會連線至 Sentry 做操作性錯誤記錄, 設定 `DISABLE_ERROR_REPORTING` 可退出.

### WebFetch 網域安全檢查

WebFetch 工具在擷取 URL 前, 會把要求的主機名稱送到 `api.anthropic.com` 核對 Anthropic 維護的安全封鎖清單, 只傳送主機名稱, 不含完整 URL、路徑、或頁面內容, 結果依主機名稱快取 5 分鐘.此檢查不受任何模型供應商影響, 也不受 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` 影響.

## 7. 法律與合規

### 授權條款

使用 Claude Code 受商業條款（適用於 Team、Enterprise、Claude API 使用者）或消費者服務條款（適用於 Free、Pro、Max 使用者）規範.無論是直接使用 Claude API（1P）或透過 Amazon Bedrock 或 Google Vertex 存取（3P）, 既有的商業協議都適用於 Claude Code 的使用.

### 醫療合規 BAA

若客戶與 Anthropic 簽有 Business Associate Agreement, 且想使用 Claude Code, 只要該客戶已簽署 BAA 並啟用 Zero Data Retention, BAA 會自動延伸涵蓋 Claude Code, 適用於該客戶流經 Claude Code 的 API 流量.ZDR 是以每個組織為單位啟用的, 因此每個組織都必須各自啟用 ZDR 才能受 BAA 涵蓋.

### 使用政策

Claude Code 的使用受 Anthropic 使用政策規範.

### 驗證與憑證使用

Claude Code 使用 OAuth token 或 API 金鑰向 Anthropic 伺服器驗證身分.OAuth 驗證僅供 Claude Free、Pro、Max、Team、Enterprise 訂閱方案的購買者使用, 用於支援 Claude Code 與其他 Anthropic 原生應用程式的一般使用.開發者若在建置與 Claude 能力互動的產品或服務, 應改用透過 Claude Console 或受支援雲端供應商的 API 金鑰驗證, Anthropic 不允許第三方開發者代表其使用者透過 Free、Pro、Max 方案的憑證路由請求.

### 安全性與信任

安全漏洞回報透過 HackerOne 管理.
