--- 
marp: true
theme: default
backgroundColor: #1A1A1A
color: #F5F0E8
paginate: true
---

<style>
section {
  font-size: 24px;
}
table {
  border-collapse: collapse;
}
th {
  background-color: #D97757;
  color: white;
}
td {
  background-color: #1A1A1A;
}
blockquote {
  background-color: #1A1A1A;
  border-left: 8px solid #D97757;
  padding: 1.5rem;
  border-radius: 4px;
  font-style: italic;
  color: #F5E0E8;
}
</style>

# Pattern Examples

---

## 從零建立一套需要符合國際稽核標準的碳排系統

對話一（Spec-First + Context Engineering）

```text
我們公司的碳排報告需要符合 ISAE 3410 稽核標準。
現在的做法是截圖存證，但稽核員說截圖有人工編輯的風險，不夠可信。
我想建一個系統：從出貨部門提交的 Excel → 自動計算碳排
→ 產生可供稽核員查驗的介面。
關鍵是要能證明「數據從採集到計算全程沒有人工干預」。
這個方向可行嗎？
```

這段話同時做了兩件事：定義目標（Spec-First），也提供了真實背景（Context Engineering）。「符合 ISAE 3410」和「稽核員說截圖有風險」這兩個句子，讓 Claude Code 知道這不是普通的資料處理工具，它需要設計成能被第三方稽核的系統，這個背景直接決定了架構方向。

---

對話二（Atomic Tasking）

```text
先從產生測試資料開始。
我需要 120 筆假的出貨運單，但資料要刻意設計成「有問題的」：
- 日期格式混用四種（含美式、點分隔）
- 車種有六種不同寫法（含簡體、英文縮寫）
- 重量單位混用 kg 和噸
這樣才能測試後面的清洗邏輯夠不夠強。
先做這步，完成後再繼續。
```

整個系統分五個步驟，這裡只要求做第一步。而且連測試資料都事先設計好「要有哪些髒資料」。這是 Atomic Tasking 的延伸：不只是拆步驟，連每一步的驗收條件都想清楚了再交辦。

---

對話三（Scaffold-and-Fill）

```text
路線計算這段，我想同時支援兩種模式：
開發測試時用 mock（不需要網路，估算就好）；
正式展示時切換成 Google Routes API 取真實路網。
設計成用一個環境變數切換，兩種模式對外介面要完全一樣，
切換時不需要改任何其他程式。
把這個架構先定好，再實作細節。
```

先定介面契約，再實作兩個 provider。框架對齊後，填入內容才不會跑偏。

---

對話四（Review-and-Iterate）

```text
等一下，我剛才說的 Google TIM API 好像不對。
我查了一下，TIM 是 Travel Impact Model，只做航班碳排，不做公路運輸。
公路距離應該用 Google Routes API v2 對嗎？
請確認後調整計畫。
```

這是 Review-and-Iterate 最真實的樣子。不是在看 AI 的輸出，而是在看自己的假設。

需求說錯了，在執行前抓出來修正，比讓錯誤跑進程式碼再回頭改要省力得多。

---

## 台股損益追蹤工具

> 換一個角度：這次的主角不是任務本身，而是學習目標。

對話一（Spec-First + Context Engineering）

```text
我想做一個個人台股損益追蹤工具。
但不只是做出來能用就好，我想藉這個專案真正學懂 PostgreSQL，
包括 Foreign Key、View、Index 這些功能，實際上在解決什麼問題。
技術選型：PostgreSQL + Python + Streamlit。
請幫我規劃專案結構和資料庫 Schema，
並且說明每個設計決策背後的理由。
```

「不只是做出來能用就好」這句話改變了整個專案的輪廓。Claude Code 因此知道：README 不能只寫安裝步驟，每個用到的 SQL 功能都要對應說明它在這個專案裡解決了什麼問題。同樣的功能，學習目標不同，交付物就完全不同。

---

對話二（Scaffold-and-Fill）

