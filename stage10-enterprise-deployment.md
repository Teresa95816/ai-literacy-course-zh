# 第十階段, 企業級部署與治理

## 1. Set up Claude Code for your organization 組織部署決策地圖

Claude Code 透過 managed settings 強制執行組織政策, 優先於本機開發者設定, 可由 admin console, MDM, 或磁碟上的檔案送達.部署決策依序為, 選擇 API 供應商（Claude for Teams/Enterprise 為預設建議, 也可用 Console, Bedrock, Vertex AI, Foundry）, 決定設定如何送達裝置, 決定要強制哪些控制（權限規則, sandbox, MCP 限制, plugin marketplace 限制, hooks 限制, 模型限制, 版本下限）, 建立使用量可見性（OpenTelemetry, Analytics dashboard, 成本追蹤）, 檢視資料處理政策（Team/Enterprise/API/雲端供應商方案不會用你的程式碼訓練模型）.

部署完成後, 讓開發者執行 `/status` 檢視 Setting sources 是否顯示 Enterprise managed settings 及來源（remote, plist, HKLM, HKCU, file）.

## 2. Feature availability 功能可用性比較

比較 Claude Code 功能在 Claude 訂閱, Anthropic Console, Amazon Bedrock, Claude Platform on AWS, Google Vertex AI, Microsoft Foundry 之間的差異.CLI 與 Agent SDK 本機執行的功能在所有供應商上一致.需要 claude.ai 帳號才能使用的功能包含 Claude Code on the web, Routines, Ultraplan, Ultrareview, Code Review, Remote Control, Chrome 擴充功能, Artifacts, 語音輸入等, 無法透過 Console API key 或第三方供應商單獨取得.依供應商而異的 CLI 能力包含 web search（Bedrock 不支援）, fast mode（僅 Claude 訂閱與 Console）, auto mode（第三方供應商需設定 `CLAUDE_CODE_ENABLE_AUTO_MODE`）, GitHub Actions 與 GitLab CI/CD（Foundry 不支援）.Server-managed settings 與 Analytics dashboard 僅 Anthropic 直接方案提供.

## 3. Configure auto mode 設定 auto mode 分類器

auto mode 讓 Claude Code 略過例行權限提示, 改用分類器審查每次工具呼叫, 封鎖不可逆, 破壞性, 或指向環境外的動作.用 `autoMode.environment` 告知分類器哪些 repo, bucket, 網域屬於組織信任範圍, 預設只信任工作目錄與目前 repo 的遠端.環境項目分為信任槽（trusted repo, source control, 內部網域, 雲端 bucket, 內部服務, 套件庫）與敏感度槽（PII 位置, 敏感遠端目標, 受保護 IaC 範圍）.

三個欄位可覆寫分類器規則, `hard_deny`（無條件安全邊界）, `soft_deny`（使用者意圖可解除的破壞性動作）, `allow`（soft_deny 的例外）, 優先順序為 hard_deny 高於使用者意圖, soft_deny 次之可被 allow 或明確意圖覆寫.加入字串 `"$defaults"` 可在保留內建規則的同時加入自訂規則, 省略則整組取代預設清單.`autoMode.classifyAllShell` 設為 true 時, 所有 Bash/PowerShell 允許規則都會被分類器攔截而非直接放行.三個 CLI 子指令可檢視設定, `claude auto-mode defaults`, `claude auto-mode config`, `claude auto-mode critique`.

## 4. Control MCP server access for your organization 控管 MCP server 存取

管理者可限制組織內可連接哪些 MCP server, 從固定部署一組伺服器到完全停用 MCP.`managed-mcp.json` 部署一組固定伺服器, 部署後使用者無法新增或使用任何其他伺服器（包含 plugin 提供的）, 空的 server map 可完全停用 MCP.此檔案是獨立檔案, 無法透過 server-managed settings 送達, 需經 MDM 或裝置管理工具部署到系統路徑.

`allowedMcpServers` 與 `deniedMcpServers` 則是政策式的允許 / 拒絕清單, 可依 `serverUrl`（支援萬用字元）, `serverCommand`（精確比對）, `serverName`（僅精確比對, 非安全控制因為名稱可被使用者任意命名）過濾.設定 `allowManagedMcpServersOnly: true` 讓允許清單只採用 managed settings 來源, 拒絕清單則永遠合併所有來源.伺服器評估順序為先合併清單, 再檢查拒絕清單（優先於一切）, 最後檢查允許清單.

