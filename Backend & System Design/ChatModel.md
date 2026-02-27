# Chat / Q&A 後端資料模型（NestJS / FastAPI）— Markmap 用 README

> 目標：用一套清楚、可擴充、好維運的資料模型，支援「聊天/問答」系統的核心能力：對話串、訊息、引用來源、使用者回饋，以及後續的分析與權限控管。  
> Target: A clear, extensible, maintainable data model for chat/Q&A: threads, messages, citations, feedback, plus analytics and access control.

---

## 1) 先用白話說：我們到底要存什麼？（Plain-language view）

- 一個「聊天室」= 一條對話線（Thread / Conversation）
  - 例如：你跟助理聊「RAG 架構」就是一條 thread
- 聊天室裡會有很多「訊息」（Message）
  - 你說一句、系統回一句，都是 message
- 有些回覆會「引用資料來源」（Citation / Source）
  - 例如：回答引用了公司內部文件、資料庫查詢結果、或某篇網頁
- 你可能會對回答做「回饋」（Feedback）
  - 例如：👍 有幫助、👎 不正確、或補充文字意見

> 這四個概念（thread / message / citation / feedback）就是最小可用核心（MVP core）。

---

## 2) 核心設計原則（Core principles）

- 可追溯（Traceability）：每一句答案最好能追到「用到哪些資料來源」(citations)
- 可擴充（Extensibility）：未來要加 RAG、工具呼叫、多人協作、都不翻桌
- 可控權限（Access control）：企業多租戶（Multi-tenant）時要能隔離資料
- 成本合理（Cost-aware）：訊息多、引用多，查詢要有效率（Index / Pagination）

---

## 3) 資料模型總覽（Entities overview）

- Thread（對話串 / Conversation）
- Message（訊息）
- Citation（引用來源 / Source reference）
- Feedback（回饋）
- （常見延伸）Attachment（附件）、ToolCall（工具呼叫）、Embedding（向量索引）

---

## 4) Thread（對話串 / Conversation）

### Thread 要解決什麼？
- 把一段聊天「分組」管理：標題、參與者、狀態、最後更新時間
- 企業情境下：同一個產品內，很多公司（tenant）各自有自己的 threads

### 建議欄位（Recommended fields）
- id（主鍵 / Primary key）
- tenant_id（租戶 / Tenant）
- title（標題 / Title）
- created_by（建立者 / Created by）
- status（狀態 / Status：active / archived / deleted）
- created_at / updated_at（建立/更新時間 / Timestamps）
- last_message_at（最後訊息時間 / Last activity）

---

## 5) Message（訊息）

### Message 要解決什麼？
- 存每一句話：誰說的、說了什麼、在第幾輪、以及（如果是 AI 回答）它是怎麼產生的

### 建議欄位（Recommended fields）
- id（主鍵 / Primary key）
- thread_id（所屬對話串 / Belongs to thread）
- role（角色 / Role：user / assistant / system / tool）
- content（內容 / Content：文字、或結構化 JSON）
- content_type（內容型別 / Content type：text / markdown / json）
- created_at（時間 / Timestamp）
- parent_message_id（父訊息 / Parent message，做成「樹狀」或「回覆」結構用，可選）
- seq（序號 / Sequence number：同一 thread 內排序用）
- model（模型 / Model name，可選：gpt-*, internal-llm）
- generation_meta（生成資訊 / Generation metadata：temperature、tokens、latency…，可選）

### 為什麼建議加 seq？
- 有時 created_at 可能會同秒、或匯入歷史資料，seq 更穩定好排序。

---

## 6) Citation（引用來源 / Source reference）

### Citation 要解決什麼？
- AI 回答不是憑空出現：它通常根據某些文件片段、資料庫查詢結果、或網頁內容
- 我們要存「這一句回答」用到了「哪些來源」，以及「用到哪一段」

### 常見兩種做法
- A. 一句 assistant message 連很多 citations（1:N）
- B. citations 先指向「來源文件」（Source / Document），再連到 message（更正規但多一張表）

這裡先用容易落地的 A 做法：Citation 直接掛在 message 上。

### 建議欄位（Recommended fields）
- id（主鍵 / Primary key）
- message_id（被引用的回答訊息 / The assistant message）
- source_type（來源類型 / Source type：document / web / db / tool）
- source_id（來源識別 / Source id：文件 ID、URL、查詢 ID…）
- source_title（來源標題 / Source title，可選）
- locator（定位資訊 / Locator：頁碼、段落、行號、時間戳）
- snippet（引用片段 / Snippet：短摘要，避免存整篇）
- confidence（信心分數 / Confidence，可選）

