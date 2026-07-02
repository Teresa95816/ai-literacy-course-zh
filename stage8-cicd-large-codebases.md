# 第八階段, CI/CD 與大型專案整合

## 1. Claude Code GitHub Actions

Claude Code GitHub Actions 把 AI 驅動的自動化帶進 GitHub 工作流程, 只要在任何 PR 或 issue 中提及 `@claude`, Claude 就能分析程式碼、建立 pull request、實作功能、修復錯誤, 同時遵循專案標準.這個 action 建構於 Claude Agent SDK 之上.

### 為什麼使用

即時建立 PR（描述需求, Claude 建立包含所有必要變更的完整 PR）, 自動化程式碼實作（一個指令把 issue 轉成可運作的程式碼）, 遵循專案標準（遵守 CLAUDE.md 指引與既有程式碼模式）, 設定簡單（透過安裝程式與 API key 幾分鐘內開始）, 預設安全（程式碼留在 GitHub 的執行器上）.

### 快速設定

在 Claude Code 終端機中執行 `/install-github-app` 進行互動式設定, 此指令會在你的儲存庫安裝 Claude GitHub App, 然後引導你加入 GitHub Actions 工作流程與 API key 密鑰.需要儲存庫管理員權限.

### 手動設定

安裝 Claude GitHub App 到儲存庫（需要 Contents、Issues、Pull requests 的讀寫權限）, 把 `ANTHROPIC_API_KEY` 加入儲存庫密鑰, 從範例目錄複製工作流程檔案到 `.github/workflows/`.

### 基本工作流程範例

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

`prompt` 輸入欄位也接受 skill 呼叫, 例如安裝 code-review 外掛並在每次 PR 開啟或更新時執行其 skill.

### 最佳實務

在儲存庫根目錄建立 CLAUDE.md 定義程式碼風格指引、審核標準、專案專屬規則.安全性方面務必使用 GitHub Secrets 而非直接寫入 API key, 限制 action 權限僅限必要範圍, 合併前檢視 Claude 的建議.

### CI 成本

GitHub Actions 成本消耗你的 Actions 分鐘數, API 成本依提示詞與回應長度計算 token.優化建議, 使用具體的 @claude 指令減少不必要的 API 呼叫, 在 claude_args 中設定適當的 --max-turns 防止過度迭代, 設定工作流程層級的逾時.

### 搭配 Amazon Bedrock 與 Google Vertex AI

企業環境可用自己的雲端基礎架構, 需要設定 Workload Identity Federation（Vertex）或 GitHub OIDC Identity Provider（Bedrock）, 建議建立自訂 GitHub App 以獲得最佳控制與安全性.

## 2. Claude Code 與 GitHub Enterprise Server

GitHub Enterprise Server（GHES）支援讓組織可以在自架的 GitHub 執行個體上使用 Claude Code, 而非 github.com.Owner 連接一次 GHES 執行個體後, 開發者就能執行 web session、取得自動程式碼審核、並從內部 marketplace 安裝外掛, 不需要每個儲存庫個別設定.適用於 Team 與 Enterprise 方案.

### 支援情況

| 功能 | GHES 支援 | 備註 |
| :-- | :-- | :-- |
| Claude Code on the web | 支援 | Owner 連接一次, 開發者照常使用 claude --remote |
| Code Review | 支援 | 與 github.com 相同的自動化 PR 審核 |
| Teleport sessions | 支援 | 用 --teleport 在網頁與終端機間移動 session |
| Plugin marketplaces | 支援 | 使用完整 git URL 而非 owner/repo 簡寫 |
| GitHub MCP server | 不支援 | 該 MCP server 無法用於 GHES 執行個體 |

### 管理者設定

Owner 需要在 claude.ai/admin-settings/claude-code 開啟連接精靈, 輸入顯示名稱與 GHES 主機名稱, 點選繼續前往 GitHub Enterprise 建立 GitHub App, 接著在 GHES 執行個體上將 app 安裝到目標儲存庫或組織, 最後回到管理設定啟用 Code Review、Claude Security、貢獻指標.

### 開發者工作流程

