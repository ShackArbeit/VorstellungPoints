# SSR / CSR / ISR + Cache + 併發處理（面試實戰心智圖）

## 0) 面試官真正想確認什麼
- 你能不能把「需求」翻成「渲染策略 / 架構 / 成本」
- 你知不知道「效能瓶頸在哪一層」：前端渲染 / 網路 / API / DB
- 你能不能處理「尖峰併發」：不爆 DB、不丟資料、不重複扣款、不把系統拖垮
- 你有沒有「可觀測性」概念：怎麼知道自己改善有效（指標/監控/trace）

---

## 1) SSR / CSR / ISR：怎麼選（用需求反推）
### 1.1 三個維度先問清楚（面試必問）
- 內容是否需要 SEO？
- 內容更新頻率（秒級、分鐘級、天級）？
- 使用者是否登入後才看得到（個人化程度）？

### 1.2 SSR（Server-Side Rendering）
- 適合
  - SEO 很重要（公開頁、landing page、文章頁）
  - 首屏體驗要快（TTFB 可接受、但要確保 server 撐得住）
  - 需要「每次都最新」或高度動態（但要注意成本）
- 面試官追問點
  - SSR 壓力打在 server：尖峰時怎麼避免 server 被打爆？
  - 你如何做 cache（CDN/edge、server cache）降低 SSR 成本？
- 常見地雷
  - SSR 每次都打 DB（無 cache）=> 併發一來 DB 先死

### 1.3 CSR（Client-Side Rendering）
- 適合
  - 登入後的 Dashboard / 後台管理（SEO 不重要、互動重）
  - 需要大量即時互動、狀態變化多
- 面試官追問點
  - CSR 首屏慢怎麼辦？（Skeleton/預取/分包/critical data）
  - API 延遲怎麼辦？（cache、retry、loading strategy）
- 常見地雷
  - 把公開頁面做 CSR（SEO 直接輸）
  - CSR 導致「每個人進來都狂打 API」=> server/DB 壓力變大

### 1.4 ISR（Incremental Static Regeneration）
- 適合
  - 內容大多可靜態，但需要「定期更新」
  - 例如：商品列表、新聞列表、文章列表（更新頻率可控）
- 面試官追問點
  - revalidate 設多久？如何決定？
  - 更新時會不會出現「短暫舊資料」？能接受嗎？
  - 有沒有更即時的需求（webhook / on-demand revalidate）？
- 常見地雷
  - 更新頻率太高卻用 ISR（等於一直在重建，浪費）

### 1.5 快速決策表（面試時用來講清楚）
- 公開頁 + SEO：SSR 或 ISR（依更新頻率）
- 登入後個人化：CSR（或 SSR + 個人化資料分離）
- 近乎不變內容：SSG
- 需要「幾乎即時」又要省成本：ISR + on-demand revalidate

---

## 2) Cache：DB Cache vs Server Cache（要講出層級）
### 2.1 Cache 層級地圖（從外到內）
- Browser cache（HTTP cache headers）
- CDN / Edge cache（離使用者最近）
- App / Server cache（Redis、memory cache）
- DB cache（DB buffer、query cache 概念、索引/執行計畫）
- 最後才是 DB disk / storage

### 2.2 DB Cache（面試重點不在「你會不會開」，而是你理解什麼）
- 你要講得像這樣
  - 「DB 的快」很多來自：索引、buffer pool、避免 full scan、避免 N+1
  - DB 不適合當 cache（因為 DB 是最貴的共享資源）
- 面試官追問點
  - 你怎麼判斷 query 慢？（explain、索引策略）
  - 什麼情況會失效？（資料量上升、selectivity 變差）

### 2.3 Server Cache（Redis / Memory）
- 適合
  - 熱門讀取：排行榜、熱門文章、商品列表、設定檔
  - 重複查詢成本高：複雜聚合、跨表 join 結果
- 兩個核心設計點（面試超愛問）
  - Cache Key 怎麼設計（包含參數、版本、語系、權限）
  - Cache Invalidation（何時失效、如何更新）
- TTL 策略
  - 穩定資料：TTL 長
  - 變動資料：TTL 短 + 主動失效（事件驅動）
- 面試官必追問的三個坑
  - Cache stampede（同時過期，大家打回 DB）
    - 解法：randomized TTL、single flight/lock、stale-while-revalidate
  - Cache penetration（查不存在的 key）
    - 解法：negative cache、Bloom filter
  - Cache inconsistency（寫入後讀到舊資料）
    - 解法：Write-through / Write-behind / Cache-aside + 正確失效策略

### 2.4 常見 cache pattern（你要能講出取捨）
- Cache-aside（最常用）
  - 讀：miss 才查 DB，再 set cache
  - 寫：先寫 DB，再刪 cache（或更新 cache）
  - 面試點：為什麼常用「刪 cache」而不是「更新 cache」？（避免 race condition）
- Write-through
  - 寫入同步寫 cache + DB
  - 面試點：一致性較好但寫入延遲提高
