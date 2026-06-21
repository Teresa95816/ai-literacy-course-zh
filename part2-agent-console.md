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

> 以前, 電腦是你的工具.
> 現在, 電腦是你的辦公室——而 AI Agent, 是在裡面替你工作的員工.

[LLM Agent Stack](llm-agent-stack.html)

---

### 2-1 Claude Code

如果 ChatGPT 是你的電話顧問——你打過去問, 它給建議, 但它看不到你桌上的東西、也不能幫你動手
那麼 Claude Code 就是坐在你旁邊的同事, 你的螢幕它看得到, 鍵盤它也能打.

> 一個讓 AI 真正在你的電腦上工作的工具

---

#### 動手做: Bookmark 三個供應商的 Chatbot

#### 動手做: 安裝 Claude Code

1. 開啟 [nodejs.org](https://nodejs.org), 下載並安裝 LTS 版
2. 開啟終端機, 確認 `node --version` 與 `npm --version` 都有輸出版本號
3. 執行 `npm install -g @anthropic-ai/claude-code`
4. 執行 `claude`, 依提示登入 Anthropic 帳號並授權

[Claude Code DOC Quick Start](https://code.claude.com/docs/zh-TW/quickstart)

> 如果 `node --version` 沒有輸出, 關掉終端機重新開啟再試一次.
> Mac 找不到 Claude 的路徑, 將終端機裡的訊息貼進 AI Chatbot 問 AI.

---

#### Human-in-the-Loop

Claude Code 的運作邏輯:

```
你的指令（Prompt）
    ↓
Claude Code 讀取當前專案的上下文（Context Window）
    ↓
呼叫工具（Tool Use）: 讀檔、搜尋、執行指令
    ↓
根據工具回傳結果, 決定下一步行動, 持續循環直到完成
    ↓
輸出結果: 修改後的檔案、執行報告、提問確認
```

> Claude Code 的設計哲學是「人始終在決策圈內」.
> 在做重大決策前會停下來確認, 你設定的信任邊界, 決定了它能自主行動的範圍.

---

#### Claude Code 能做什麼

| 能力 | 說明 |
|------|------|
| 讀取整個專案 | 掃描資料夾結構, 理解各檔案的關聯與脈絡 |
| 讀寫任意檔案 | 直接建立、修改、刪除工作目錄中的文件 |
| 執行系統指令 | 安裝工具、處理檔案、呼叫外部服務 |
| 搜尋整個資料夾 | 用關鍵字在所有文件中找到相關內容 |
| 多步驟任務規劃 | 把複雜任務拆解成子步驟, 逐步執行並回報進度 |
| Git 整合 | 讀取差異、撰寫 commit 訊息、管理分支 |

> Claude Code 的操作介面是終端機的命令列, 也可以在 VS Code 裡直接呼叫它.

---

### 2-2 Git

你有沒有這樣的資料夾:

```
報告_最終版.docx
報告_最終版_修改後.docx
報告_最終版_修改後_確認版.docx
報告_最終版_修改後_確認版_老闆改過.docx
```

> 只需要一份檔案, 卻能看到任何歷史版本、隨時回到過去任何一個時間點.

---

#### 動手做: 安裝 Git

| 平台 | 安裝方式 |
|------|---------|
| Windows | 開啟 [git-scm.com](https://git-scm.com) 下載安裝程式, 安裝過程中所有選項保持預設即可 |
| Mac | 在終端機執行 `git --version`, 若尚未安裝, 系統會自動提示安裝 Xcode Command Line Tools |

安裝完成後, 執行以下指令確認成功:

```bash
git --version
```

> 有輸出版本號（例如 `git version 2.x.x`）即代表安裝成功.

---

#### Git 和 GitHub 不是同一件事

| | Git | GitHub |
|---|---|---|
| 是什麼 | 版本控制工具 | 雲端存放平台 |
| 在哪裡 | 安裝在你的電腦 | 網站 github.com（微軟旗下）|
| 做什麼 | 在本機記錄每次變更 | 把記錄備份到雲端、分享給他人 |

> Git 是工具, GitHub 是平台; GitHub 只是讓 Git 的記錄「上雲」.

---

#### Repository

簡稱 repo, 是一個有完整歷史記錄的資料夾.

#### Commit

把當前的變更打包成一個有名字的歷史節點, 附上說明記錄「這次改了什麼、為什麼改」.

#### Remote

本機的 repo 只活在你的電腦裡. Remote 是在 GitHub 建立一份備份.

```
你的電腦（本機 repo）
    ^
    |  pull: 把雲端的最新變更拉回本機
    |  push: 把本機的新 commit 上傳到雲端
    v
遠端 repo（GitHub / GitLab）
    |  pull  ^  push
    v        | 
同事的電腦（本機 repo）
```

---

#### GitHub 能帶給你什麼

推上 GitHub, 你得到的遠不只是備份:

1. 雲端備份 — 電腦壞了, 工作還在
2. 跨裝置存取 — 新電腦用 `git clone` 複製整個 repo, 之後 `git pull` 拿最新版本
3. 連結即分享 — 把 repo 網址給同事, 他們立刻能看到你的工作
4. GitHub Pages — 把 Marp 匯出的 HTML 發布上線, 一個連結就能分享簡報
5. GitHub Actions — 固定時間自動執行腳本, 不需要開電腦

---

#### 動手做: 初次使用 Git

告訴 Git 你是誰 : 這個設定只需要做一次, 之後所有 repo 都會沿用.

```bash
git config --global user.name "Your Name"
```

```bash
git config --global user.email "your.email@example.com"
```

---

### 2-3 VS Code

#### 動手做: 安裝 VS Code 和 延伸套件

[code.visualstudio.com](https://code.visualstudio.com)

在 VS Code 中, 點選左側「Extensions」→ 搜尋套件名稱 → 點選 Install.

| 套件名稱 | 功能說明 |
|---------|--------|
| Chinese (Traditional) Language Pack | 將 VS Code 介面切換為繁體中文 |
| markdownlint | 自動檢查 Markdown 語法規範, 提示格式錯誤 |
| Markdown Preview Enhanced | 即時預覽 Markdown, 並可匯出整頁捲動式 HTML |
| Marp for VS Code | 將 Markdown 渲染成投影片, 支援即時預覽與匯出 |
| Material Icon Theme | 以直覺圖示區分不同類型的檔案與資料夾 |

---

#### Markdown 和 AI 說同一種語言

如果你的工作筆記本來就是 Markdown, 就等於你和 AI 之間不需要格式翻譯.

提示詞、輸出、儲存, 都在同一個格式生態裡流轉.

```markdown
# 這是標題

這是一段普通文字, *這是斜體*.

- 項目一
- 項目二

> 這是引言區塊
```

> 這份投影片本身, 就是用 Markdown 寫成的.

---

#### Marp（Markdown Presentation Ecosystem）

「這份投影片本身, 就是用 Markdown 寫成的.」更精確地說, 是用 Marp 製作的.

在 `.md` 檔案開頭加上 frontmatter 宣告, 三條橫線 `---` 就是換頁.

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

#### 動手做: VS code 基本操作

開啟資料夾, 建立新檔案

| Windows | Mac / Linux |
|---------|-------------|
| `C:\Users\<名稱>\.claude\` | `~/.claude/` |

```markdown
# 全域偏好設定

- 在空資料夾 session 中, 除非使用者明確要求, 否則只以模型知識為基礎協作, 不主動探索或讀取當前工作目錄以外的任何檔案或專案.
- 不主動觸發任何 custom skill.即使當下對話情境符合某個 skill 的描述, 也不得自行判斷並調用. Skill 的使用必須由使用者明確以 slash command 發起.

## Markdown 生成慣例

- 預設不使用粗體（`**text**`）. 若需強調, 優先改用標題層級、條列結構或程式碼區塊來呈現層次, 而非加粗文字.
- 預設不使用 emoji.除非使用者明確要求, 否則所有回應與檔案內容均不得包含表情符號.
- 預設不使用破折號（— 或 --）.若需連接、轉折或補充說明, 優先改用逗號、句號分句.
- 逗號、句號、分號、冒號統一使用半形 (, . ; :)
```

---

#### 動手做: 拿到一份你自己的副本

| | Fork | Clone |
|---|------|-------|
| 動作發生在 | GitHub 網站 | 你的電腦 |
| 做的事 | 把別人的 repo 複製一份到你自己的帳號 | 把一份 repo（自己的或別人的）下載到本機 |
| 結果 | 你的帳號下多了一份完全獨立的 repo | 本機資料夾裡有一份可以編輯的副本 |

1. 開啟 [本課程的 GitHub repo](https://github.com/AlleninTaipei/ai-literacy-course-zh), 點擊右上角的 Fork
2. Fork 完成後, 網址會變成 `github.com/你的帳號/ai-literacy-course-zh`, 這是你自己的副本
3. 點擊綠色的 Code 按鈕, 複製 HTTPS 網址, 在終端機執行:

```bash
git clone https://github.com/你的帳號/ai-literacy-course-zh.git
```

> Clone 下來的 origin, 指向的是你帳號下的副本, 不是原作者的 repo.
> 之後每一次 commit、push, 都只會進到你自己的 fork, 不會動到原始版本.

---

#### 動手做: markdown file 開啟預覽

| | Markdown Preview Enhanced | Marp for VS Code |
|---|---|---|
| 負責什麼 | 一般文件, 整頁捲動閱讀 | 有 frontmatter（`marp: true`）的投影片檔 |
| 預覽畫面 | 整頁捲動式 | 逐頁切換式 |

> 兩個套件都裝的情況下, 開啟一份有 `marp: true` frontmatter 的 `.md` 並點擊預覽, VS Code 預設仍會用 Markdown Preview Enhanced 開啟, 同時跳出提示, 問你要不要改用 Marp for VS Code 顯示.

---

#### 動手做: 完整工具鏈閉環

> 新資料夾 → AI 生成 → VS Code 編輯 → New Repo → Git 版控 → 雲端備份.

---

## 小結: Part 2 概念關係總覽

| 主題 | 是什麼 | 關鍵認識 |
|------|-------|---------|
| Claude Code | 在你電腦裡工作的 AI Agent | Human-in-the-Loop: 人始終在決策圈內 |
| Git | 版本控制（Repo / Commit / Remote）| 版本記錄讓每一次操作都可以回溯, push 讓工作不綁在單一台電腦上 |
| GitHub | 雲端存放平台 | 備份、跨裝置存取、連結分享、網頁發布、定時自動執行 |
| VS Code / Markdown / Marp | 工作環境與文件格式 | Agent 的操作空間, 人與 AI 共讀的原生格式 |

---

> 恭喜你. 你已經理解新的工作方式. 你安裝了工具、下了指令、把成果推上了 GitHub.
> 從這一刻起, AI 不再只是你聽說過的東西, 而是坐在你旁邊、等你開口的同事.

---

## 補充: 第一次 push 遇到 GitHub 驗證怎麼辦

| 平台 | 驗證機制 | 你需要做什麼 |
|------|---------|------------|
| Windows | Git Credential Manager（Git for Windows 內建）| 第一次 push 自動開啟瀏覽器 → 登入 GitHub → 授權完成, 憑證存入系統 |
| Mac | 需要先安裝 GitHub CLI | 在終端機執行 `brew install gh`, 再執行 `gh auth login`, 依提示在瀏覽器完成授權 |

```bash
git remote add origin https://github.com/你的帳號/repo名稱.git
```

```bash
git push -u origin main
```

`-u` 的作用是「記住這條上傳路徑」, 之後直接 `git push` 就能推到同一個地方.

Mac 用戶: `brew` 是 macOS 的套件管理工具（Homebrew）, 如果還沒安裝, 先前往 [brew.sh](https://brew.sh) 安裝, 再執行上述指令.