```text
先把資料庫的 Schema 設計好，不要急著寫 Python 程式。
需要包含：
- stocks 和 transactions 的 Foreign Key 關聯
- CHECK constraint 限制 trade_type 只能是 BUY 或 SELL
- portfolio_summary 的 View，封裝損益計算邏輯
- daily_prices 的 Index，加速最新股價查詢
Schema 確認沒問題，再開始寫 Dashboard 和操作模組。
```

資料模型是地基，地基沒設計好，上面的程式遲早要打掉重來。先把 Schema 定好、驗證關聯邏輯正確，再開始寫應用層。這是 Scaffold-and-Fill 在資料庫設計上最直接的體現。

---

對話三（Atomic Tasking）

```text
db_ops.py 設計成共用模組，Dashboard 和 CLI 腳本都從這裡呼叫，
不要讓 Dashboard 直接寫 SQL，也不要讓 CLI 各自維護一份 DB 邏輯。
這樣改 DB 操作時只需要改一個地方。
先把這個模組的函式介面定好，再實作 Dashboard 的 UI。
```

一次只做一件事：先定函式簽名，確認介面設計對了，再填實作。同時也把架構約束說清楚。單一 DB 操作入口，這個決策在一開始說定，之後就不需要再討論。

---

對話四（Review-and-Iterate）

```text
更新 pandas 之後，Dashboard 的報酬率顏色標記壞掉了。
pandas 3.x 改了 Styler API，原本的寫法不相容。
請找出差異並修正，不要動其他部分。
```

環境升版導致既有功能損壞，這是開發過程裡的常態。

「不要動其他部分」讓修改範圍明確，避免為了修一個問題又引入新的變動。

---

## 主機板市場情報工具

> 從單一市場起步、逐步擴展成三地區自動化管線。

對話一（Atomic Tasking + Scaffold-and-Fill）

```text
做一個主機板市場分析工具，流程固定是三步：
抓資料 → 生成 HTML Dashboard → 寄信。
每一步各自獨立，輸出格式統一用 JSON。

先只做北美市場（Newegg）的版本，
三步都跑通、Dashboard 確認沒問題，再來擴展其他市場。
```

流程架構先定好，再實作第一個市場。因為骨架固定（scraper → JSON → dashboard → email），日本、韓國版本之後只需要填入各自的資料來源，不需要重新設計流程。

---

對話二（Context Engineering）

```text
Newegg 版本確認了，接下來擴展到日本（Kakaku）和韓國（Danawa）。
注意三個市場的核心指標不一樣：
- Newegg：有評分（1–5 星）和評論數
- Kakaku：只有評論數，沒有評分
- Danawa 是比價網站，用「上架店家數」衡量市場熱度，沒有用戶評分

每個市場的 Dashboard 要用各自適合的指標呈現，
不要把 Newegg 的評分格式硬套到沒有評分的市場。
```

三個市場表面上做的是一樣的事，但背後的數據結構不同。這段話讓 Claude Code 知道不能複製貼上 Newegg 的邏輯。Danawa 的欄位是 `store_count`，不是 `rating`。沒有這個背景，AI 很可能生出一份對 Danawa 套用評分格式的錯誤 Dashboard。

---

對話三（Review-and-Iterate）

```text
Danawa 的資料有問題 : 產品被歸到錯誤的品牌。
例如 ASUS 的主機板出現在 MSI 的資料裡。
請檢查 danawa_scraper.py 的品牌判斷邏輯，找出原因，
修正後重新抓一次，確認四個品牌的資料都正確。
```

Danawa 的頁面結構和其他市場不同，brand 欄位的解析需要各別處理。測試後發現問題、回頭修正 scraper 邏輯。

這是 Review-and-Iterate 在資料管線裡的標準樣貌。

---

對話四（Spec-First + Context Engineering）

```text
這個工具需要自動執行，不能靠手動觸發。
需求：
- 每週一早上九點（台灣時間）自動跑全部三個市場
- API 金鑰和收件人 email 不能寫進程式碼，用 GitHub Secrets 管理
- 三個市場可以寄給不同的收件人，每個市場要有獨立的收件人變數
- 任何一個市場失敗，要在 log 裡明確標出是哪個市場出錯

用 GitHub Actions 實作。
```

