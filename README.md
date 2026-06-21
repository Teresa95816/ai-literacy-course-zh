# AI 素質養成課程

![banner](assets/banner.svg)

![Made with Marp](https://img.shields.io/badge/Made%20with-Marp-1a1a1a?style=flat&logo=markdown&logoColor=D97757&labelColor=1a1a1a&color=D97757)
![Made with Markdown Preview Enhanced](https://img.shields.io/badge/Made%20with-Markdown%20Preview%20Enhanced-1a1a1a?style=flat&logo=markdown&logoColor=D97757&labelColor=1a1a1a&color=D97757)
![Language](https://img.shields.io/badge/語言-繁體中文-D97757?style=flat&labelColor=1a1a1a)
![License](https://img.shields.io/badge/License-CC%20BY--SA%204.0-D97757?style=flat&labelColor=1a1a1a)
![Last Commit](https://img.shields.io/github/last-commit/AlleninTaipei/ai-literacy-course-zh?style=flat&labelColor=1a1a1a&color=D97757)

**為白領上班族設計的三部曲 AI 實戰課程**

這套課程以 Markdown 撰寫，並透過 Marp 與 Markdown Preview Enhanced 分別輸出簡報模式與瀏覽模式的 HTML，目標是讓每一位每天在電腦前工作的人，都能真正把 AI 當成自己的工作夥伴——不需要寫程式，不需要技術背景。

---

## 課程結構

| Part | 標題 | 核心主題 | 檔案 |
|------|------|----------|------|
| **Part 1** | 認識你的工作夥伴 | AI 素質養成基礎概念 | `part1-ai-literacy-intro.md` |
| **Part 2** | 個人電腦新情境 | 從文書處理機到 Agent 控制台 | `part2-agent-console.md` |
| **Part 3** | 啟動 JARVIS 模式 | 用 Claude Code 指揮真實工作 | `part3-jarvis-mode.md` |

### Part 1 — 認識你的工作夥伴

AI 基礎概念的白話版講解：神經網路、預訓練、對齊、微調、推論、MoE、Token、無狀態與多輪對話、系統提示、工具呼叫、多模態。

### Part 2 — 個人電腦新情境

建立現代 AI 工作環境：VS Code、Markdown、Marp、Claude Code、Git、Agentic Coding。

### Part 3 — 啟動 JARVIS 模式

真實工作場景的 AI Agent 執行實作，學會「指揮」而不只是「使用」AI。

---

## 其他教材

| 檔案 | 說明 |
|------|------|
| `ai-course-intro.md` | 課程開場白：給所有參與者的一封信 |
| `cpn-learning-path.md` | Claude Partner Network Learning Path：四門官方課程與合作夥伴認證指引（部分課程需程式基礎，適合技術人員） |
| `claude-code-agents.md` | Claude Code `←` 多 Agent 並行管理說明 |
| `claude-code-git-setup.md` | Claude Code 搭配 Git 的環境設定參考 |
| `license-notes.md` | CC BY-SA 4.0 授權條款的白話說明：保留與開放的權利、著作權歸屬、版本比較 |

---

## 技術規格

- **簡報格式**：[Marp](https://marp.app/)（Markdown Presentation Ecosystem）
- **瀏覽格式**：[Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced)（輸出整頁捲動 HTML）
- **配色方案**：深色主題，背景 `#1A1A1A`，強調色 `#D97757`
- **互動展示**：純 HTML，無需伺服器，直接瀏覽器開啟

### HTML 命名與輸出規則

每份教材的 HTML 分成兩種模式，依檔名規則對應不同的產生工具，`index.html` 會依使用者選擇的模式自動連到對應檔案：

| 命名規則 | 模式 | 產生工具 |
|------|------|------|
| `*-slide.html` | 簡報模式（逐頁切換） | [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) 匯出 HTML，匯出後手動更名加上 `-slide` 後綴 |
| `*.html`（不帶後綴） | 瀏覽模式（整頁捲動） | [Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced) 輸出 HTML |

---

## 對象

這套課程為企業內訓設計，適合：

- 完全不懂程式的白領工作者
- 希望將 AI 導入日常工作流程的團隊
- 想了解 AI Agent 實際運作方式的主管或同仁
