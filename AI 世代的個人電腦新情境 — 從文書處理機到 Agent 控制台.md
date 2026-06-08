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

# AI 世代的個人電腦新情境 — 從文書處理機到 Agent 控制台

## Part 2 : 什麼是 Agent ？

> 以前，電腦是你的工具。
> 現在，電腦是你的辦公室——而 AI Agent，是在裡面替你工作的員工。

[LLM Agent Stack](llm_agent_stack.html)

---

### 前置準備：個人筆電 和 Anthropic 帳戶

> 請使用自己的個人筆電上課：基於公司 IT 政策，公司電腦不適合安裝 Claude Code。
> 保持網路連線狀態。

一般的 Windows 10 或 Windows 11 筆電皆可使用 Claude Code。

| Apple | 晶片 | 是否適用 |
|:------|:------|:---------|
| MacBook Air / Pro（2020 年後）| Apple M1 / M2 / M3 / M4 | 完全適用，效能極佳 |
| MacBook Air / Pro（2017–2020）| Intel Core i5 / i7 | 完全適用 |
| 2016 年以前的 MacBook | Intel | 建議 macOS 12（Monterey）以上|

---

#### Step 1：建立 Google 帳戶

前往 [accounts.google.com/signup](https://accounts.google.com/signup)，依照指示填寫姓名、使用者名稱、密碼，完成手機號碼驗證後即可啟用。

> 如果你已有 Google 帳戶（Gmail、Google Drive、YouTube 等），跳過此步驟，直接使用現有帳號即可。

Google 帳戶在本課程中扮演三個角色：

| 用途 | 說明 |
|------|------|
| 登入 Anthropic | Step 2 建立 Claude 帳號時，推薦用 Google 登入，省去記一組新密碼 |
| 雲端備份 | Google Drive 可儲存課程素材與輸出成果 |
| 跨裝置同步 | 筆電與手機可存取相同的 Google 帳號資源 |

---

#### Step 2：建立 Anthropic 帳號

[claude.ai](https://claude.ai), 推薦 Google 的登入方式

#### Step 3：升級為 Pro 用戶

> 訂閱是月付制，你可以隨時在帳號設定中取消，不會被鎖死。
> 訂閱之後怎麼取消？
> 進入 Settings → Billing → Manage Subscription，點選取消。取消後，你的 Pro 資格會維持到當月結束，之後自動降為免費版，不會退費已付的月費。

1. 登入 Claude 的網頁介面後， 點選左下角的 「Upgrade to Pro」（或 「升級方案」）
2. 選擇 「Claude Pro」 方案
3. 填入信用卡資料，確認訂閱，完成付款

---

#### Step 4：確認 Pro 狀態

升級完成後，你可以在網頁版對話介面的左下角看到你的帳號名稱，旁邊會出現「Pro」的標示。
或者：點選右上角的帳號頭像 → 「Settings」 → 「Billing」，確認方案顯示為「Claude Pro」。

#### 前置準備完成

---

### 2-1 VS Code

1991 年，微軟推出 Word 1.0，從此「個人電腦」在多數人眼中就等於「文書處理機」。三十年後，同一家公司的另一款軟體正在重新定義這件事——那就是 Visual Studio Code。

VS Code 的誕生是為了寫程式。但它在 2024–2025 年的爆炸性成長，很大一部分來自非程式設計師：行銷人員、產品經理、研究員、顧問。他們用 VS Code 寫筆記、管文件、跑 AI 助理，把這個輕量的文字編輯器當成新一代的工作台。

原因很簡單：VS Code 是目前唯一對 AI Agent 友善的通用工作環境。

---

| VS Code 的核心能力 | 說明 |
|------|------|
| 檔案總管 | 以資料夾為單位管理專案，Agent 能看到整個工作目錄的結構 |
| 整合終端機 | 直接在編輯器內執行指令，Agent 可在此運行程式、安裝工具 |
| 延伸套件 | 超過 5 萬個套件，AI 助理、Git 介面、語言支援一鍵安裝 |
| 即時預覽 | Markdown、HTML、CSV 等格式可在編輯時即時看到結果 |
| 跨平台同步 | Windows、Mac、Linux 以及瀏覽器版本，設定和套件同步 |

對知識工作者來說，VS Code 最重要的不是「寫程式」，而是它提供了一個 Agent 可以感知環境、執行動作、讀寫檔案的操作空間——這是 Word 或 Google Docs 永遠做不到的事。

> 把 VS Code 想像成一間附有工具間的辦公室。Word 是一張只有書桌的房間；VS Code 是一棟裡面有檔案室、工作台、對講機、任何工具隨時可以拿的建築。Agent 需要的正是這樣的空間。

[Visual Studio Code](https://code.visualstudio.com)

---

#### VS Code 簡易操作

| 操作 | 快捷鍵（Win / Mac）| 備註 |
|------|-------------------|------|
| 安裝 Extension | Ctrl+Shift+X / ⌘+Shift+X | 左側 Extensions 圖示 → 搜尋名稱 → Install |
| 開啟資料夾 | Ctrl+K, Ctrl+O / ⌘K, ⌘O | 把整個工作目錄交給 VS Code 管理 |
| 開啟檔案 | Ctrl+O / ⌘+O | 快速開啟單一檔案 |
| 建立新檔 | Ctrl+N / ⌘+N | 或在左側檔案總管右鍵 → New File |
| 儲存檔案 | Ctrl+S / ⌘+S | 養成頻繁儲存的習慣 |
| 開啟終端機 | Ctrl+\` / Control+\` | 在 VS Code 內直接執行指令、啟動 Claude Code |
| 命令列面板 | Ctrl+Shift+P / ⌘+Shift+P | 搜尋所有功能的統一入口 |

---

### 2-2 Markdown

Markdown 是 2004 年由 John Gruber 發明的一種純文字標記語言，設計目標只有一個：讓人可以用鍵盤上已有的符號來排版，寫出來的文字直接閱讀也不覺得奇怪。

```markdown
# 這是標題

這是一段普通文字，*這是斜體*。

- 項目一
- 項目二

> 這是引言區塊
```

二十年後，Markdown 意外地成為 AI 時代的「通用語言」——幾乎所有大型語言模型，輸出時預設就是 Markdown 格式。

---

為什麼 Markdown 對知識工作者愈來愈重要？

它是純文字，沒有隱藏格式。

Word 檔案是二進位格式，裡面藏著字型資訊、樣式定義、版面設定，AI 工具很難直接讀取。Markdown 是純文字，任何工具都能讀，任何 AI 都能處理，用 Git 做版本控制也不會炸掉差異比對。

它和 AI 說同一種語言。

當你要求 Claude 或 GPT 整理會議記錄，它輸出的格式幾乎必然是 Markdown。如果你的工作筆記本來就是 Markdown，就等於你和 AI 之間不需要格式翻譯——提示詞、輸出、儲存，都在同一個格式生態裡流轉。

---

Markdown 與 Word 的實際差異：

| 比較項目 | Word / Google Docs | Markdown |
|---------|-------------------|----------|
| 儲存格式 | 二進位或 XML | 純文字（.md）|
| AI 可讀性 | 需要轉換 | 直接讀取 |
| 版本控制 | 難以逐行比對 | 完整支援 Git diff |
| 跨工具相容 | 格式常跑掉 | 任何文字編輯器都能開 |
| 學習成本 | 幾乎為零（已會用）| 30 分鐘上手 |

Markdown 語法只有十幾個符號，`#` 代表標題、`**` 代表粗體、`-` 代表清單、`>` 代表引言，其餘都是普通文字。

> 這份投影片本身，就是用 Markdown 寫成的。

---

### 2-2.5 Marp — Markdown 直接變成投影片

「這份投影片本身，就是用 Markdown 寫成的。」——更精確地說，是用 Marp 製作的。

Marp（Markdown Presentation Ecosystem）是一套把 Markdown 渲染成投影片的工具。在 `.md` 檔案開頭加上 frontmatter 宣告，三條橫線 `---` 就是換頁。你只需要寫文字，版面與配色由設定決定。

```markdown
---
marp: true
theme: default
paginate: true
---

# 第一張投影片

這是內文

---

## 第二張投影片

用三條橫線分隔每一頁
```

---

Marp 與傳統簡報工具的差異：

| 比較項目 | PowerPoint / Keynote | Marp |
|---------|---------------------|------|
| 製作方式 | 拖拉排版 | 純文字撰寫 |
| 版本控制 | 難以追蹤變更 | 完整支援 Git diff |
| AI 可生成 | 需要外掛或手動輸入 | AI 直接輸出完整 .md |
| 跨平台播放 | 需要對應軟體 | 匯出 HTML，任何瀏覽器皆可開 |
| 主題切換 | 逐頁調整 | 改一行 frontmatter，全部更新 |

> 這份課程的所有投影片，都是用 Claude Code + Marp 製作的——從空白資料夾到你現在看到的版本，沒有手動調整過任何一張投影片的版面。

---

#### 推薦安裝的延伸套件

安裝方式：點選左側「Extensions」圖示（`Ctrl+Shift+X`）→ 搜尋套件名稱 → 點選 Install。

| 套件名稱 | 發布者 | 功能說明 |
|---------|--------|---------|
| Marp for VS Code | marp-team | 將 Markdown 渲染成投影片，支援即時預覽與匯出 |
| markdownlint | David Anson | 自動檢查 Markdown 語法規範，提示格式錯誤 |
| Material Icon Theme | Philipp Kief | 以直覺圖示區分不同類型的檔案與資料夾 |
| Chinese (Traditional) Language Pack | Microsoft | 將 VS Code 介面切換為繁體中文 |

---

#### 讓 AI 幫你生成簡報

```
請用 Marp 格式製作一份 8 頁簡報，主題是：
「為什麼我們的團隊需要 AI 工具」
聽眾：主管，不懂技術，重視成本
風格：每頁不超過 5 個重點，繁體中文
請直接輸出完整的 .md 檔案內容，包含 frontmatter
```

---

### 2-3 Claude Code

2025 年，Anthropic 推出了一個改變工程師工作方式的產品：Claude Code（[code.claude.com](https://code.claude.com)）。

Claude Code 不是一個聊天視窗，而是一個在你的電腦終端機裡運作的 AI Agent。它不只是回答問題，它能直接讀取你的檔案、執行程式、修改程式碼、查看輸出結果，並根據結果調整下一步的行動。

用一個比喻來說：如果 ChatGPT 是你的電話顧問——你打過去問，它給建議，但它看不到你桌上的東西、也不能幫你動手——那麼 Claude Code 就是坐在你旁邊的同事，你的螢幕它看得到，鍵盤它也能打。

---

Claude Code 能做什麼：

| 能力 | 說明 |
|------|------|
| 讀取整個專案 | 掃描資料夾結構，理解各檔案的關聯與脈絡 |
| 讀寫任意檔案 | 直接建立、修改、刪除工作目錄中的文件 |
| 執行終端機指令 | 安裝套件、執行測試、呼叫外部工具 |
| 搜尋程式碼庫 | 用關鍵字或模式在整個專案中定位相關程式碼 |
| 多步驟任務規劃 | 把複雜任務拆解成子步驟，逐步執行並回報進度 |
| Git 整合 | 讀取差異、撰寫 commit 訊息、管理分支 |

Claude Code 的操作介面是終端機的命令列，但你也可以在 VS Code 裡直接呼叫它——這就是為什麼 VS Code 是它最自然的工作環境。

---

Claude Code 的運作邏輯，完全承繼了 Part 1 介紹的基礎：

```
你的指令（Prompt）
    ↓
Claude Code 讀取當前專案的上下文（Context Window）
    ↓
呼叫工具（Tool Use）：讀檔、搜尋、執行指令
    ↓
根據工具回傳結果，決定下一步行動（Agentic Loop）
    ↓
輸出結果：修改後的檔案、執行報告、提問確認
```

每一步都透明可見，每一次檔案修改都被 Git 記錄——這是一個被設計為可審計、可撤回的 Agent 環境，而不是一個黑盒子。

> Claude Code 的設計哲學是「人類在圈內（Human-in-the-loop）」——它在做重大決策前會停下來確認，而不是默默地把你的資料夾改得面目全非。你設定的信任邊界，決定了它能自主行動的範圍。

---

#### 終端機（Terminal）

終端機是電腦的純文字操作介面。圖形界面出現之前，所有電腦都靠它操作：你輸入一行文字指令，電腦執行後把結果印回來。

Claude Code 就活在終端機裡——它不是一個有視窗的應用程式，而是你用文字呼叫、以文字回應的 Agent。在 VS Code 裡，也內建了終端機，Claude Code 直接在裡面執行，不需要另外開視窗。

| 終端機元素 | 說明 |
|-----------|------|
| 提示符號 | `>` 或 `$` 開頭那行，代表電腦在等你輸入 |
| 路徑 | 提示符號前的文字，代表你現在「在哪個資料夾」|
| 指令 | 你輸入的文字，例如 `claude`、`node --version` |
| 輸出 | 執行後顯示的結果，可能是文字、錯誤訊息，或什麼都沒有 |

> 好消息：使用 Claude Code 時，你幾乎不需要自己輸入終端機指令——Claude Code 會替你打。你只需要知道終端機是什麼，以及如何在 VS Code 裡開啟它。

---

#### Node.js

Node.js 是讓 JavaScript 能在瀏覽器之外、在你電腦本機執行的環境。你不需要學 JavaScript，也不需要了解 Node.js 的運作細節。你只需要知道一件事：Claude Code 是一個 Node.js 應用程式，安裝它必須透過 npm。

npm（Node Package Manager）是 Node.js 內建的套件管理工具。就像 App Store 是你安裝手機應用程式的管道，npm 是 Node.js 生態系的工具倉庫——一行指令，把工具裝進你的電腦：

```
npm install -g @anthropic-ai/claude-code
```

| npm 概念 | 說明 |
|---------|------|
| npm | Node.js 的套件管理工具，隨 Node.js 自動安裝 |
| `install` | 下載並安裝套件到本機 |
| `-g`（global）| 全系統安裝，電腦任何資料夾都能呼叫，只裝一次即可 |
| LTS 版 | Long-Term Support，穩定長期支援版，推薦一般使用者安裝 |

---

#### 安裝 Claude Code

安裝 Claude Code 前，必須先安裝 Node.js——Claude Code 透過 npm（Node.js 的套件管理工具）安裝，沒有 Node.js 就無法執行安裝指令。

| 步驟 | 操作 |
|------|------|
| 1. 安裝 Node.js（LTS 版）| [nodejs.org](https://nodejs.org) 下載安裝，npm 一併就緒 |
| 2. 確認安裝成功 | 在終端機執行 `node --version` 與 `npm --version` |
| 3. 安裝 Claude Code | `npm install -g @anthropic-ai/claude-code` |
| 4. 啟動 | 在 VS Code 終端機執行 `claude` |

> 第一次執行 `claude` 時，系統會要求以 Anthropic 帳號登入並授權。完成後，在任何資料夾的終端機輸入 `claude`，即可呼叫 Claude Code 開始工作。

---

### 2-4 Git

你有沒有這樣的資料夾：

```
報告_最終版.docx
報告_最終版_修改後.docx
報告_最終版_修改後_確認版.docx
報告_最終版_修改後_確認版_老闆改過.docx
```

Git 它讓你只需要一份檔案，卻能看到任何歷史版本、隨時回到過去任何一個時間點。

好消息是：你不需要自己學 Git 指令。Claude Code 會幫你操作 Git。

你真正需要的，是理解 Git 的概念——這樣當 Claude Code 詢問你的時候，你才能做出正確的決定。

---

#### 安裝 Git 與初次設定

[Git 官方下載](https://git-scm.com/downloads)（Windows / Mac / Linux 皆適用）

初次使用 Git，需要先進行全域設定，告訴 Git 你是誰——這樣每一筆 commit 才知道是誰操作的：

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

這個設定只需要做一次，之後所有 repo 都會沿用。

---

#### 你的新角色：監督者，而非操作者

過去學 Git 最難的地方，是要記一堆指令。現在這件事由 Claude Code 代勞：

```
你：「整理這份會議記錄，存成 Markdown」
    ↓
Claude Code 修改檔案，然後詢問你：
「已完成修改。要 commit 嗎？
 建議訊息：'整理 0606 會議記錄為 Markdown 格式'」
    ↓
你：確認訊息正確，按下同意
    ↓
Claude Code 執行 git commit
```

Git 指令由 Claude Code 負責輸入。你負責理解它在做什麼，並決定要不要同意。

---

#### Repository — Claude Code 的工作場域

Repository（簡稱 repo）是 Git 管理的基本單位——一個有完整歷史記錄的資料夾。當你在 VS Code 開啟一個資料夾，那個資料夾就可以初始化成一個 repo。

```
my-project/              ← Claude Code 看到的工作範圍
├── .git/                ← Git 的帳本（隱藏資料夾，不要手動修改）
├── meeting-notes.md
├── report.md
└── data.csv
```

Claude Code 啟動時，它掃描的就是這個資料夾。它能看見所有檔案，也能讀取 `.git/` 裡的完整歷史——這就是為什麼它能告訴你：「上次修改這個檔案是三天前，改了哪幾行。」

---

#### Commit — Claude Code 的存檔點

Commit 是把當前的變更打包成一個有名字的歷史節點，附上說明記錄「這次改了什麼、為什麼改」。

Claude Code 完成一段工作後，通常會主動建議 commit。你需要判斷的事只有兩件：

1. 這次的變更值得留下一個記錄點嗎？（通常是：是）
2. commit 的說明描述正確嗎？（可以直接接受，也可以要求 Claude Code 修改）

Branch（分支）和 Merge（合併）也是 Claude Code 會自動處理的動作——它會在需要實驗的時候開分支，完成後再合併，整個過程都會告知你。

---

#### Remote — 把帳本搬上雲端

本機的 repo 只活在你的電腦裡。Remote 是在 GitHub 建立一份鏡像，讓你的工作有雲端備份。你的電腦和 GitHub 之間只有一個方向概念需要記：

```
你的電腦（本機 repo）
    |  push：把本機的新 commit 上傳到雲端
    |  pull：把雲端的最新變更拉回本機
    v
遠端 repo（GitHub / GitLab）
    |  push  ^  pull
    v
同事的電腦（本機 repo）
```

第一次將本機 repo 與 GitHub 連結，只需執行一次：

```bash
git remote add origin https://github.com/你的帳號/repo名稱.git
git push -u origin main
```

`-u` 的作用是「記住這條上傳路徑」——設定過一次，之後直接 `git push` 就能推到同一個地方，不需要重複指定。

---

#### 第一次 push 的 GitHub 驗證

第一次 push 時，Git 需要確認你有權限寫入那個 GitHub repo。Windows 和 Mac 的處理方式不同：

| 平台 | 驗證機制 | 你需要做什麼 |
|------|---------|------------|
| Windows | Git Credential Manager（Git for Windows 內建）| 第一次 push 自動開啟瀏覽器 → 登入 GitHub → 授權完成，憑證存入系統 |
| Mac | 需要先安裝 GitHub CLI | 在終端機執行 `brew install gh`，再執行 `gh auth login`，依提示在瀏覽器完成授權 |

兩種方式完成後的結果相同：之後每次 `git push`，不需要再輸入帳號密碼或重新驗證。

> Mac 用戶：`brew` 是 macOS 的套件管理工具（Homebrew），如果還沒安裝，先前往 [brew.sh](https://brew.sh) 安裝，再執行上述指令。

---

#### 你真正需要會的三件事

不是指令，而是讀懂 Claude Code 給你看的資訊：

| 情境 | Claude Code 怎麼呈現 | 你要做什麼 |
|------|---------------------|-----------|
| 確認 AI 改了什麼 | 列出 `git diff`：哪些行被新增或刪除 | 確認改動符合你的意圖 |
| 回顧工作軌跡 | 列出 `git log`：每個 commit 的時間與說明 | 確認進度與操作記錄 |
| 發現改錯了 | 建議執行 `git revert` | 同意後，Claude Code 自動復原到指定版本 |

> Git 儲存你的歷史，但它能做的遠不止於此。同一個 repo，可以是你的雲端備份、你的協作空間、你的網站發布按鈕、你的定時自動化工作流程。當 AI Agent 住進這個生態系，你得到的不只是一個追蹤記錄的工具，而是一條從構想到發布、從指令到執行的現代工作管道。

---

### 2-5 Agentic Coding

「Agentic Coding」是 2024–2025 年軟體開發界最劇烈的典範轉移，但它的影響範圍遠超過工程師群體。

傳統的 AI 輔助寫作（如 GitHub Copilot 的自動補全）是一個反射動作：你打幾個字，AI 猜你接下來要寫什麼，你接受或拒絕。整個過程是逐行、即時、被動的。

Agentic Coding 完全不同。它是一個完整的工作循環：

```
你提出目標（不是指令，是目標）
    ↓
Agent 規劃步驟
    ↓
Agent 執行 → 觀察結果 → 調整計畫 → 再執行
    ↓
Agent 回報完成，請你審查
```

你給的不是「幫我在第 37 行加一個函式」，而是「幫我把這份 Excel 資料整理成每月銷售報告，輸出成 PDF，並寄給行銷團隊」。

---

Agentic Coding 能處理的任務類型：

| 任務類型 | 傳統方式 | Agentic Coding |
|---------|---------|----------------|
| 建立新功能 | 工程師逐行撰寫 | 描述需求，Agent 生成並測試 |
| 重構舊程式碼 | 逐檔手動修改 | Agent 掃描全庫、批次重構 |
| 撰寫測試 | 工程師手寫測試案例 | Agent 分析程式碼，自動生成測試 |
| 調查 Bug | 逐步加印 debug 訊息 | Agent 追蹤錯誤訊息、定位根源 |
| 資料處理 | 人工寫腳本或套用公式 | Agent 理解需求，直接產出處理腳本 |

對非工程師的知識工作者而言，Agentic Coding 最大的意義在於：你不需要知道怎麼寫程式，只需要能清楚描述你想要什麼結果。 Agent 負責把自然語言翻譯成可執行的步驟。

---

Agentic Coding 的核心循環（ReAct Loop）：

```
Reason（推理）：根據目標與當前狀態，思考下一步該做什麼
    ↓
Act（行動）：執行工具——讀檔、寫檔、執行命令、搜尋
    ↓
Observe（觀察）：把工具回傳的結果讀入上下文
    ↓
回到 Reason，繼續下一輪循環
```

這個循環可以持續數十輪，直到任務完成或遇到需要人類判斷的岔路口。每一輪循環都在消耗 Token（參考 1-7），這也是為什麼複雜的 Agentic 任務費用遠高於單次問答——它本質上是讓 AI 「想了很久才給你答案」。

> 黃仁勳說的「Token 消耗量代表你讓 AI 參與工作的程度」，在 Agentic Coding 這裡得到最直接的體現——一個真正跑完整任務的 Agent，往往會燒掉比你預期多十倍的 Token，因為它在背後做了大量你看不見的思考與嘗試。

---

Agentic Coding 的人機分工：

當 Agent 能自主完成大部分執行工作，人類的價值在哪裡？

| 工作類型 | Agent 擅長 | 人類仍不可或缺 |
|---------|-----------|--------------|
| 執行 | 快速、不眠、可批量 | 判斷「這個目標值得做嗎」|
| 記憶 | 能讀取所有文件 | 知道哪些知識「重要但沒有記錄」|
| 一致性 | 遵循規則不會忘 | 發現規則本身需要改變 |
| 試錯 | 可以快速迭代 | 決定「夠好了，可以停」的時機 |
| 溝通 | 產出文字報告 | 讀懂利害關係人真正想要什麼 |

Agentic Coding 不是「AI 取代工程師」，而是讓每一位知識工作者都擁有一個能獨立執行任務的工程師助理——前提是你能清楚地設定目標、審查結果、並為最終決策負責。

---

從文書處理機到 Agent 控制台，本質上是一個工作授權的轉移。

文書處理機時代，電腦執行你每一個細微的操作——你按 B，字就變粗體；你按 Enter，游標就換行。電腦是工具，人是操作者。

Agent 控制台時代，你設定目標與邊界，Agent 自行決定每一個細微的步驟——它選擇讀哪個檔案、執行什麼指令、以什麼順序進行。人是決策者，Agent 是執行者。

這個轉移要求你具備新的能力：不是打字更快，而是描述目標更清晰；不是記更多操作步驟，而是懂得審查 Agent 的行為是否符合你的意圖。

> 在 AI Agent 的世界裡，能力的天花板不再是「你能自己做多少」，而是「你能讓 Agent 替你做多少，同時不失去對結果的掌控」。這是新一代知識工作者的核心競爭力。

---

## 小結：Part 2 概念關係總覽

```
VS Code  ────────────────────────────── Agent 能感知與操作的工作環境
  |
  ├─► Markdown  ──────────────────────── AI 原生的文件格式，人機共讀
  |     └─► Marp  ────────────────────── Markdown 直接渲染成投影片
  |
  ├─► Git  ───────────────────────────── Claude Code 的操作記錄，你的審查安全網
  |     ├─► Repository  ──────────────── Claude Code 的工作場域
  |     └─► Commit / Remote  ─────────── 由 Claude Code 執行，你確認訊息並 push
  |
  └─► Claude Code  ───────────────────── 在終端機裡運作的 AI Agent
        ├─► Node.js（npm）  ────────────── Claude Code 的安裝前置條件
        └─► Agentic Coding  ────────────── 目標導向的人機協作工作模式
              ├─► ReAct Loop  ─────────── Reason → Act → Observe 的執行循環
              └─► 人機分工  ────────────── 人負責目標與審查，Agent 負責執行
```

---

> 恭喜你，你已經準備好了在 AI 時代重新定義自己工作方式的完整框架。