自動化的需求在動手之前一次說清楚：排程時間、秘密管理方式、收件人分離、失敗通知。這些細節任何一個沒說，都可能在第一次自動執行時才發現問題。

---

## 透過實作理解 MCP 協定

> 學習本身是明確目標。

對話一（Spec-First）

```text
我想透過實作來真正理解 MCP（Model Context Protocol）的運作方式。
目標：一個 CLI 聊天工具，AI 可以透過 MCP 使用工具和讀取文件。
功能：
- @文件名 把文件內容注入 prompt
- /指令 執行預設的提示模板
- Tab 鍵自動補全可用的文件和指令

在開始寫任何程式之前，請先說明建議的架構分層，
我確認方向後再動手。
```

學習目標說清楚，要求動手前先看架構。Claude Code 因此知道這不是「做一個 CLI 就好」，而是要做一個值得仔細閱讀的範例，每一層的設計決策都要能說得清楚。

---

對話二（Scaffold-and-Fill）

```text
架構分成這幾層，每層只做一件事：
- main.py：接線，啟動事件迴圈
- core/cli.py：輸入介面（自動補全、歷史記錄）
- core/chat.py：agentic loop 邏輯
- core/claude.py：跟 LLM 通話
- core/tools.py：執行工具

先把每一層的介面定好，再填實作邏輯。
不要讓任何一層碰到不屬於它的邏輯。
```

分層架構先定好再填內容。之後換 API 只需要動 `core/claude.py` 一個檔案，其他層完全不受影響。

框架設計的品質，決定了日後修改的代價。

---

對話三（Context Engineering）

```text
@文件名 的處理方式有兩個選擇：
A. 讓 AI 自己決定要不要去抓文件
B. 在送給 AI 之前，直接把文件內容注入 prompt

我選 B。
原因：讓使用者明確控制哪些資訊進入 context，
比讓 AI 猜測更可靠、更可預期。
請設計成 @mention 在進入 agentic loop 之前就被處理掉。
```

A 或 B 技術上都可行，但背後的理由改變了整個 UX 設計。說清楚「為什麼選 B」，Claude Code 才能設計出符合這個原則的架構，而不是做出邊界情況下行為完全不同的東西。

---

對話四（Review-and-Iterate）

```text
把 API 從 Anthropic 換成 OpenAI 之後，tool result 格式出問題了。
Anthropic 的 tool result 是包在一個 user message 裡的 list；
OpenAI 的每個 tool result 是獨立的 message，role 是 "tool"。
現在 AI 拿不到工具執行結果，一直在循環。
請修正 core/chat.py 和 core/tools.py，其他部分不要動。
```

格式差異不看文件看不出來，跑起來才會發現。問題定位清楚之後，「其他部分不要動」把修改範圍框住。這正是分層架構發揮作用的時刻。

---

## 企業 AI 展示網站

> 部署環境的限制主導技術選型。

對話一（Spec-First + Context Engineering）

```text
這個網站要部署在 GitHub Pages 上，
GitHub Pages 只能直接服務靜態檔案，沒有 CI/CD 執行 build 的能力。
技術選型：純 HTML + CSS + JS，不用任何框架。
目標：不需要 Node.js、不需要任何工具，直接開 index.html 就能執行。
請設計檔案結構，確認這個限制沒有問題後再開始。
```

部署環境的限制在動手前就說清楚。這讓 Claude Code 直接排除 React、Vite 這類需要 build 步驟的選項。不是因為它們不好，而是根本不符合部署條件。環境限制是一種常被忽略的 Context。

---

對話二（Scaffold-and-Fill）

```text
這個網站要支援中英雙語。
架構分三層：
- index.html：只放頁面結構，不放任何文字內容
- locales.js：所有中英文字內容統一放這裡
- main.js：語言切換邏輯和所有動態行為

任何內容的新增或修改，都只動 locales.js，
不要讓文字散落在 HTML 或 JS 裡。
```