## 5. Configure server-managed settings 伺服器管理設定

Server-managed settings 讓組織 Owner 從 claude.ai 主控台 Admin Settings > Claude Code > Managed settings 集中設定 Claude Code, 不需裝置管理基礎架構.客戶端在使用者以組織 OAuth 或直接設定的 API key 驗證時自動抓取設定, 每小時輪詢更新一次.需要 Claude for Teams 或 Enterprise 方案, Owner 或 Primary Owner 角色, 以及 `api.anthropic.com` 的網路存取.

設定內容支援 `settings.json` 幾乎所有欄位（除少數僅限 OS 層級政策傳遞的例外）, 包含 hooks, 環境變數, `allowManagedPermissionRulesOnly` 等 managed-only 設定.涉及 shell 指令, 自訂環境變數, hooks, 或 CLAUDE.md 內容的設定會觸發安全核准對話框要求使用者同意.Server-managed settings 不適用於 Bedrock, Vertex AI, Foundry, Claude Platform on AWS, 或自訂 `ANTHROPIC_BASE_URL`, 這些情境改用自架的 Claude apps gateway 提供對等的遠端設定送達.

## 6. Enterprise network configuration 企業網路設定

Claude Code 支援標準代理伺服器環境變數 `HTTPS_PROXY`, `HTTP_PROXY`, `NO_PROXY`（不支援 SOCKS 代理）, 代理若需基本驗證可在 URL 中內嵌帳密.預設信任內建 Mozilla CA 憑證與作業系統憑證存放區, `CLAUDE_CODE_CERT_STORE` 可設為 `bundled`, `system`, 或兩者逗號分隔的組合.自訂 CA 用 `NODE_EXTRA_CA_CERTS`, mTLS 驗證用 `CLAUDE_CODE_CLIENT_CERT`, `CLAUDE_CODE_CLIENT_KEY`, `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE`.

網路存取需求涵蓋 `api.anthropic.com`（API 請求）, `claude.ai`（帳號驗證）, `platform.claude.com`（Console 驗證）, `downloads.claude.ai`（外掛與自動更新下載）, `bridge.claudeusercontent.com`（Chrome 擴充功能）, `raw.githubusercontent.com`（changelog）.使用 Bedrock, Vertex AI, Foundry, 或已登入的 Claude apps gateway 時, 模型流量改走該供應商或 gateway, 但 WebFetch 工具仍會呼叫 `api.anthropic.com` 做網域安全檢查, 除非設定 `skipWebFetchPreflight: true`.

## 7. Choose a sandbox environment 選擇沙盒環境

隔離 Claude Code 限制 session 能讀寫與連線的範圍, 尤其在減少權限提示, 無人值守執行, 或處理不完全信任的程式碼時最重要.比較六種方式, sandboxed Bash tool（只隔離 Bash 指令, 免安裝 Docker）, sandbox runtime（隔離整個 Claude Code 行程, 包含檔案工具, MCP, hooks, 不需 Docker, 為 beta 研究預覽）, dev container（完整開發環境, 需 Docker）, custom container（完整開發環境, 中高設定成本）, virtual machine（整個作業系統層級隔離, 最強但設定成本最高）, Claude Code on the web（Anthropic 代管的隔離虛擬機, 需訂閱與 GitHub 帳號）.

`--dangerously-skip-permissions` 移除逐項核准, 因此務必搭配容器, VM, 或 sandbox runtime 執行以確保檔案工具, MCP, hooks 也在邊界內.auto mode 的分類器是逐動作控制而非隔離邊界, 對無人值守執行仍建議加上隔離邊界做縱深防禦.組織層級只有內建 Bash sandbox 能由 Claude Code 自身強制執行（透過 managed settings 送達 `sandbox` 設定鍵）, 其餘方式需搭配裝置管理或軟體允許清單工具強制.

## 8. Configure the sandboxed Bash tool 設定沙盒化 Bash 工具

Bash sandbox 讓 Claude 執行多數 shell 指令而不需逐一詢問, 改由你定義哪些檔案與網域可被觸及, 作業系統負責強制邊界.macOS 用內建 Seatbelt, Linux 與 WSL2 需安裝 `bubblewrap` 與 `socat`.用 `/sandbox` 開啟面板, 三個分頁為 Mode（auto-allow 或一般權限）, Overrides（是否允許失敗指令退回非沙盒執行, 即 `allowUnsandboxedCommands`）, Config（檢視解析後設定）.