Owner 完成連接後, 開發者不需要任何額外設定.正常從 GHES clone 儲存庫, Claude Code 會自動從 git remote 偵測 GHES 主機名稱, 執行 `claude --remote "..."` 即可啟動 web session, 在 GHES 上 clone 儲存庫並把變更推回分支.

### GHES 上的 plugin marketplace

`owner/repo` 簡寫只會解析到 github.com, GHES 託管的 marketplace 需使用完整 git URL, 例如 `/plugin marketplace add git@github.example.com:platform/claude-plugins.git`.

## 3. Claude Code GitLab CI/CD

此整合目前為 beta 階段, 由 GitLab 維護.建構於 Claude Code CLI 與 Agent SDK 之上, 可在 CI/CD 工作中程式化使用 Claude.

### 為什麼使用

即時建立 MR、自動化實作、專案感知（遵循 CLAUDE.md 指引）、設定簡單（加一個 job 到 .gitlab-ci.yml 加一個遮罩的 CI/CD 變數）、企業就緒（可選 Claude API、Amazon Bedrock、或 Google Vertex AI）.

### 運作方式

事件驅動編排（GitLab 監聽你選擇的觸發條件, 例如 issue、MR、或討論串中提及 @claude 的留言）, 供應商抽象化（依環境選擇 Claude API、Bedrock、或 Vertex AI）, 沙盒化執行（每次互動都在容器中執行, 有嚴格的網路與檔案系統規則, 每個變更都透過 MR 呈現）.

### 快速設定

在 Settings 到 CI/CD 到 Variables 加入遮罩的 `ANTHROPIC_API_KEY`, 並在 `.gitlab-ci.yml` 加入 Claude job,

```yaml
claude:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  script:
    - curl -fsSL https://claude.ai/install.sh | bash
    - claude -p "${AI_FLOW_INPUT:-'Review this MR'}" --permission-mode acceptEdits
```

### 安全性與治理

每個 job 在隔離容器中執行且網路存取受限, Claude 的變更都透過 MR 呈現讓審核者看到每一個差異, 分支保護與核准規則同樣適用於 AI 產生的程式碼.

## 4. 在 monorepo 或大型程式碼庫設定 Claude Code

隨著程式碼庫成長, 為小型專案調校的預設值可能會用與目前任務無關的指示與檔案讀取塞滿情境視窗, 浪費 token 並降低 Claude 的表現.這篇文件說明如何把 Claude 的範圍限縮到任務實際觸及的那部分程式碼.

### 選擇啟動 Claude 的位置

啟動 `claude` 的位置決定了 Claude 能讀寫哪些檔案、啟動時載入哪些 CLAUDE.md 檔案、以及套用哪些專案設定.

| 啟動位置 | 檔案存取範圍 | 啟動時載入的 CLAUDE.md |
| :-- | :-- | :-- |
| 儲存庫根目錄 | 每個檔案 | 只有根目錄, 子目錄檔案於讀取時按需載入 |
| 子目錄 | 僅該子樹, 除非額外授予 | 該目錄的加上每個上層目錄的 |

`.claude/settings.json` 中的專案設定只會從你的啟動目錄載入, 不會像 CLAUDE.md 一樣從父目錄繼承.

### 依目錄分層 CLAUDE.md

常見的分法是兩層, 根目錄 CLAUDE.md 涵蓋到處適用的指示（程式碼標準、commit 慣例、儲存庫佈局）, 每個子目錄的 CLAUDE.md 加上該區域技術堆疊專屬的慣例.在 monorepo 中每個套件一份, 在大型單一樹狀結構中每個子系統一份.把這些檔案提交到儲存庫讓團隊成員都能繼承.

### 排除不相關的 CLAUDE.md 檔案

`claudeMdExcludes` 設定可依路徑或 glob 模式跳過特定檔案, 適用於你從不涉足的目錄, 如其他團隊的套件、舊有程式碼、或引入的第三方子樹.

```json
{
  "claudeMdExcludes": [
    "**/packages/admin-dashboard/**",
    "**/packages/legacy-*/**"
  ]
}
```

### 減少 Claude 讀取的內容

