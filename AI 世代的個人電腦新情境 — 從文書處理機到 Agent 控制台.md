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

### 前置準備：個人筆電

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

| 用途 | 說明 |
|------|------|
| 登入 GitHub | Step 2 建立 GitHub 帳號時，可直接以 Google 帳號登入 |
| 登入 Anthropic | Step 3 建立 Claude 帳號時，可直接以 Google 帳號登入 |

---

#### Step 2：建立 GitHub 帳號

前往 [github.com/signup](https://github.com/signup)，填寫使用者名稱、電子郵件與密碼，完成電子郵件驗證後即可啟用。推薦以 Google 帳號直接登入，省去記一組新密碼。

> 如果你已有 GitHub 帳號，跳過此步驟，直接使用現有帳號即可。

GitHub 在本課程中扮演兩個角色：

| 用途 | 說明 |
|------|------|
| 儲存 repo | 把你的工作資料夾推送到雲端，任何時間、任何裝置都能取回 |
| 協作與分享 | 課程成果可以分享連結，或與他人共同維護同一份專案 |

---

#### Step 3：建立 Anthropic 帳號

[claude.ai](https://claude.ai), 推薦 Google 的登入方式

#### Step 4：升級為 Pro 用戶

> 訂閱是月付制，你可以隨時在帳號設定中取消，不會被鎖死。
> 訂閱之後怎麼取消？
> 進入 Settings → Billing → Manage Subscription，點選取消。取消後，你的 Pro 資格會維持到當月結束，之後自動降為免費版，不會退費已付的月費。

1. 登入 Claude 的網頁介面後， 點選左下角的 「Upgrade to Pro」（或 「升級方案」）
2. 選擇 「Claude Pro」 方案
3. 填入信用卡資料，確認訂閱，完成付款

---

#### Step 5：確認 Pro 狀態

升級完成後，你可以在網頁版對話介面的左下角看到你的帳號名稱，旁邊會出現「Pro」的標示。
或者：點選右上角的帳號頭像 → 「Settings」 → 「Billing」，確認方案顯示為「Claude Pro」。

#### Step 6：預先下載三個工具的安裝檔

課堂上會逐步安裝以下工具。請在上課前先下載安裝檔，存放在桌面或下載資料夾即可，**不需要現在安裝**。

| 工具 | 下載來源 | 備註 |
|------|---------|------|
| Node.js | [nodejs.org](https://nodejs.org) | 選 LTS 版本，用途是安裝 Claude Code |
| Git | [git-scm.com/downloads](https://git-scm.com/downloads) | 選 Windows / Mac 對應版本 |
| VS Code | [Visual Studio Code](https://code.visualstudio.com) | 選 Windows / Mac 對應版本 |

> 課堂網路環境不一定穩定，提前下載可以讓安裝過程更順暢。安裝步驟將於課堂中逐一進行，講師會全程在場協助，不需要自己摸索。

---

### 2-1 Claude Code

2025 年，Anthropic 推出了 Claude Code。

Claude Code 不是一個聊天視窗，而是一個在你的電腦裡運作的 AI Agent。它不只是回答問題，它能直接讀取你的檔案、修改內容、執行任務、查看結果，並根據結果調整下一步的行動。

用一個比喻來說：
如果 ChatGPT 是你的電話顧問——你打過去問，它給建議，但它看不到你桌上的東西、也不能幫你動手
那麼 Claude Code 就是坐在你旁邊的同事，你的螢幕它看得到，鍵盤它也能打。

> 一個讓 AI 真正在你的電腦上工作的工具

---

#### 安裝 Claude Code

安裝 Claude Code 前，必須先安裝 Node.js——Claude Code 透過 npm（Node.js 的套件管理工具）安裝，沒有 Node.js 就無法執行安裝指令。

| 步驟 | 操作 |
|------|------|
| 1. 安裝 Node.js（LTS 版）| [nodejs.org](https://nodejs.org) 下載安裝，npm 一併就緒 |
| 2. 確認安裝成功 | 在終端機執行 `node --version` 與 `npm --version` |
| 3. 安裝 [Claude Code](https://code.claude.com) | `npm install -g @anthropic-ai/claude-code` |
| 4. 啟動 | 在終端機執行 `claude` |

> 第一次執行 `claude` 時，系統會要求以 Anthropic 帳號登入並授權。完成後，在任何資料夾的終端機輸入 `claude`，即可呼叫 Claude Code 開始工作。

---

Claude Code 的運作邏輯，完全承繼了 Part 1 介紹的基礎：

```
你的指令（Prompt）
    ↓
Claude Code 讀取當前專案的上下文（Context Window）
    ↓
呼叫工具（Tool Use）：讀檔、搜尋、執行指令
    ↓
根據工具回傳結果，決定下一步行動，持續循環直到完成
    ↓
輸出結果：修改後的檔案、執行報告、提問確認
```
> Claude Code 的設計哲學是「人始終在決策圈內」——它在做重大決策前會停下來確認，而不是默默地把你的資料夾改得面目全非。你設定的信任邊界，決定了它能自主行動的範圍。

---

#### Claude Code 能做什麼

| 能力 | 說明 |
|------|------|
| 讀取整個專案 | 掃描資料夾結構，理解各檔案的關聯與脈絡 |
| 讀寫任意檔案 | 直接建立、修改、刪除工作目錄中的文件 |
| 執行系統指令 | 安裝工具、處理檔案、呼叫外部服務 |
| 搜尋整個資料夾 | 用關鍵字在所有文件中找到相關內容 |
| 多步驟任務規劃 | 把複雜任務拆解成子步驟，逐步執行並回報進度 |
| Git 整合 | 讀取差異、撰寫 commit 訊息、管理分支 |

> Claude Code 的操作介面是終端機的命令列，但你也可以在 VS Code 裡直接呼叫它——這就是為什麼 VS Code 是它最自然的工作環境。

---

### 2-2 VS Code

1991 年，微軟推出 Word 1.0，從此「個人電腦」在多數人眼中就等於「文書處理機」。三十年後，同一家公司的另一款軟體正在重新定義這件事——那就是 Visual Studio Code。

VS Code 的誕生是為了寫程式。但它在 2024–2025 年的爆炸性成長，很大一部分來自非程式設計師：行銷人員、產品經理、研究員、顧問。他們用 VS Code 寫筆記、管文件、跑 AI 助理，把這個輕量的文字編輯器當成新一代的工作台。

原因很簡單：VS Code 是目前唯一對 AI Agent 友善的通用工作環境。

---

#### 推薦安裝的延伸套件

安裝方式：點選左側「Extensions」→ 搜尋套件名稱 → 點選 Install。

| 套件名稱 | 發布者 | 功能說明 |
|---------|--------|---------|
| Marp for VS Code | marp-team | 將 Markdown 渲染成投影片，支援即時預覽與匯出 |
| markdownlint | David Anson | 自動檢查 Markdown 語法規範，提示格式錯誤 |
| Material Icon Theme | Philipp Kief | 以直覺圖示區分不同類型的檔案與資料夾 |
| Chinese (Traditional) Language Pack | Microsoft | 將 VS Code 介面切換為繁體中文 |

> 對知識工作者來說，VS Code 最重要的不是「寫程式」，而是它提供了一個 Agent 可以感知環境、執行動作、讀寫檔案的操作空間。

---

#### Markdown

Markdown 是 2004 年由 John Gruber 發明的一種純文字標記語言，設計目標只有一個：讓人可以用鍵盤上已有的符號來排版，寫出來的文字直接閱讀也不覺得奇怪。

Markdown 語法只有十幾個符號，`#` 代表標題、`**` 代表粗體、`-` 代表清單、`>` 代表引言，其餘都是普通文字。

```markdown
# 這是標題

這是一段普通文字，*這是斜體*。

- 項目一
- 項目二

> 這是引言區塊
```

---

#### Markdown 和 AI 說同一種語言

如果你的工作筆記本來就是 Markdown，就等於你和 AI 之間不需要格式翻譯——提示詞、輸出、儲存，都在同一個格式生態裡流轉。

> 這份投影片本身，就是用 Markdown 寫成的。

---

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

#### 傳統簡報工具和 Marp 的差異

| 比較項目 | PowerPoint / Keynote | Marp |
|---------|---------------------|------|
| 製作方式 | 拖拉排版 | 純文字撰寫 |
| 版本控制 | 難以追蹤變更 | 完整支援 Git diff |
| AI 可生成 | 需要外掛或手動輸入 | AI 直接輸出完整 .md |
| 跨平台播放 | 需要對應軟體 | 匯出 HTML，任何瀏覽器皆可開 |
| 主題切換 | 逐頁調整 | 改一行 frontmatter，全部更新 |

> 這份課程的所有投影片，都是用 Claude Code + Marp 製作的——從空白資料夾到你現在看到的版本，沒有手動調整過任何一張投影片的版面。

---

#### 練習：讓 AI 幫你生成簡報

```
請用 Marp 格式製作一份 8 頁簡報，主題是：
「為什麼我們的團隊需要 AI 工具」
聽眾：主管，不懂技術，重視成本
風格：每頁不超過 5 個重點，繁體中文
請直接輸出完整的 .md 檔案內容，包含 frontmatter
```

---

### 2-3 Git

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

Repository（簡稱 repo）是 Git 管理的基本單位，是一個有完整歷史記錄的資料夾。
當你在 VS Code 開啟一個資料夾，那個資料夾就可以初始化成一個 repo。

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

---

#### 初次使用 Git

需要先進行全域設定，告訴 Git 你是誰——這樣每一筆 commit 才知道是誰操作的。
這個設定只需要做一次，之後所有 repo 都會沿用。

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

第一次將本機 repo 與 GitHub 連結，只需執行一次。
`-u` 的作用是「記住這條上傳路徑」——設定過一次，之後直接 `git push` 就能推到同一個地方，不需要重複指定。

```bash
git remote add origin https://github.com/你的帳號/repo名稱.git
git push -u origin main
```

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

#### 第一次 push 的 GitHub 驗證

第一次 push 時，Git 需要確認你有權限寫入那個 GitHub repo。Windows 和 Mac 的處理方式不同：

| 平台 | 驗證機制 | 你需要做什麼 |
|------|---------|------------|
| Windows | Git Credential Manager（Git for Windows 內建）| 第一次 push 自動開啟瀏覽器 → 登入 GitHub → 授權完成，憑證存入系統 |
| Mac | 需要先安裝 GitHub CLI | 在終端機執行 `brew install gh`，再執行 `gh auth login`，依提示在瀏覽器完成授權 |

兩種方式完成後的結果相同：之後每次 `git push`，不需要再輸入帳號密碼或重新驗證。

> Mac 用戶：`brew` 是 macOS 的套件管理工具（Homebrew），如果還沒安裝，先前往 [brew.sh](https://brew.sh) 安裝，再執行上述指令。

---

### 2-4 從「操作工具」到「指揮 Agent」

過去使用電腦，你需要「知道怎麼做」才能讓電腦幫你做：要合併 Excel，你得會公式；要批次命名檔案，你得懂腳本；要排版投影片，你得熟悉軟體操作。

這件事正在改變。

Claude Code 理解你說的目標，然後自己規劃步驟、找工具、執行、回報。你不需要知道「怎麼做」，只需要說清楚「要什麼」。

這不是更先進的搜尋引擎，也不是語音助理——而是一位有能力獨立工作、但每一步都在等你確認的數位員工。

---

以「整理會議記錄」為例，看看你說完之後，Agent 實際上做了什麼：

```
你說：把 transcript.txt 整理成結構化會議記錄，輸出為 .md

  規劃 ── 需要先讀取檔案，了解內容結構
    ↓
  行動 ── 讀取 transcript.txt
    ↓
  觀察 ── 3,200 字，5 位出席者，討論了 3 個議題
    ↓
  規劃 ── 提取決議事項、待辦清單、待確認問題
    ↓
  行動 ── 寫入 meeting_0606.md
    ↓
  觀察 ── 完成，請你確認內容是否符合預期
```

「規劃 → 行動 → 觀察」這個循環，可以跑幾輪、幾十輪，直到任務完成——或遇到需要你判斷的地方才停下來。

---

當 Agent 能自主完成大部分執行工作，你的位置在哪裡？

| 這件事 | Agent 擅長做 | 你仍不可或缺 |
|--------|------------|------------|
| 執行 | 快、不停、可批量 | 判斷「這個方向值得花時間嗎」 |
| 記憶 | 讀遍你給的所有資料 | 知道哪些背景「沒有記錄但很關鍵」 |
| 一致性 | 照規則跑不會忘 | 察覺規則本身需要更新 |
| 試錯 | 快速迭代不怕重來 | 判斷「夠好了，可以交出去了」 |
| 文字產出 | 速度快、格式整齊 | 知道這份文件的真正讀者要什麼 |

> 能力的天花板不再是「你能自己做多少」，而是「你能讓 Agent 有效替你完成多少——同時保持對結果的掌控」。

---

## 小結：Part 2 概念關係總覽

```
Claude Code  ──────────────────────────── 在你電腦裡工作的 AI Agent
  |
  ├─► Node.js（npm）  ────────────────── 安裝 Claude Code 的前置條件
  |
  ├─► VS Code  ──────────────────────── Agent 能感知與操作的工作環境
  |     ├─► Markdown  ────────────────── AI 原生的文件格式，人機共讀
  |     └─► Marp  ──────────────────── Markdown 直接渲染成投影片
  |
  ├─► Git  ──────────────────────────── 你的審查安全網與操作記錄
  |     ├─► Repository  ────────────── Claude Code 的工作場域
  |     ├─► Commit  ────────────────── 由 Claude Code 執行，你確認訊息
  |     └─► GitHub（Remote）  ────────── 雲端備份與協作，push / pull
  |
  └─► 指揮，不是操作  ────────────────── 說目標，Agent 規劃步驟、執行、回報
        ├─► 規劃→行動→觀察  ────────── Agent 的工作循環，可持續數十輪
        └─► 人機分工  ────────────────── 人負責方向與判斷，Agent 負責執行
```

---

> 恭喜你，你已經準備好了在 AI 時代重新定義自己工作方式的完整框架。