預設沙盒指令只能寫入工作目錄與 session 暫存目錄, 讀取則除特定拒絕目錄外開放全機, 這代表預設仍可讀取 `~/.aws/credentials` 等憑證檔案, 需用 `sandbox.credentials` 明確封鎖.網路存取無預先允許網域, 第一次需要新網域時會提示核准.`sandbox.filesystem.allowWrite/denyWrite/denyRead/allowRead` 可調整檔案系統邊界, 路徑合併自所有設定範圍.組織可透過 managed settings 送達 `sandbox.enabled`, `failIfUnavailable`, `allowUnsandboxedCommands: false` 強制每位開發者使用沙盒, `allowManagedReadPathsOnly` 與 `allowManagedDomainsOnly` 可鎖定讀取路徑與網域只採用 managed 來源.

## 9. Run Claude Code through a gateway gateway 總覽

gateway 是組織自架, 位於 Claude Code 與模型供應商之間的代理, Claude Code 把 API 流量送到 gateway 而非直接送到供應商, 由 gateway 用組織持有的憑證轉發.開發者以 gateway 核發的憑證驗證, 因此驗證, 用量追蹤, 預算, 稽核紀錄都集中在一處.Claude Code 內建自架 gateway, Claude apps gateway, 也支援組織已在使用的既有 LLM gateway.

透過 gateway 連線且設定 gateway 憑證時, 用量以 API 費率計入組織的供應商帳戶, claude.ai 訂閱不會被使用或計費.只設定 `ANTHROPIC_BASE_URL` 而未設定 gateway 憑證時, 已儲存的 claude.ai 登入仍是有效憑證, 訂閱額度與計費照常適用.

## 10. Other LLM gateways 使用既有 LLM gateway

任何公開受支援 API 格式的 gateway 都能搭配 Claude Code, Anthropic 不背書, 維護, 或稽核第三方 gateway 產品, 也不支援透過任何 gateway 路由到非 Claude 模型.gateway 提供憑證集中管理, 用量追蹤, 成本控管, 稽核紀錄, 供應商切換（若 gateway 對外統一提供 Anthropic 格式端點）.

推出步驟固定為四步, 部署 gateway 並給予供應商憑證, 核發每位開發者的 gateway 憑證, 透過 managed settings 檔與密鑰工具分發設定, 讓每位開發者確認設定已生效.gateway 產品需要隨 Claude Code 演進持續更新轉發規則, 否則新版本引入的 `anthropic-beta` 值與請求欄位會被裁切導致功能失效.

## 11. Connect Claude Code to an LLM gateway 開發者連接 gateway

個別開發者確認組織是否已透過 managed settings 分發 gateway 設定, 方法是啟動 `claude` 觀察是否直接進入 session（略過登入畫面）, 並用 `/status` 檢查 Anthropic base URL 與 Auth token 欄位.若尚未分發, 需自行設定憑證變數（`ANTHROPIC_AUTH_TOKEN` 用於 bearer token, `ANTHROPIC_API_KEY` 用於 x-api-key）與 `ANTHROPIC_BASE_URL`, 可寫入 shell 或 settings 檔的 `env` 區塊.

驗證連線可先用 curl 直接送出一個 token 的請求確認 URL 與憑證可用.VS Code 擴充功能需在 `claudeCode.environmentVariables` 另外設定, 因為擴充功能自身的登入檢查不會讀取 `~/.claude/settings.json`.GitHub Actions, Agent SDK, Slack, 網頁版各自有不同的設定路徑與限制, 例如 Claude Code on the web 與 Slack 一律使用 Anthropic API, 不受 gateway 設定影響.

## 12. Gateway protocol reference gateway 協定參考

這是 gateway 維運者需實作的 API 合約.gateway 必須公開至少一種 API 格式, Anthropic Messages（`/v1/messages`）, Bedrock InvokeModel, 或 Agent Platform rawPredict, 且推論回應必須支援串流, 不可整批緩衝後才轉發.請求標頭中 `anthropic-version` 與 `anthropic-beta` 必須原封不動轉發（視為開放清單而非白名單, 因為 Claude Code 每次發版都可能新增能力值）, `x-claude-code-session-id` 等標頭則可供 gateway 自行消費做路由與歸因.