- Write-behind
  - 先寫 cache，異步落 DB
  - 面試點：高吞吐但一致性與資料安全要另外保護

---

## 3) 大量併發打 POST/GET：怎麼扛（實戰框架）
## 3.1 先分類：讀多？寫多？尖峰型？持續高流量？
- 讀多（GET）：優先 CDN/Edge + Server cache + 降低 DB hit
- 寫多（POST）：優先「保護 DB」+「正確性」+「可恢復」

### 3.2 GET 併發：把壓力擋在 DB 外面
- CDN/Edge cache（public data）
- Server cache（Redis）+ 防 stampede
- DB 層：索引、連線池、read replica（如有）
- 面試官常問
  - 你怎麼設定 HTTP cache header（ETag/Cache-Control）？
  - 你怎麼避免同一秒爆量 miss 打 DB？

### 3.3 POST 併發：正確性優先（尤其交易/扣款/狀態流）
- 三個面試關鍵字
  - Idempotency（冪等）
  - Concurrency control（併發控制）
  - Backpressure（背壓）
- Idempotency（必會）
  - 客戶端送 Idempotency-Key
  - Server 端用 key 記錄「已處理結果」
  - 面試官追問：key 存哪？TTL？重試怎麼回同一結果？
- 併發控制（你要能講至少兩種）
  - Optimistic locking（version / updated_at）
    - 適合：衝突不多、想要高吞吐
  - Pessimistic locking（select for update / 分散式鎖）
    - 適合：衝突多、一定要避免超賣/重複扣款
  - 原子操作（Redis INCR / Lua script）
    - 適合：計數、庫存扣減（但要設計一致性）
- Queue（排隊削峰）
  - 把「立即回應」與「背景處理」拆開
  - 面試官追問：哪些任務適合 async（寄信、影像處理、報表）？哪些不行（扣款需強一致）？

### 3.4 限流、熔斷、降級（保命三件套）
- Rate limiting（保護入口）
  - per IP / per user / per token
  - 面試官追問：怎麼做分散式限流？（Redis sliding window / token bucket）
- Circuit breaker（下游掛了不要拖死自己）
- Graceful degradation（降級）
  - 例：推薦系統掛了先回熱門清單；報表先回「排程中」

### 3.5 DB / Server 的「容量」與「連線」面試必考
- Connection pool
  - pool 太小：吞吐上不去
  - pool 太大：DB context switch 爆炸、反而更慢
- N+1 / 重複查詢：在併發下會被放大成災難
- 交易隔離等級與死鎖
  - 面試官常問：你遇過死鎖嗎？怎麼查？怎麼解？

---

## 4) Next.js + NestJS 的落地講法（面試說法模板）
### 4.1 渲染策略如何落在頁面上（你要能舉例）
- Public landing / blog
  - ISR（更新頻率可控）或 SSR（每次要最新）
- Product list / news list
  - ISR + revalidate（降低 server 成本）
- User dashboard（登入後）
  - CSR + API（避免 SSR 個人化造成 server 壓力）
- Admin console（內部）
  - CSR（互動多、SEO 不重要）

### 4.2 Cache 怎麼落地（面試講出你真的做過）
- CDN：快取公開頁面/靜態資源
- Redis：熱門列表、查詢結果、session/permission（視需求）
- Cache-aside：讀 miss 查 DB，寫入後刪 cache
- 防 stampede：鎖 / single flight / stale-while-revalidate

### 4.3 併發怎麼落地
- GET：CDN + Redis + 索引
- POST：Idempotency-Key + locking（optimistic/pessimistic）+ queue
- 入口保護：Rate limit + validation + payload size limit

---

## 5) 面試快問快答（你要能「一句話」回答）
- 什麼頁面用 SSR？
  - 「SEO 重要且要較新資料的公開頁，但我會用 cache/ISR 降低 SSR 壓力」
- ISR 的風險？
  - 「會有短暫舊資料；我會用 revalidate/on-demand revalidate 控制」
- Cache 最難的是什麼？
  - 「失效策略與一致性；我偏好 cache-aside + 寫入後刪 cache」
- 大量 POST 怎麼避免重複寫入？
  - 「Idempotency-Key + server 記錄已處理結果」
- 超賣怎麼避免？
  - 「悲觀鎖或原子扣減；看一致性與吞吐需求取捨」
- 併發高 DB 掛了怎麼辦？
  - 「入口限流 + queue 削峰 + 熔斷降級，優先保核心交易」

---

## 6) 自我檢查（你講得出來才算會）
- 我能不能用「SEO / 更新頻率 / 個人化」三句話決定 SSR/CSR/ISR？
- 我能不能畫出 cache 層級（browser/CDN/redis/db）並說明為什麼？
- 我能不能說清楚三個 cache 災難：stampede / penetration / inconsistency？
- 我能不能說清楚 POST 的三件事：idempotency / locking / backpressure？
- 我能不能用一個真實案例講完「我怎麼從瓶頸定位到改善驗證」？