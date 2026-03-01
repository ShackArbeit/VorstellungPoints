# LLM + Knowledge Datastore 流程架構說明

## 1️⃣ Data Pipeline（資料管道）
- 將各種資料來源匯入 Knowledge Datastore
- 資料來源包含：
  - 結構化資料（Structured）
    - Network
    - Billing
    - CRM
    - Ordering
    - Usage
    - Inventory
  - 非結構化 / 半結構化資料
    - Clickstream
    - Sensors
    - Machine Logs
    - Social
- 目的：
  - 整理、清洗、轉換資料
  - 建立可供查詢的知識儲存體系

---

## 2️⃣ Ask（使用者提問）
- 使用者提出問題
- 進入系統查詢流程
- 問題可能是：
  - 問答（Q&A）
  - 故事生成（Story Generation）
  - 資料分析

---

## 3️⃣ LLM 語意理解（Interpret Meaning）
- 透過 API 呼叫大型語言模型（LLM）
  - Gemini
  - OpenAI
  - Azure OpenAI
  - LLaMA (Meta)
  - Mistral AI
  - Claude
- 功能：
  - 理解使用者問題語意
  - 分析字詞與上下文關係
  - 推導意圖（Intent Recognition）

---

## 4️⃣ Augment Prompt（提示詞增強）
- 根據語意與關聯關係生成查詢條件
- 行為示例：
  - Action: load
  - Method: by relation
  - Nodes: Parts, Supplier
  - Relationship: Supply
  - Subject: Intel
- 目的：
  - 建立強化後的 Prompt
  - 提供 LLM 結構化背景知識
  - 將知識圖譜關聯納入上下文

---

## 5️⃣ Query（查詢 Knowledge Datastore）
- 將增強後的查詢送入 Knowledge Datastore
- 查詢方式：
  - Graph 查詢
  - 關聯查詢
  - 節點與關係分析
- 目標：
  - 取得與問題高度相關的資料

---

## 6️⃣ Relevant Data Retrieval（相關資料擷取）
- 從 Knowledge Datastore 擷取相關資料
- 可進行：
  - 視覺化（Visualization）
  - Q&A 模式呈現
  - 故事型生成（Story Generation）
- 作為 LLM 的補充上下文（Context Injection）

---

## 7️⃣ Answer（產生最終答案）
- LLM 綜合：
  - 使用者問題
  - 擷取資料
  - 關聯分析結果
- 產生最終回覆
- 回傳給使用者

---

# 整體流程總結

Data Sources  
→ (1) Data Pipeline  
→ Knowledge Datastore  
→ (2) User Ask  
→ (3) LLM 語意理解  
→ (4) Prompt Augmentation  
→ (5) Query  
→ (6) Relevant Data Retrieval  
→ (7) Answer