功能是否正常取決於標頭與請求本文欄位是否成對轉發, 例如 context management, structured outputs 等 beta 能力若標頭與欄位其中之一被裁切, 通常會產生硬性 400 錯誤而非悄悄停用.自動重試邏輯依上游錯誤文字比對, 因此 gateway 不應把錯誤包進自己的信封格式.`CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` 可讓 Claude Code 於啟動時查詢 gateway 的 `/v1/models` 並將結果加入 `/model` 選單.

## 13. Roll out an LLM gateway for your organization 管理者推出 gateway

管理者推出 gateway 給組織的完整檢查清單, 確認 gateway 正確路由每個模型名稱, 核發每位開發者的憑證並逐一驗證, 用即將分發的相同設定親自測試 Claude Code, 透過 managed settings 或直接告知開發者分發設定, 最後從開發者機器（而非 gateway 主機）驗證串流請求, 模型路由, `/status` 顯示的 gateway 位址皆正確.

推出後仍需長期維護, Claude Code 新版本會新增 beta 標頭與請求欄位, 新模型上線需更新 gateway 路由設定, 憑證需依排程輪替.設定每個金鑰的速率限制時應考慮 client 對暫時性失敗（含 429）最多重試 10 次並遵循 `Retry-After` 的行為.

## 14. Claude apps gateway 總覽

Claude apps gateway 是 Anthropic 自製, 內建於 `claude` 執行檔中的自架 gateway, 讓開發者以企業身分識別提供者（IdP）登入, 而非持有 API key 或雲端憑證.以 `claude gateway --config gateway.yaml` 啟動, 底層路由到 Amazon Bedrock, Google Cloud, Microsoft Foundry, 或 Anthropic API, 依 IdP 群組強制模型存取與 managed settings 政策, 並將 OTLP 遙測資料送到組織自有的可觀測性堆疊.

適用於必須將推論流量留在自有雲端供應商內（如資料落地需求）的組織.快速開始需要 Claude Code v2.1.195 以上, 一個 OIDC 相容的身分供應商, PostgreSQL 14 以上, 模型上游憑證, 以及可透過 HTTPS 存取的私有網路位址（gateway 必須部署在私有網路, 這是安全防護機制, 因為受信任的 gateway 可推送會在開發者機器上執行指令的設定）.已知限制包含不支援伺服器端 web search, 不支援 1 小時快取 TTL, 不支援 SAML/LDAP, 一個 gateway 只服務一個 OIDC 發行者, 不支援 Windows 伺服器.

## 15. Claude apps gateway configuration gateway.yaml 設定參考

gateway.yaml 是單一設定檔, 定義 gateway 的一切行為, 於啟動時讀取一次並依 schema 驗證.五個必要區塊為, `listen`（監聽位址, 對外 URL, TLS 終止）, `oidc`（身分供應商設定）, `session`（gateway 核發的 bearer token 簽章金鑰與存活時間）, `store`（PostgreSQL 連線, 用於裝置授權流程與速率限制計數器）, `upstreams`（模型上游, 支援 Bedrock, Agent Platform, Foundry, Anthropic API 並可設定容錯移轉）.

選用區塊包含 `models`（依上游轉譯模型名稱, 多上游容錯移轉, 供應商產能 ARN）, `managed`（依 IdP 群組配置 RBAC 與 managed settings 政策）, `telemetry`（OTLP 遙測扇出到多個目的地, 依目的地選擇是否含 logs 與 traces）, `admin`（花費上限管理 API 的驗證金鑰與稽核保留期）.

## 16. Claude apps gateway deployment and operations 部署與維運

生產部署四步驟, 在 IdP 註冊 OAuth 用戶端（單一重新導向 URI `https://<gateway>/oauth/callback`）, 建置容器映像並部署到 Kubernetes 或 Cloud Run（gateway 是無狀態單一 Linux 二進位檔, 可水平擴展, Postgres 為共享協調層）, 建立日誌, 健康探測, 異常行為, 密鑰輪替, 升級流程, 檢視安全態勢.

