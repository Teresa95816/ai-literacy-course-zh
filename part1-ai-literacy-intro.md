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

# AI 素質養成入門課 — 為白領上班族設計

## Part 1 : 認識你的工作夥伴

> 你不需要會寫程式，也不需要懂技術。
> 你只需要願意開口問，AI 就能成為你最強的工作夥伴。

---

### 1-1 神經網路 (Neural Network)

大型語言模型 (LLM) 是一種利用大量文字資料訓練而成的人工智慧模型。

其底層 Transformer 模型是一組神經網絡。

透過 Transformer 架構與自注意力機制，學習文字之間的關聯與語意。

> 神經網路的命名和設計靈感直接來自人腦。

[Neural Network Demo](neural-network-demo.html)

---

### 1-2 預訓練 (Pre-training)

- 開發者取得 Internet 上海量的文本，利用數千個 GPU 持續運行十數日，最終將這些資訊「壓縮」進一個檔案中。
- 基礎模型 (Base Model)= 壓縮檔案 + 一段 C 語言程式碼

> 每個模型都有「知識截止日(Knowledge Cutoff)」，截止日之後的事它一概不知。

---

### 1-3 微調 (Fine-tuning) 與 對齊(Alignment)

- 一個「助手模型」(Assistant Model)，以符合規範的方式和語氣回答問題。
- 不能教人製作武器、不能散布仇恨言論
- 不能產生幻覺(Hallucination)而捏造不存在的事實

> 企業「自建 AI 助理」最務實的路線是 - 
> 檢索增強生成(Retrieval-Augmented Generation，簡稱 RAG) 
> 把內部文件整理成可以查找的資料庫，讓 AI 回答前先查，既比重訓便宜，資料也隨時可以更新。

---

### 1-4 MoE (混合專家模型)

> 當你在聽音樂時，大腦的聽覺皮質最活躍；當你在解數學題時，前額葉最忙碌。人腦不會讓所有神經元同時全力運轉——它會按需啟動對應的專區。

MoE 模型的設計正是如此：每次收到輸入，由「路由器」決定派哪幾個專家子網路處理，其餘保持靜止，最後將各專家的輸出加權合併。這讓模型能同時達到兩件事：知識更廣(各領域各有擅長的專家)，和成本更低(每次只啟動部分資源)。

---

### 1-5 推論 (Inference)

- 運行現有模型以生成文字。
- 每次只預測一個 Token，預測完再把它加進去，然後預測下一個，如此循環直到輸出完整。

> Token 是 AI 閱讀和思考的基本單位，不完全等於「字」或「詞」。