> 小提醒：snippet 建議限制長度，避免把整份機密文件存進資料庫。

---

## 7) Feedback（回饋）

### Feedback 要解決什麼？
- 你要知道：回答到底好不好？哪裡需要改？
- 企業場景：也可能需要稽核（audit）誰給了什麼回饋

### 建議欄位（Recommended fields）
- id（主鍵 / Primary key）
- message_id（被評分的訊息 / Target message）
- user_id（誰給的 / Who）
- rating（評分 / Rating：+1 / -1 或 1~5）
- tags（標籤 / Tags：hallucination、too_long、not_helpful…）
- comment（文字意見 / Comment）
- created_at（時間 / Timestamp）

---

## 8) 最小可用（MVP）關聯圖（Relationship sketch）

- Thread 1 ── N Message  
- Message 1 ── N Citation（通常只針對 assistant message）  
- Message 1 ── N Feedback（多人都能評同一句）  

---

## 9) 企業常見需求：Multi-tenant（多租戶）怎麼放進模型？

### 一句話版本
- 每張表都要能「分租戶」（tenant_id），避免 A 公司看到 B 公司資料。

### 實作建議（Practical patterns）
- 方案 1：每張表都有 tenant_id（Row-level isolation / 列級隔離）
  - 優點：簡單、成本低、好維運
  - 缺點：必須非常嚴格做查詢條件與索引（tenant_id + thread_id…）
- 方案 2：不同 tenant 不同 schema（Schema per tenant / 每租戶一個 schema）
  - 優點：隔離更強
  - 缺點：維運與遷移成本高（migration 很痛）
- 方案 3：不同 tenant 不同 DB（Database per tenant / 每租戶一套資料庫）
  - 優點：隔離最強、法規友好
  - 缺點：成本最高、運維最複雜

> 多數中大型 SaaS：先從「Row-level」開始，除非客戶/法規要求更高隔離。

---

## 10) 索引與查詢（Indexing & querying essentials）

你要確保這些查詢很快：
- 列出某 thread 的訊息（List messages by thread）
  - index: (tenant_id, thread_id, seq)
- 取某句回答的 citations（Get citations for a message）
  - index: (tenant_id, message_id)
- 統計某段時間的回饋（Feedback analytics）
  - index: (tenant_id, created_at)

---

## 11) 參考：簡化的資料表草稿（Simplified table draft）

### threads
- id, tenant_id, title, created_by, status, created_at, updated_at, last_message_at

### messages
- id, tenant_id, thread_id, role, content, content_type, seq, parent_message_id, model, generation_meta, created_at

### citations
- id, tenant_id, message_id, source_type, source_id, source_title, locator, snippet, confidence

### feedback
- id, tenant_id, message_id, user_id, rating, tags, comment, created_at

---

## 12) NestJS / FastAPI 落地小技巧（Implementation tips）

- 「寫入」流程（Write path）
  - 先建立 user message
  - 再由 AI 產生 assistant message
  - 把 citations 一起寫入（同一個 transaction）
- 「讀取」流程（Read path）
  - 先取 messages（分頁 / pagination）
  - 再批次取 citations 與 feedback（避免 N+1 query）
- API 設計（API shape）
  - GET /threads
  - POST /threads
  - GET /threads/{thread_id}/messages?cursor=...
  - POST /threads/{thread_id}/messages（送出 user message）
  - POST /messages/{message_id}/feedback
- 事件/稽核（Audit / Event log，可選）
  - 企業常需要「誰在什麼時間做了什麼」：可加一張 audit_log 表

---

## 13) 你在面試可以怎麼講（Interview-friendly phrasing）

- 我用 Thread 管對話生命週期，用 Message 存每輪互動  
- 我把可追溯性做進核心：assistant message 對應多個 citations  
- feedback 讓我們能做品質監控（quality monitoring）與持續迭代  
- 多租戶用 tenant_id 做 row-level isolation，配合索引與中介層強制過濾，避免資料外洩

---

## 14) 英中對照小辭典（Mini glossary）

- Thread = 對話串 / Conversation
- Message = 訊息
- Citation = 引用 / Source reference
- Feedback = 回饋
- Tenant = 租戶
- Multi-tenant = 多租戶
- Row-level isolation = 列級隔離
- Schema per tenant = 每租戶一個 schema
- Database per tenant = 每租戶一套資料庫
- Traceability = 可追溯性
- Pagination = 分頁
- Index = 索引