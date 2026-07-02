# 好好用 AI

![banner](assets/banner.svg)

![Made with Marp](https://img.shields.io/badge/Made%20with-Marp-1a1a1a?style=flat&logo=markdown&logoColor=D97757&labelColor=1a1a1a&color=D97757)
![Made with Markdown Preview Enhanced](https://img.shields.io/badge/Made%20with-Markdown%20Preview%20Enhanced-1a1a1a?style=flat&logo=markdown&logoColor=D97757&labelColor=1a1a1a&color=D97757)
![Language](https://img.shields.io/badge/語言-繁體中文-D97757?style=flat&labelColor=1a1a1a)
![License](https://img.shields.io/badge/License-CC%20BY--SA%204.0-D97757?style=flat&labelColor=1a1a1a)
![Last Commit](https://img.shields.io/github/last-commit/AlleninTaipei/ai-literacy-course-zh?style=flat&labelColor=1a1a1a&color=D97757)

**帶你一步步上手 AI 的三部曲課程**

這套課程以 Markdown 撰寫，目標是讓每一位每天在電腦前工作的人，都能真正把 AI 當成自己的工作夥伴——不需要寫程式，不需要技術背景。

---

## 課程結構

| Part | 標題 | 核心主題 | 檔案 |
|------|------|----------|------|
| **Part 1** | 認識你的工作夥伴 | AI 基礎概念 | `part1-ai-literacy-intro.md` |
| **Part 2** | 個人電腦新情境 | 從文書處理機到 Agent 控制台 | `part2-agent-console.md` |
| **Part 3** | 啟動 JARVIS 模式 | 指揮 AI Agent | `part3-jarvis-mode.md` |

---

## 其他教材

### Part 2 工具入門指南

| 檔案 | 說明 |
|------|------|
| `vscode-guide.md` | 給職場工作者的 VS Code 入門指南: 介面總覽、擴充套件、內建 Git 整合、Markdown 即時預覽 |
| `markdown-guide.md` | 給職場工作者的 Markdown 入門指南: 基礎語法、常見應用情境、常見錯誤提醒 |
| `marp-guide.md` | 給職場工作者的 Marp 入門指南: front matter、投影片分隔、版面切版、匯出格式 |
| `git-github-guide.md` | 給職場工作者的 Git & GitHub 入門指南: 基本概念、常用指令、GitHub CLI、認證方式 |
| `claude-code-guide.md` | 給職場工作者的 Claude Code 入門指南: 與聊天機器人的差異、安全機制、辦公應用情境 |

---

## 技術規格

- 簡報格式: [Marp](https://marp.app/)(Markdown Presentation Ecosystem)
- 瀏覽格式: Part 1～3、`ai-policy`、`anthropic-academy` 已轉換為互動學習頁面(分頁導覽、深色 / 亮色切換), 由 Claude Code 透過 `/md2course` 指令依 Markdown 內容生成; 其餘教材仍由 [Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced) 輸出整頁捲動 HTML
- 配色方案: 深色主題, 背景 `#1A1A1A`, 強調色 `#D97757`
- 互動展示: 純 HTML, 無需伺服器, 直接瀏覽器開啟

### HTML 命名與輸出規則

每份教材的 HTML 分成兩種模式, 依檔名規則對應不同的產生工具, `index.html` 會依使用者選擇的模式自動連到對應檔案:

| 命名規則 | 模式 | 產生工具 |
|------|------|------|
| `*-slide.html` | 簡報模式(逐頁切換) | [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) 匯出 HTML, 匯出後手動更名加上 `-slide` 後綴 |
| `*.html`(不帶後綴) | 瀏覽模式(整頁捲動 / 互動學習頁面) | Part 1～3、`ai-policy`、`anthropic-academy`: Claude Code 透過 `/md2course` 轉換為互動學習頁面; 或使用 [Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced) 輸出 HTML |

### 瀏覽模式連結自動修正

Markdown Preview Enhanced 輸出瀏覽模式 HTML 時, 偶爾會把同層檔案的相對連結展開成本機絕對路徑（例如 `file:///c:\...\demo.html`）, 這種連結一旦推上 GitHub Pages 就會失效.

`.githooks/pre-commit` 會在每次 commit 前自動掃描即將提交的 HTML, 找到這種絕對路徑連結就改回相對連結並重新加入這次 commit, 腳本本身發生任何錯誤都一律放行, 不會卡住 commit.

這個 hook 需要在本機 repo 執行過 `git config core.hooksPath .githooks` 才會生效, 此設定存在 `.git/config`, 不會隨 clone 或 fork 傳播, 對單純練習 Git 操作的學員沒有影響.

---

## 對象

這套課程為企業內訓設計，適合：

- 每天用電腦工作, 但從未寫過程式的人
- 希望將 AI 導入日常工作流程的團隊
- 想了解 AI Agent 實際運作方式的主管或同仁

## Ending

《倚天屠龍記》中, 張無忌初學太極時, 張三豐告訴他: 「忘掉招式.」

真正的太極, 不是記住每一招、每一式, 而是在千錘百鍊之後, 將所有招式內化為本能; 不拘泥於形式, 不執著於套路, 因時制宜, 隨勢而動, 以柔克剛.

AI 也是如此.

今天課堂上介紹的, 甚至未來還會出現更多新的工具與名詞, 它們都只是「招式」.

真正重要的, 不是背會每一個工具, 不是記住每一個 Prompt, 而是培養一種能力:

當面對新的工作、新的問題、新的 AI 時, 你能自然地思考:

- 我真正要解決的是什麼問題？
- 哪些事情應該交給 AI？
- 哪些判斷必須由人負責？
- 如何驗證 AI 的結果？
- 如何讓 AI 成為自己的能力延伸, 而不是思考的替代品？

當這些思考已經成為你的習慣, 你就不再需要依賴任何一套固定的方法.

因為你學會的, 不是 AI 的招式, 而是 AI 的心法. 具備駕馭未來任何 AI 的能力.