日誌分兩串流, 稽核事件（單行 JSON, 涵蓋 `session.mint`, `auth.denied`, `inference` 等安全相關事件）與可讀的操作日誌.健康檢查提供 `GET /healthz`（存活探測）與 `GET /readyz`（就緒探測, 驗證 store 可達）.JWT 簽章金鑰輪替採三步驟（新增金鑰到陣列前端, 滾動部署, TTL 後移除舊金鑰）以確保既有 session 不中斷.升級因 replica 無狀態而可安全滾動重啟, migrations 只增不減, 因此回退到舊版本二進位檔也是安全的.

## 17. Deploy Claude apps gateway on Google Cloud GCP 部署範例

在 Google Cloud 上執行 Claude apps gateway 的具體範例, 以 Agent Platform 作為模型上游, Google Workspace 作為範例 IdP.provisioning 涵蓋 Cloud Run 或 GKE 執行 gateway 容器, Artifact Registry 存放映像, Cloud SQL for PostgreSQL（僅私有 IP）作為 store, Secret Manager 存放 `gateway.yaml`, JWT 簽章金鑰, OIDC client secret, Postgres URL, 服務帳戶被授予 `roles/aiplatform.user` 並透過 Workload Identity 綁定.

Cloud Run 路徑需設定 `--no-invoker-iam-check` 或 `--allow-unauthenticated` 讓未經 Cloud Run IAM 驗證的請求進入（gateway 自身跑 OIDC 驗證）, 並需要內部應用程式負載平衡器或私有可解析網域滿足 `/login` 的私有網路檢查.GKE 路徑需在同一 VPC 上建立叢集以連到 Cloud SQL 私有 IP, 並啟用 Workload Identity 讓 Pod 繼承服務帳戶憑證.

## 18. Claude apps gateway spend limits 花費上限

花費上限限制每位開發者透過 Claude apps gateway 花費的金額, 依日, 週, 月為週期.因為 gateway 用單一共用上游憑證轉發所有推論, 供應商帳單會把一切歸到同一憑證, 若無個別上限, 一個失控的代理群可能耗盡組織整個額度.透過 `admin` 區塊設定後, gateway 在 `/v1/organizations/spend_limits` 提供管理 API, 每次 `POST` 建立或取代一個 `{scope, amount, period}` 上限.

`scope.type` 可為 `user`（依 OIDC sub）, `rbac_group`（依 IdP 群組）, `organization`（組織預設）, 群組或組織上限是每位成員各自套用的預設值而非共享額度.超過任一週期上限時, 該開發者下一次請求會收到 `429` 並附 `billing_error`.用量以串流回應中的 token 數乘以 USD 牌價估算, 屬於斷路器而非正式發票, 正式計費仍應與供應商自身用量報告核對.

## 19. Claude Code on Amazon Bedrock 透過 Bedrock 設定

登入精靈可引導完成 Bedrock 設定, 執行 `claude` 於登入畫面選擇 3rd-party platform 再選 Amazon Bedrock, 精靈會偵測憑證, 區域, 並確認帳號可呼叫哪些模型.手動設定則需先在 Bedrock 主控台的 Model catalog 送出模型存取申請表（每個 AWS 帳號一次）, 設定 AWS 憑證（CLI, 環境變數存取金鑰, SSO profile, 或 Bedrock API key）, 再設定 `CLAUDE_CODE_USE_BEDROCK=1` 與 `AWS_REGION`.

部署給多人時務必釘選模型版本（`ANTHROPIC_DEFAULT_OPUS_MODEL` 等）, 否則模型別名會解析到 Claude Code 內建預設值, 可能落後最新發布.IAM 政策需要 `bedrock:InvokeModel`, `bedrock:InvokeModelWithResponseStream`, `bedrock:ListInferenceProfiles`, `bedrock:GetInferenceProfile` 等權限.Mantle 是另一種以原生 Anthropic API 格式（而非 Bedrock Invoke API）服務 Claude 模型的 Bedrock 端點, 用 `CLAUDE_CODE_USE_MANTLE=1` 啟用, Sonnet 5 一律透過 Mantle 以 1M 情境視窗執行.

## 20. Claude Code on Google Vertex AI 透過 Vertex AI 設定

登入精靈（需 v2.1.98 以上）在 `claude` 登入畫面選擇 3rd-party platform 再選 Google Vertex AI, 引導完成 Application Default Credentials, service account 金鑰檔, 或環境變數驗證, 並確認專案可呼叫的模型.手動設定需先在 GCP 專案啟用 Vertex AI API, 於 Vertex AI Model Garden 申請 Claude 模型存取（約需 24 至 48 小時核准）, 再設定 `CLAUDE_CODE_USE_VERTEX=1`, `CLOUD_ML_REGION`, `ANTHROPIC_VERTEX_PROJECT_ID`.

