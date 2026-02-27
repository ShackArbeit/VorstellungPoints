# Hybrid RAG（混合式檢索生成, Hybrid Retrieval-Augmented Generation）

## 為什麼企業需要 Hybrid RAG？

Hybrid RAG 是同時結合：

- 關鍵字搜尋（BM25, Keyword Search）
- 語意搜尋（Vector Search, Semantic Search）

來提高搜尋準確度與穩定性。

---

## 企業資料的特性

企業資料常包含：

- 法律條文編號（Legal Article Numbers）
- 技術型號（Model Numbers）
- 內部代碼（Internal Codes）
- 公司名稱（Company Names）
- 人名（Personal Names）
- 專有名詞（Proper Nouns）

這些資訊需要「精準匹配」。

---

## 語意搜尋（Vector Search, Semantic Search）

### 原理
將文字轉換為向量（Embedding），
找出意思相近的內容。

### 優點
- 能理解自然語言
- 能處理不同說法

### 缺點
- 對數字與條文可能不夠精準

---

## 關鍵字搜尋（BM25, Sparse Retrieval）

### 原理
依據字詞出現次數與重要性計算分數。

### 優點
- 精準匹配條文與型號
- 適合法律與金融場景

### 缺點
- 不理解語意

---

## Hybrid RAG 的做法

1. 同時進行：
   - BM25 搜尋
   - Vector 搜尋
2. 合併結果
3. 交給大型語言模型（LLM, Large Language Model）生成答案

---

## 何時必須使用 Hybrid？

當資料包含：

- 法律條文
- 金融代碼
- 技術型號
- 專有名詞（Proper Nouns）
- 企業內部專名

Hybrid 能提升準確率與穩定性。

---

## 系統架構角色分工

### Next.js（前端框架, Frontend Framework）

負責：
- 使用者介面
- 顯示答案
- 顯示引用來源（Citation）
- 登入與權限控制

---

### NestJS（後端框架, Backend Framework）

負責：
- 同時呼叫 BM25 與 Vector 搜尋
- 合併搜尋結果
- 呼叫 AI 模型（LLM）
- 控制安全與審計紀錄（Audit Log）

---

## 簡單比喻

Hybrid RAG 就像在圖書館找資料：

- 語意搜尋 = 問圖書館員
- 關鍵字搜尋 = 自己查索引
- Hybrid = 兩種方式同時使用

能確保找到正確且完整的資訊。