在 `permissions.deny` 加入 Read 拒絕規則, 阻擋 Claude 開啟建置產物、產生的程式碼、引入的第三方相依套件, 即使搜尋結果列出這些檔案.安裝程式碼智慧外掛（如 typescript-lsp）可讓 Claude 透過語言伺服器直接跳到定義、尋找參照, 而非逐一掃描檔案樹.

### 限縮 worktree 與檔案存取範圍

`worktree.sparsePaths` 設定使用 git sparse-checkout, 只把列出的目錄與根層級檔案寫入磁碟, 讓 worktree 啟動更快、佔用空間更少.搭配 `symlinkDirectories` 可避免 node_modules 這類大型目錄在各 worktree 間重複.

```json
{
  "worktree": {
    "sparsePaths": [".claude", "packages/api", "packages/shared"],
    "symlinkDirectories": ["node_modules"]
  }
}
```

`additionalDirectories` 設定或 `--add-dir` 旗標可讓 Claude 存取工作目錄以外的目錄, 適合任務需要跨套件變更（例如更新 api 與 web 都會匯入的共用型別）時使用.

### 加入依目錄劃分的 skills

任何子目錄都可以定義限定於自己技術堆疊的 skills, 放在該目錄下的 `.claude/skills/`, 只有在 Claude 判定相關時才會按需載入, 因此 API 專屬的工具不會在前端工作時消耗情境.

### 當分層無法再擴展時集中化慣例

隨著程式碼庫成長, 依目錄分層的 CLAUDE.md 可能變得難以治理.把慣例與參考內容移出永遠載入的 CLAUDE.md, 改用按需載入的機制, skills（只在任務相關時載入的參考資料）, plugins（由平台團隊集中維護的版本化 skills、hooks、commands 套件）, MCP servers（若組織已有程式碼搜尋或 RAG 索引, 可包裝成 MCP 工具讓 Claude 查詢而非直接讀檔）.

## 5. Development containers 開發容器

開發容器（dev container）讓團隊中的每位工程師都能執行完全相同、隔離的環境.在容器中安裝 Claude Code 後, Claude 執行的指令會在容器內執行而非主機上, 而你對專案檔案的編輯會照常出現在本機儲存庫中.

警語, 使用 --dangerously-skip-permissions 時, 開發容器無法防止惡意專案外洩容器內可存取的任何內容, 包含存放在 ~/.claude 的 Claude Code 憑證, 只應在開發受信任的儲存庫時使用開發容器, 並監看 Claude 的活動, 避免把主機的密鑰（如 ~/.ssh）掛載進容器.

### 把 Claude Code 加進開發容器

透過 Claude Code Dev Container Feature 安裝, 在 `.devcontainer/devcontainer.json` 加入,

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/anthropics/devcontainer-features/claude-code:1.0": {}
  }
}
```

接著用 Dev Containers, Rebuild Container 重建容器, 在重建後的容器中開啟終端機執行 `claude` 並完成驗證流程.

### 跨重建保留驗證與設定

容器的家目錄預設在重建時會被捨棄, 因此工程師每次都得重新登入.Claude Code 把驗證 token、使用者設定、session 歷史存放在 `~/.claude`, 在該路徑掛載一個具名 volume 即可跨重建保留這些狀態.

```json
"mounts": [
  "source=claude-code-config,target=/home/node/.claude,type=volume"
]
```

### 強制執行組織政策

開發容器是套用組織政策的便利位置, 因為每位工程師的機器都執行相同的映像檔與設定.Claude Code 在 Linux 上會讀取 `/etc/claude-code/managed-settings.json` 並以最高優先權套用, 可從 Dockerfile 複製檔案到位.

### 限制網路對外連線

可把容器的對外流量限制在 Claude Code 需要的網域, 參考容器內含一個 init-firewall.sh 腳本, 封鎖除了必要網域以外的所有對外流量.

### 不使用權限提示執行

因為容器以非 root 使用者執行 Claude Code 且指令執行侷限於容器內, 可傳入 `--dangerously-skip-permissions` 做無人值守操作, 但這會移除你在工具呼叫執行前檢視的機會, 建議搭配上述的網路對外限制一起使用.