[OpenAI/Platfom/Tokenzier](https://platform.openai.com/tokenizer)

---

### 1-6 思維鏈 (Chain of Thought)

2022 年，研究人員發現一個有趣的現象：只要在 Prompt 中加入「請一步一步思考（Let's think step by step）」等引導語，模型在數學計算、邏輯推理與常識判斷等任務上的表現便能明顯提升。

早期的 CoT 是一種 Prompt Engineering 技巧，需要由使用者主動觸發；而自 2024 年起，許多先進模型已將推理能力內建於模型架構中，能自動進行多步驟思考。

> CoT 產生的推理 Token 會消耗額外運算資源，因此在部分模型或服務中，這些 Token 也會納入計費或用量統計。

---

### 1-7 無狀態與多輪對話 (Stateless & Multi-turn Conversation)

電影《我的失憶女友》(50 First Dates)裡，女主角 Lucy 因車禍每天醒來都會失去前一天的記憶。每一天對她來說都是全新的開始。男主角 Henry 為了讓她記得兩人的感情，每天早上準備一捲影帶，讓她從頭看完所有的故事，才能繼續當天的對話。

模型一次能「看到」的 Token 數量是有上限的，稱為上下文視窗 (Context Window)。

超過這個上限，AI 就會「忘記」對話最前面的內容。
  
> 開始一段長工作前，先在對話開頭貼上「背景摘要」，確保每次它重新讀取時，關鍵資訊都在視窗最顯眼的位置。

---

### 1-8 系統提示 (System Prompt)

在你輸入第一個字之前，產品開發者早已先向 AI 下達一份隱藏指令，規範它的角色、能力邊界與回應風格。這份指令稱為 System Prompt（系統提示）。

[Claude API Docs/Models & pricing/Models/System Prompts](https://platform.claude.com/docs/en/release-notes/system-prompts)

> 對 AI 而言，System Prompt 就像員工手冊；而使用者的 Prompt，則像是當下交辦的工作。

---

### 1-9 工具呼叫 (Tool Use)

早期的大型語言模型只能根據訓練資料生成文字；而現代 AI 助理則能在需要時主動使用工具，取得即時資訊、執行計算，甚至操作外部系統。

以「整理某公司的歷年融資紀錄並製作趨勢圖」為例，AI 可能會依序完成以下流程：

|步驟|工具|用途|
|:-|:-|:-|
|1|網頁搜尋|查找公開融資資料|
|2|計算工具|推算缺失數據或估值|
|3|資料分析工具|計算成長率並繪製圖表|
|4|圖像生成工具|產生相關示意圖片|

> 使用者只需描述需求，AI 便能自行判斷何時呼叫工具、如何串接結果，以及何時將成果回傳給使用者。

---

### 1-10 多模態 (Multimodality)

早期語言模型只能處理文字。現在已能同時處理多種形式的資訊。

|能力 | 說明 | 實際場景 |
|:-----|:----|:--------|
| 看圖(Vision) | 讀取並理解圖片內容 | 上傳手寫筆記或工作表格照片，AI 直接整理成結構化文件 |
| 生圖(Image Generation) | 根據文字描述產生圖像 | 輸入產品概念，AI 輸出視覺化設計稿 |
| 語音輸入(Speech Input) | 將口語轉換為文字再處理 | 會議錄音直接送入 AI 生成摘要 |
| 語音輸出(Speech Output) | 將回答轉換為語音播出 | 與 AI 進行即時語音對話 |
| 影片理解(Video Understanding) | 分析影片中的畫面與事件| 自動整理課程重點、監控異常或比賽內容 |

---

### 小結：Part 1 概念一覽

| 階段 | 概念 | 核心要點 |
|:---|:---|:---|
| 訓練 | 神經網路 | AI 的學習架構，靈感來自人腦 |
| 訓練 | 預訓練 | 從海量文字壓縮知識，形成基礎模型 |
| 訓練 | 微調與對齊 | 將基礎模型調教為守規矩的助手 |
| 訓練 | MoE | 按需啟動專家子網路，效率與廣度兼顧 |
| 推論 | Token | AI 閱讀與計費的基本單位 |
| 推論 | Context Window | 模型每次能「看到」的內容上限 |
| 推論 | 思維鏈 | 引導逐步推理，複雜任務表現更佳 |
| 使用 | 無狀態 + 多輪對話 | 對話歷史由介面管理，模型本身不記憶 |
| 使用 | 系統提示 | 隱藏的底稿，規範 AI 的角色與邊界 |
| 使用 | 工具呼叫 | 串接外部工具，從文字預測到問題解決 |
| 使用 | 多模態 | 擴展至圖像、語音、影片的輸入與輸出 |

---

### 前置準備：個人筆電

> 請使用自己的個人筆電上課：基於公司 IT 政策，公司電腦不適合安裝 Claude Code。
> 保持網路連線狀態。

一般的 Windows 10 或 Windows 11 筆電皆可使用 Claude Code。

| Apple | 晶片 | 是否適用 |
|:------|:------|:---------|
| MacBook Air / Pro(2020 年後)| Apple M1 / M2 / M3 / M4 | 完全適用，效能極佳 |
| MacBook Air / Pro(2017–2020)| Intel Core i5 / i7 | 完全適用 |
| 2016 年以前的 MacBook | Intel | 建議 macOS 12(Monterey)以上|

---

#### Step 1：建立 Google 帳戶

建議安裝 [Chrome](https://www.google.com/chrome) 瀏覽器，並設為預設瀏覽器。

前往 [accounts.google.com/signup](https://accounts.google.com/signup)，依照指示填寫姓名、使用者名稱、密碼，完成手機號碼驗證後即可啟用 Google 帳戶。

> 如果你已有 Google 帳戶(Gmail、Google Drive、YouTube 等)，跳過此步驟，直接使用現有帳號即可。

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

[claude.ai](https://claude.ai)，推薦 Google 的登入方式

#### Step 4：升級為 Pro 用戶

> 訂閱是月付制，你可以隨時在帳號設定中取消，不會被鎖死。
> 訂閱之後怎麼取消？
> 進入 Settings → Billing → Manage Subscription，點選取消。取消後，你的 Pro 資格會維持到當月結束，之後自動降為免費版，不會退費已付的月費。

1. 登入 Claude 的網頁介面後， 點選左下角的 「Upgrade to Pro」(或 「升級方案」)
2. 選擇 「Claude Pro」 方案
3. 填入信用卡資料，確認訂閱，完成付款

---

#### Step 5：確認 Pro 狀態

升級完成後，你可以在網頁版對話介面的左下角看到你的帳號名稱，旁邊會出現「Pro」的標示。
或者：點選右上角的帳號頭像 → 「Settings」 → 「Billing」，確認方案顯示為「Claude Pro」。

#### Step 6：已有其他供應商訂閱？確認替代工具

如果你已有 ChatGPT Plus 或 Gemini Advanced 的付費訂閱，不需要另外訂閱 Claude Pro。三家供應商都有對應的 CLI 工具，功能定位相同：

| 你已有的訂閱 | 對應工具 | 安裝指令 |
|------------|---------|---------|
| Claude Pro(Anthropic)| Claude Code | `npm install -g @anthropic-ai/claude-code` |
| ChatGPT Plus(OpenAI)| Codex CLI | `npm install -g @openai/codex` |
| Google 帳號 / Gemini Advanced | Gemini CLI | `npm install -g @google/gemini-cli` |

> 課堂示範統一使用 Claude Code，若你使用其他工具，操作邏輯完全相同，可平行對照。Node.js、Git、VS Code 的安裝步驟對三種工具都適用。

---

#### Step 7：預先下載三個工具的安裝檔

課堂上會逐步安裝以下工具。請在上課前先下載安裝檔，存放在桌面或下載資料夾即可，**不需要現在安裝**。

| 工具 | 下載來源 | 備註 |
|------|---------|------|
| Node.js | [nodejs.org](https://nodejs.org) | 選 LTS 版本，用途是安裝 Claude Code |
| Git | [git-scm.com/downloads](https://git-scm.com/downloads) | 選 Windows / Mac 對應版本 |
| VS Code | [Visual Studio Code](https://code.visualstudio.com) | 選 Windows / Mac 對應版本 |

> 課堂網路環境不一定穩定，提前下載可以讓安裝過程更順暢。安裝步驟將於課堂中逐一進行，講師會全程在場協助，不需要自己摸索。

---

> 認識一個夥伴，從了解他的思維方式開始。你現在知道它怎麼學習、怎麼思考、為什麼有時說錯話、又為什麼有時令你驚艷。認識之後，才是真正合作的起點。
