# RAG 架構與 Rerank 設計說明（Retrieval-Augmented Generation Architecture）

---

## 一、RAG 整體流程（RAG Pipeline）

- 使用者問題（User Query）
- 向量化（Embedding）
- 向量檢索（Vector Retrieval, Top-K）
- 重新排序（Rerank, Cross-Encoder Reranker）
- 大型語言模型生成（LLM Generation）

---

## 二、Rerank 的核心價值（Value of Rerank）

### 第一階段：向量檢索（Vector Retrieval）

特性：
- 使用近似最近鄰搜尋（Approximate Nearest Neighbor, ANN）
- 計算向量相似度（Vector Similarity）
- 速度快（Fast）
- 偏重召回率（Recall-oriented）

問題：
- 向量相似 ≠ 真正語意最相關（Semantic Relevance）

---

### 第二階段：重新排序（Rerank, Cross-Encoder）

特性：
- 同時輸入 Query + Document
- 使用 Transformer 完整注意力機制（Full Attention）
- 計算相關性分數（Relevance Score）
- 偏重精準度（Precision-oriented）
- 速度較慢但更精準（Slower but Accurate）

---

## 三、Rerank 在系統中的戰略價值（Strategic Value）

- 提升 Precision@Top-N
- 降低幻覺風險（Hallucination Risk）
- 優化 LLM 上下文使用（Context Optimization）
- 提升引用品質（Citation Quality）

設計理念：
- Retrieve 先便宜（Cheap Retrieval）
- Rerank 再精準（Precise Reranking）

---

## 四、延遲控制設計（Latency Control Design）

### 延遲預算（Latency Budget）

目標：
Total Latency < 1.5 秒

建議分配：

- Embedding：100ms
- Retrieval：100ms
- Rerank：400ms
- LLM：800ms

---

## 五、控制延遲的方法（Latency Optimization Strategies）

### 1️⃣ 控制 Top-K

範例：

- retrieve top-20 → rerank → top-5
- 若延遲超標則降低 k 值

核心概念：
動態調整（Dynamic K Adjustment）

---

### 2️⃣ 兩階段重新排序（Two-Stage Rerank）

- 第一階段：輕量模型（Lightweight Model）
- 第二階段：重模型（Heavy Model）

---

### 3️⃣ 快取策略（Caching Strategy）

- Query Cache
- Embedding Cache
- Retrieval Result Cache

---

### 4️⃣ 降級策略（Degradation Strategy）

當延遲超出門檻（Latency Threshold）時：

- 跳過 rerank
- 降低 top-k
- 使用 BM25 備援（Fallback）

---

## 六、NextJs 與 NestJs 在架構中的角色（System Role）

### NestJs（後端協調器 Backend Orchestrator）

負責：

- API 層（API Layer）
- 呼叫 Embedding 服務
- 呼叫 Vector DB
- 呼叫 Reranker
- 呼叫 LLM
- 延遲控制邏輯（Latency Control Logic）
- 快取與降級策略（Cache & Fallback Logic）

核心角色：
RAG 協調中樞（RAG Orchestration Layer）

---

### NextJs（前端體驗層 Frontend UX Layer）

負責：

- 使用者提問介面（User Interface）
- 串流回應（Streaming Response）
- 顯示引用來源（Citation Rendering）
- Loading 狀態控制

核心角色：
使用者體驗優化層（UX Optimization Layer）

---

## 七、給非技術背景的說明（Non-Technical Explanation）

想像你在圖書館找答案：

第一步：
館員快速幫你找出 20 本可能相關的書（Retrieve）

第二步：
專家幫你翻閱這 20 本書，挑出 3 本真正最符合問題的（Rerank）

第三步：
你閱讀最準確的內容得到答案（LLM）

設計哲學：

- 先用便宜快速的方法篩選
- 再用精準但較慢的方法確認
- 在速度與品質之間取得平衡（Balance Speed and Quality）