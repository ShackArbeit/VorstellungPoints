# LLM 系統流程與部署心智圖（RAG / Agent / Fine-tune）

## 1) 兩種常見流程對照

### A. Retrieval-first RAG Pipeline
- 目標
  - 先「找資料」再「生成/回答」
  - 強調可評估的檢索品質（Top-K、Rerank）
- 流程
  - User Query
    - 使用者輸入問題/關鍵字（例：疑似詐欺）
  - Embedding
    - 將 query 轉成向量（embedding vector）
  - Vector Retrieval (Top-K)
    - 到向量資料庫取回最相近的 K 筆候選片段（chunks/documents）
  - Rerank（Cross-Encoder Reranker）
    - 用 Cross-Encoder 對「query + 候選片段」做更精準排序
    - 產出 reranked Top-N（通常 N < K）
  - LLM Generation
    - 將 reranked 內容作為 context
    - 產生最終回答（可含引用/證據）
- FastAPI 的常見呈現方式（API 設計）
  - 一鍵端點（最常用）
    - POST /rag/answer
      - input: query, options(top_k, rerank_top_n)
      - output: answer, citations, trace
  - 分段端點（便於測試/觀測）
    - POST /rag/retrieve
    - POST /rag/rerank
    - POST /rag/generate
- 典型適用情境
  - 需要依據「規範/文件/類案」回答
  - 需要可引用、可稽核、可調參（Top-K / Rerank）

---

### B. LLM-first Orchestrated Retrieval（Agent / Tool-use Pipeline）
- 目標
  - 先「理解意圖/拆解任務」再「決定查哪裡、怎麼查」
  - 強調工具調用、資料源整合、治理（權限/稽核）
- 流程
  - Data Pipeline
  - User Ask
    - 使用者提出需求（可能更模糊、需要澄清）
  - LLM Interpret Meaning
    - LLM 解析意圖、抽取槽位（entities/constraints）
    - 決定下一步要用哪些工具/資料源
  - Augment Prompt
    - 加入系統規則、角色、限制、可用工具描述
    - 可能生成「結構化查詢」或「工具呼叫參數」
  - Query (Knowledge Datastore)
    - 呼叫工具：SQL / Keyword search / Graph / API / Doc store
    - 注意：通常不允許 raw SQL 直通，會做模板化/白名單
  - Relevant Data Retrieval
    - 取得資料後再彙整（可再交給 LLM 回答/決策）
- FastAPI 的常見呈現方式（API 設計）
  - 工具型端點（Tool Server）
    - POST /tools/kb_search
    - POST /tools/case_lookup
    - POST /tools/db_query (模板化)
  - 強調治理欄位
    - principal / scope / policy_decision / audit_log_id / trace_id
- 典型適用情境
  - 多資料源、多工具、流程複雜的任務
  - 需要強治理（RBAC、稽核、敏感資料遮罩）

---

### A vs B：最直覺差異
- A（Retrieval-first）
  - 先檢索再生成
  - FastAPI 像「檢索+生成管線服務」
  - 觀測重點：Top-K、rerank score、引用片段、latency breakdown
- B（LLM-first）
  - 先理解再查詢（工具編排）
  - FastAPI 像「工具與資料服務」
  - 觀測重點：工具呼叫成功率、policy block、資料遮罩、審計鏈

---

## 2) OpenAI Fine-tune：訓練過程 + FastAPI / Next.js 角色

### Fine-tune 的定位（LLM-as-a-Classifier）
- 用 LLM 但只做「分類/判斷」而非長文生成
- 輸入：短文本（例：新聞標題）
- 輸出：固定 label（例：YES/NO/REVIEW）+（可選）confidence/reasons

---

### OpenAI Fine-tune 訓練資料準備流程（SFT 監督式微調）
- Step 1：定義任務與標籤
  - label 設計（建議）
    - YES / NO / REVIEW（灰區交人工）
  - 輸出格式
    - 建議固定 JSON schema（可驗證）
- Step 2：整理資料集
  - 收集樣本（例：2000 筆）
  - 清洗與去重
  - 切分資料
    - train（訓練集）
    - validation/test（驗證/測試集）
- Step 3：轉成 JSONL
  - 一行一筆樣本
  - 常見採用 messages 格式（role: system/user/assistant）
- Step 4：上傳訓練檔
  - Files API 上傳（purpose: fine-tune）
- Step 5：建立 Fine-tuning Job
  - 指定 base model + training_file
  - 可選 validation_file
- Step 6：監看訓練結果
  - 完成後取得 fine-tuned model id
- Step 7：上線推論
  - 以新的 model id 進行 inference（像一般模型呼叫）
- Step 8：評估與迭代
  - 檢查誤判類型（false positive/negative）
  - 補資料、調標籤、重訓或做版本化

---

### 在 Fine-tune 架構下：FastAPI 的角色
- 角色定位：App Backend / Business API
- 核心職責
  - 對前端提供穩定端點
    - POST /fraud/classify
  - 呼叫外部 OpenAI（使用 fine-tuned model id）
  - 輸出驗證（重要）
    - JSON schema 驗證
    - label 必須在 enum 內
    - 不合格：重試 / fallback / 回 REVIEW
  - 決策層（policy）
    - confidence 門檻
      - 高：YES/NO
      - 中：REVIEW
  - 安全與成本控管
    - API key 不外流（前端不可直連 OpenAI）
    - rate limit / quota
    - timeout / retry / circuit breaker
  - 可觀測性與稽核
    - trace_id
    - latency、錯誤率、（可得的）token usage
    - audit log（誰查了什麼、結果是什麼）

---

### 在 Fine-tune 架構下：Next.js 的角色
- 角色定位：前端/體驗層（UI）
- 核心職責
  - 收集使用者輸入（title/content）
  - 呼叫 FastAPI（你的後端）
  - 顯示結果
    - label（YES/NO/REVIEW）
    - confidence（可視需求顯示）
    - reasons/evidence（若你決定提供）
    - trace_id（供除錯/稽核）
  - 人工覆核流程（若有 REVIEW）
    - 建立覆核 UI
    - 提交人工判定結果回 FastAPI（用於後續資料迭代）

---

### 部署建議（精簡版）
- 你自己的 Server
  - Next.js（UI）
  - FastAPI（Business API）
- 外部供應商
  - OpenAI（fine-tuned 模型推論/訓練）
- 推薦理由
  - 金鑰與治理留在後端
  - 前端簡單、安全、易維護
  - 後端可替換供應商/模型版本而不影響前端

---

## 3) 建議落地策略（可選）
- Two-stage routing（省成本 + 更穩）
  - 先走「純分類器」（fine-tuned classifier）
  - 若結果落灰區（REVIEW）或需要可引用證據
    - 再升級走 Retrieval-first RAG（Top-K + Rerank + LLM）