內容、結構、行為三層分離，修改文字時只需要開一個檔案。這個規則在一開始說定，之後新增任何語系內容都不需要重新討論架構。

---

對話三（Context Engineering）

```text
YouTube Data API 的金鑰必須放在前端頁面裡，因為這是純靜態網站，
沒有辦法藏在伺服器端。
我擔心金鑰暴露的問題，請說明最穩健的處理方式。
```

說明約束條件（純靜態、無伺服器），讓 Claude Code 給出真正可行的方案。最終設計是兩層：在 Google Cloud Console 限制金鑰只能從指定網域使用；同時透過 GitHub Actions 在部署時注入金鑰，原始碼裡永遠只有佔位符。少了「純靜態、無伺服器」這個背景，AI 可能直接建議用環境變數，而那在這個部署環境裡根本行不通。

---

對話四（Review-and-Iterate）

```text
加了淺色主題切換按鈕之後，整體質感變差了。
這個品牌定位是深色科技感，淺色模式破壞了視覺一致性。
請把 light mode 的所有相關程式碼全部移除，恢復深色專一版本。
```

試了，不對，回退。這是 Review-and-Iterate 最乾淨的形式。不是 AI 做錯了，而是在看到成果之前無法確定方向，看到之後立刻做出判斷，並且清楚說明原因和範圍。

---

## 把 API 文件變成三種可學習的格式

> 使用者是在設計給別人學的東西。

對話一（Spec-First）

```text
我想做一個教人使用 Claude API 的學習套件。
學習者需要三種不同深度的接觸方式：
1. 一份可以印出來查閱的參考文件（Markdown）
2. 一個在瀏覽器裡可以互動操作的視覺化工具（HTML，不需要伺服器）
3. 可以直接跑的 Python 範例程式碼

三種格式要涵蓋相同的概念，但各自以最適合的方式呈現。
請先說明整體結構，再開始動手。
```

三種交付物在動手前就定義清楚。這讓 Claude Code 知道：同一個概念要被設計三次，每次適應不同的學習情境，而不是做完一個再問「還要加什麼」。

---

對話二（Atomic Tasking）

```text
HTML 視覺化工具，先只做第一個章節：API 請求生命週期。
動畫流程：client → server → Claude API → server → client，
每個步驟暫停，讓學習者看清楚再繼續。
做完這個讓我確認效果，再繼續做下一個章節。
```

視覺化工具有八個章節，每次只做一個，確認效果後再繼續。git 記錄了每個章節獨立 commit。

temperature slider、multi-turn conversation、streaming、tool use……一個一個疊上去。

---

對話三（Context Engineering）

```text
Phase 1 要做兩個版本：
A：直接呼叫 API，沒有 server layer（demo_claude_api.py）
B：加入 server layer，API Key 只放在 server 端（server.py + client.py）

設計重點不是功能，而是讓學習者「看到差異」，
為什麼 client 不能直接拿著 API Key 呼叫？
程式碼的結構和輸出訊息都要能清楚說明這一點。
```

A 和 B 功能完全相同，但學習目的不同。這段話讓 Claude Code 知道設計優先順序是「說得清楚」而不是「寫得簡潔」。server.py 的啟動訊息要明確印出「API Key is loaded — client will never see it.」，這是教學設計的一部分。

---

對話四（Review-and-Iterate）

```text
Phase 2 多供應商版本做完之後，我想加一個架構結論進文件：
OpenAI SDK 格式已經是本地模型的事實標準，
Ollama、LM Studio 都實作了 OpenAI 相容的 endpoint；
但 Anthropic 和 Google 的 SDK 沒有對應的路徑。
含義是：如果之後要把計算遷移到本地模型，用 OpenAI SDK 格式（即使呼叫 Claude 也透過 proxy）
會讓切換成本最低。請把這個洞察整理成獨立章節，加進 Markdown 和 HTML 兩個版本。
```

