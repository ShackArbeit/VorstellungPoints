# Hybrid RAG 高併發：核心指標 + 名詞超精簡解釋（Markmap）

## 目標
- 尖峰大量使用者同時問：**不排隊爆炸、TTFT 穩、tokens/s 穩、p95/p99 低、快取命中高、租戶公平**

---

# 1) LLM queue length / wait time（最重要）

## LLM queue length
- **等待 LLM/GPU 處理的請求數**（pending requests）
- 越長＝越多人卡住＝體感變差

## LLM wait time
- **請求進到 LLM serving 後，開始推論前的等待時間**
- wait time ↑ 會直接讓 **TTFT ↑**

---

# 2) TTFT 與 tokens/s（體感 + 吞吐）

## TTFT（Time To First Token）
- **送出請求 → 收到第一個 token** 的時間
- 使用者最敏感：TTFT 低＝「馬上有回應」

## tokens/s
- **LLM 輸出速度**（每秒吐出多少 token）
- tokens/s 低或不穩＝「回答慢吞吞」

---

# 3) cache hit rate（retrieval / semantic）

## Retrieval cache hit rate
- **檢索結果快取命中率**
- 命中＝少打 Vector/Graph DB → p95/p99 降、成本降

## Semantic cache hit rate
- **語意相似就命中快取**（不同問法但同一問題）
- 企業場景必加維度：**tenant / role / dept**（避免跨權限命中造成外洩）

---

# 4) Vector/Graph query p95/p99（尾延遲）

## p95 / p99
- p95：95% 請求延遲低於這個值
- p99：最慢 1% 的「典型很慢」
- 尾延遲高＝少數人超卡，但最容易被抱怨

## 要分開量
- vector_query_latency p95/p99
- graph_query_latency p95/p99
- end-to-end retrieval p95/p99（含權限過濾/合併）

---

# 5) continuous batching（高併發救命）

## 一句話
- **動態 batch**：新請求可隨時加入、完成的隨時移出
- 目的：**提高 GPU 吞吐、降低排隊、讓 TTFT 更穩**

---

# 6) KV Cache Reuse（Prefix caching）

## 一句話
- 新請求如果和舊請求 **prompt 前綴相同**，就**重用已算過的 KV**
- 目的：**少算重複 prompt → 延遲降、吞吐升**

## 多副本注意
- 要「相似 prefix 導到同一 replica」才容易命中（routing 影響命中率）

---

# 7) NestJS / FastAPI 可能的角色（怎麼分工）

## NestJS（偏治理 / 多租戶 / 入口）
- API Gateway / BFF：Auth、RBAC、tenant 配額、rate limit
- Orchestrator：決定走哪條 pipeline、是否降級、串快取與 LLM

## FastAPI（偏高效 I/O / 模型周邊）
- Retrieval service：向量/圖譜查詢、合併、過濾
- Rerank service：需要時才 rerank（尖峰可關）
- LLM adapter：對接 vLLM/Triton/模型服務，負責 streaming 回傳

---

# 8) 每租戶用量（配額）＝公平 + 成本控制

## 常見配額
- Request quota：req/min、req/day
- Concurrency quota：同時進行請求上限（最能控 queue）
- Token quota：input/output tokens/day + 每次 max_output_tokens（硬限制）

## 與指標連動（尖峰保命）
- 若某租戶造成 queue/TTFT 明顯升高：
  - 降該租戶 concurrency
  - 啟用更激進降級：停 rerank、降 Top-K、縮短輸出

---

# 9) 最小監控面板（MVP）

## LLM
- queue length、wait time、TTFT(p50/p95/p99)、tokens/s

## Cache
- retrieval hit rate、semantic hit rate（分 tenant/role）

## Retrieval
- vector p95/p99、graph p95/p99

## Governance
- per-tenant req/min、concurrency、tokens/day(in/out)