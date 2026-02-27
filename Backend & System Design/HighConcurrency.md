# 大量併發與延遲怎麼處理（給非 AI / 非資訊背景的人看的版本）
> 技術基礎：NestJS / Next.js  
> 可直接使用 VSCode + Markmap 轉心智圖  
> 所有重要名詞皆附英文對應詞

---

# 一句話理解整件事

當很多人同時使用 AI 系統時，我們要做到：

- 不讓系統被擠爆（Rate Limiting）
- 不做重複工作（Cache）
- 讓使用者感覺沒有那麼慢（Streaming）
- 不讓系統卡死（Timeout）
- 把很花時間的事情放到後面慢慢做（Queue）

---

# 1. 什麼是大量併發？（High Concurrency）

意思是：
很多人「同一時間」一起使用系統。

就像百貨公司週年慶，同時很多人進門。

如果沒有控管：
- 伺服器會當機
- AI 服務會爆掉
- 資料庫會過載

---

# 2. 什麼是延遲？（Latency）

延遲就是：
從你按送出到看到回答，中間花的時間。

AI 延遲常見原因：
- AI 思考時間（Inference）
- 查資料時間（Retrieval / Vector Search）
- 資料庫查詢（Database）
- 網路傳輸（Network）

---

# 3. 第一件事：限制入口（Rate Limiting）

意思是：
不要讓太多人同時衝進來。

就像：
餐廳門口有服務生控管人數。

常見方式：
- 每個人每分鐘只能問幾次（User-based rate limit）
- 每個公司有使用上限（Tenant-based rate limit）
- 每個 IP 有限制（IP-based rate limit）

技術目的：
保護 AI 與資料庫。

---

# 4. 第二件事：減少重複工作（Cache）

意思是：
同樣的問題，不要每次都重新算一次。

就像：
常點的菜可以先準備好。

可以快取的東西：
- AI 回答（Response Cache）
- 查詢結果（Retrieval Cache）
- 向量結果（Embedding Cache）

好處：
- 變快
- 省錢（因為 AI 有 token 成本）

---

# 5. 第三件事：先回一點結果（Streaming）

意思是：
不要等全部完成才顯示。

就像：
餐廳先上前菜，不讓客人乾等。

技術方式：
- SSE（Server-Sent Events）
- WebSocket
- Chunked Response

效果：
使用者會感覺比較快。

---

# 6. 第四件事：設定超時（Timeout）

意思是：
如果太久沒回應，就停止。

就像：
等太久就取消訂單。

為什麼重要？
避免整個系統被一個卡住的請求拖累。

進階保護機制：
- 電路斷路器（Circuit Breaker）
  - 如果下游服務一直出錯，就暫時停止呼叫

---

# 7. 第五件事：用排隊方式處理（Queue）

意思是：
很花時間的事情不要馬上做。

就像：
拿號碼牌慢慢等。

適合丟進佇列的事情：
- 產生長篇報告（Long Generation）
- 文件大量匯入（Ingestion）
- 建立向量索引（Indexing）

做法：
- 先回傳一個工作編號（Job ID）
- 背景工作者處理（Worker）
- 完成後通知使用者

---

# 8. 整體邏輯總結

當很多人同時用 AI：

- 先限制人數（Rate Limiting）
- 能重複用就重複用（Cache）
- 先給一點結果（Streaming）
- 不要讓它卡死（Timeout）
- 太慢的放後面做（Queue）

這樣系統才會：

- 穩定（Stable）
- 快速（Fast）
- 可擴充（Scalable）
- 不容易當機（Reliable）

---

# 英文詞彙對照（Glossary）

- 大量併發：High Concurrency
- 延遲：Latency
- 速率限制：Rate Limiting
- 快取：Cache
- 串流：Streaming
- 超時：Timeout
- 電路斷路器：Circuit Breaker
- 佇列：Queue
- 背景工作者：Worker
- 向量搜尋：Vector Search
- 推理：Inference
- 嵌入向量：Embedding