`CLOUD_ML_REGION` 支援 global, 多區域（如 `eu`, `us`）, 或特定區域端點, 特定模型的可用性因端點類型而異, 可用 `VERTEX_REGION_<MODEL_NAME>` 個別覆寫.部署給多人時同樣需釘選模型版本.IAM 只需要 `roles/aiplatform.user` 角色所含的 `aiplatform.endpoints.predict` 權限.MCP tool search 在 Vertex AI 上預設停用（僅 Sonnet 4.5 以上與 Opus 4.5 以上支援, 需另設 `ENABLE_TOOL_SEARCH=true`）.

## 21. Claude Code on Microsoft Foundry 透過 Foundry 設定

Foundry 沒有互動式設定精靈, 純以環境變數設定.先在 Microsoft Foundry 主控台建立 Claude 資源與模型部署（Opus, Sonnet, Haiku）, 驗證支援 API key（`ANTHROPIC_FOUNDRY_API_KEY`）或 Microsoft Entra ID（Azure SDK 預設憑證鏈, 本機常用 `az login`）兩種方式.再設定 `CLAUDE_CODE_USE_FOUNDRY=1` 與 `ANTHROPIC_FOUNDRY_RESOURCE`（或直接提供完整 `ANTHROPIC_FOUNDRY_BASE_URL`）.

Foundry 沒有啟動時模型檢查機制, 因此模型版本未釘選且預設不可用時請求會直接失敗（而非像 Bedrock, Vertex AI 那樣自動回退舊版本）, 部署時務必釘選 `ANTHROPIC_DEFAULT_OPUS_MODEL` 等變數並在建立 Azure 部署時選擇特定模型版本而非自動更新到最新.RBAC 需要 `Azure AI User` 與 `Cognitive Services User` 角色所含的權限.

## 22. Claude Code on Claude Platform on AWS 透過 Claude Platform on AWS 設定

Claude Platform on AWS 是 Anthropic 直接維運的 Claude API, 搭配 AWS 驗證, IAM 存取控制, AWS Marketplace 計費, 請求直達 Anthropic API, 因此功能與發布節奏與 Claude API 相同, 但透過 AWS 憑證或 workspace API key 驗證並經 AWS Marketplace 付費.透過 AWS Marketplace 訂閱會建立一個綁定 AWS 帳號的全新 Anthropic 組織, 與既有組織的憑證不互通.

驗證方式分兩種, AWS 憑證搭配 SigV4（標準 AWS 憑證鏈）或 workspace API key（`ANTHROPIC_AWS_API_KEY`, 優先於 SigV4）.設定 `CLAUDE_CODE_USE_ANTHROPIC_AWS=1`, `ANTHROPIC_AWS_WORKSPACE_ID`, `AWS_REGION` 啟用, workspace ID 會隨每個請求以 `anthropic-workspace-id` 標頭送出.Bedrock 與 Foundry 在供應商路由上優先於 Claude Platform on AWS, 若同時設定需取消 `CLAUDE_CODE_USE_BEDROCK` 與 `CLAUDE_CODE_USE_FOUNDRY`.

## 23. Enterprise deployment overview 企業部署總覽與比較

多數組織適合用 Claude for Teams 或 Enterprise, 團隊成員以單一訂閱同時取得 Claude Code 與網頁版 Claude, 集中計費, 不需自建基礎架構.Teams 為自助式含協作與帳單管理功能, Enterprise 再加上 SSO, 網域認領, 角色型權限, compliance API, managed policy settings.

若有特定基礎架構需求, 六種選項在計費, 區域, 驗證, 成本追蹤, 是否含網頁版 Claude, 企業功能上各有差異, 詳見 Feature availability 對照表.企業最佳實務包含, 於多層級部署 CLAUDE.md 文件（組織層級系統目錄與 repository 層級）, 建立一鍵安裝流程簡化導入, 鼓勵新使用者先從程式碼問答與小型修復開始, 為雲端供應商部署釘選模型版本, 用 managed permissions 設定安全政策, 由中央團隊維護 checked-in 的 `.mcp.json` 供全員受益.
