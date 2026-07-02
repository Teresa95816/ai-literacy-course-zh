# 第十二階段, 其他資源

## 1. Escalate hard decisions with the advisor tool 用 advisor 工具升級困難決策

advisor 工具讓 Claude 在任務關鍵時刻（決定方案前, 卡在重複出現的錯誤時, 宣告任務完成前）諮詢第二個, 通常更強的模型.advisor 收到完整對話（含每次工具呼叫與結果）並回傳 Claude 會採納的建議, 是在 Anthropic 基礎架構上執行的伺服器端工具, 目前為實驗性功能, 僅支援 Anthropic API（不支援 Bedrock, Vertex AI, Foundry）, 需 Claude Code v2.1.98 以上.

可用三種方式設定, `/advisor` 指令（互動選擇並存為預設）, settings 檔的 `advisorModel` 欄位（持久預設）, `--advisor` 旗標（單次 session）.advisor 能力必須至少與主模型相當（如 Sonnet 主模型可配對 Fable, Opus, Sonnet advisor, Opus 主模型只能配對相同或更高版本的 Opus 或 Fable）.每次呼叫 advisor 會依 advisor 模型費率額外計費, 因 Claude 只在決策點呼叫而非每回合都呼叫, 通常比全程用更強模型划算.`/advisor off` 可停用, `CLAUDE_CODE_DISABLE_ADVISOR_TOOL=1` 可完全關閉此功能.

## 2. Champion kit 內部推廣者手冊

給已經在使用 Claude Code 並想協助團隊採用的工程師.採用一項開發工具很少是因為有人發布了推廣公告, 而是因為某人開始把工具用得好, 公開分享, 並讓其他人容易跟上.這份手冊定義三個互相強化的行為, 分享你的發現（把提示詞, 截圖, 小勝利貼到團隊已經在看的地方）, 成為大家會問的人（回答時附上實際用的提示詞而非解釋）, 擴大圈子（建立少量輕量, 可持續的習慣, 如專屬頻道或每週分享串）.

建議每週投入時間, 貼分享約 15 分鐘, 在共享頻道回答問題約 20 分鐘, 主持每週分享串約 5 分鐘, 選擇性結對或走查 0 到 30 分鐘.文件提供 30 天推廣時程表（第一週播種頻道, 第二週啟動節奏, 第三週結對並整理常見問答, 第四週交棒給下一位推廣者）, 以及對常見疑慮（"我不用它更快", "junior 工程師會變弱"）的回應建議, 核心原則是承認疑慮, 提供簡短的重新框架, 並在對方自己的程式碼上做一次具體示範.

## 3. Communications kit 組織溝通套件

給要在工程組織內推出 Claude Code 的管理者與工程主管, 提供可直接複製使用的發布公告, 逐週提示技巧宣傳文案, 以及常見問答的一行回覆.發布前檢查清單包含建立 `#claude-code` 頻道, 在自己環境測試過安裝指令, 準備好資料處理與安全性連結, 選定一個具體的第一個任務範例, 指定頻道前 48 小時的負責人, 安排高層贊助者具名發送公告（比管理者發送的採用率更高）.

提示技巧宣傳文案涵蓋模型選擇（Sonnet 為日常工作預設, Opus 用於大型重構, Haiku 用於快速機械式編輯）, `/init` 產生 CLAUDE.md, `@` 檔案引用, 權限模式（`Shift+Tab` 切換 default, acceptEdits, plan）, checkpointing 與 `/rewind`, MCP 連接器, skills, hooks, 螢幕截圖輸入, git 工作流程, plugins.每則文案遵循相同結構, 一個吸引點, 效益說明, 立即可試的動作, 文件連結.

## 4. Prompt library 提示詞庫

依開發生命週期階段與角色標記的可直接複製貼上提示詞集合.涵蓋探索階段（了解程式碼庫架構, 解釋不熟悉的程式碼, 尋找特定行為發生位置, 追溯程式碼演進歷史）, 到後續的規劃, 實作, 測試, 審查, 部署各階段, 部分提示詞額外標記適合 PM 或設計角色使用.每個提示詞條目包含可代換的占位符（如 `{path}`, `{behavior}`）與後續延伸閱讀連結.

這是一個可依任務類型與角色篩選的實用提示詞範例集合, 定位為降低撰寫有效提示詞門檻的參考起點, 而非要逐字背誦的公式, 適合貼到內部 wiki 或分享頻道供團隊成員直接取用.

## 5. Claude Code changelog 版本更新紀錄

記錄 Claude Code 每個版本的新功能, 改進, 錯誤修復, 版本號格式如 `2.1.198 (July 1, 2026)`, 來源為 GitHub 上的官方 CHANGELOG.md.近期重點更新包含, Claude in Chrome 正式推出, 新增 `Notification` hook（`agent_needs_input` / `agent_completed`）支援背景代理通知, 新增 `/dataviz` skill 協助圖表與儀表板設計, 背景代理完成工作時可自動 commit, push, 並開啟 draft PR, Explore 代理改為繼承主 session 模型（上限 Opus）, 串流連線在短暫網路中斷時會自動重試.再往前的版本帶來 Claude Sonnet 5 上線（新預設模型, 原生 1M token 情境視窗, 限時促銷價格）, 組織層級預設模型支援, 可點擊的檔案附件, MCP 伺服器安全性改進.

執行 `claude --version` 可檢查目前安裝版本, 完整更新紀錄應查閱官方文件索引, 因為 Claude Code 更新頻繁, 課程中列出的版本號僅供參考時間點, 實際版本應以官方 changelog 為準.