Phase 2 做完之後才看清楚的架構含義，不是修正錯誤，而是把實作過程裡得到的洞察回饋進文件，讓整個學習套件更完整。這是 Review-and-Iterate 的另一種形式。

---

## 反覆維護的內容追蹤工具

> 不是蓋一次就結束。

對話一（Spec-First）

```text
我想建一個工具，定期更新數位時代「一天一AI」專欄的文章索引。
需求是：
1. 資料來源：bnext.com.tw 的三頁文章列表
2. 本地存一份 .md 檔當作資料庫，記錄每篇文章的標題、時間、URL
3. 本地存一份 .html 互動式簡報，呈現分類和文章列表
4. 之後只要說「請更新」，就能自動完成：抓新文章 → 比對 → 更新 .md → 更新 .html

在開始動手之前，先幫我設計 CLAUDE.md，把更新流程寫成操作手冊。
```

這次 Spec-First 的重點不是某個功能的規格，而是定義「更新」這個操作本身的完整含義。在動手之前先寫 CLAUDE.md，等於把所有決策記錄下來。

怎麼識別新文章（用 article ID 不用標題）、MD 和 HTML 要同步更新、發布時間保留網站原始的相對格式，讓後續每次說「請更新」，都按同一份標準執行。

---

對話二（Atomic Tasking）

```text
好，先執行更新流程的第一步：
抓取三頁文章列表，列出所有新文章的標題和 URL。
不要動 .md 和 .html，讓我確認抓到的資料正確再繼續。
```

四步流程（抓取 → 比對 → 更新 MD → 更新 HTML）拆開執行，每次只做一步。第一步先確認資料來源是否正確，避免把錯誤的資料寫進文件才發現問題。確認之後再說「繼續」，Claude Code 才會動到本地檔案。

---

對話三（Context Engineering）

```text
HTML 的設計規範要鎖死，之後更新時不能跑掉：
- 配色用 Anthropic 暖棕系，CSS 變數我已經列在 CLAUDE.md 裡
- 六張 slide 順序固定：封面、更新摘要、主題分類、第一頁、第二頁、第三頁
- 主題分類每個條目必須是可點擊的 <a> 連結，不能只是純文字
- 鍵盤導覽保留，但禁止滑鼠點擊翻頁

把這些規則全部寫進 CLAUDE.md，確保下次更新不需要重新說明。
```

HTML 有 6 張 slide、自定義 CSS 變數、鍵盤導覽邏輯。如果設計規範只存在對話裡，下次更新時 Claude Code 就要重新猜配色用哪個？導覽怎麼處理？把規範全部寫進 CLAUDE.md，每次更新都當作第一次讀到一樣，不會因為對話上下文消失而走樣。

---

對話四（Review-and-Iterate）

```text
這次更新完我發現出現了幾篇新文章：
「他為弟弟 vibe coding，把幾百人的記憶拼成即時照片牆」
「API Key 是什麼？為什麼 Vibe Coding 也可能噴掉上萬元？」
這兩篇都在講從零開始用 AI 寫程式，跟現有八個分類都對不上。
你覺得這算新主題嗎？如果是，幫我提案一個分類名稱，
說明它跟「工具應用」和「AI 隱私安全」的邊界在哪裡。
```

更新了兩個月之後才看到某類文章開始積累但原有分類裝不下。使用者不需要自己決定，先問 Claude Code 分析邊界，確認判斷合理後再說「好，把這個分類加進去，順便把那兩篇文章移過來」。分類系統因為實際內容的積累而演化，這是長期維護任務的 Review-and-Iterate。

---

> 從整理 Excel、做個人 App、建稽核系統、開發 AI 影片工具、學習資料庫、建立自動化管線、理解協定、考量部署限制的靜態網站、設計給別人學的教材，到反覆維護的內容追蹤工具。
> 任務性質、技術複雜度、開發動機都不同，但驅動每一次有效協作的，始終是同一套溝通節奏。
