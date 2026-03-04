# Next.js 16 實務：大量使用者 GET/POST/DELETE 時，讓 Server 與畫面不卡的策略（Markmap）

## 核心心法：把流量分成兩種
- 讀取（GET）
  - 目標：回應快、重用結果、避免每個 request 重算
- 變更（POST / DELETE）
  - 目標：UI 先回、後端慢慢做、用精準失效維持一致性

---

## 1) GET 大量讀取：把「回應速度」做成預設很快
### A) Cache Components + use cache：可重用結果鎖住（Opt-in）
- Next.js 16：Cache Components 是 opt-in（需明確開啟/標記）
- 實務適用
  - 產品列表、商品詳情的「公共資訊」
  - 熱門搜尋、分類頁、常用設定
- 策略
  - 同一份資料避免被每個 request 重算
  - 用 tag / revalidate 設計失效策略（例如價格更新→失效對應 tag）

### B) 拆頁：快的殼 + 慢的資料（PPR + Streaming + Suspense）
- Partial Prerendering（PPR）
  - 先回 static shell（版型/框架）
  - 動態區塊用 streaming 補上
- Suspense
  - 慢資料（推薦、庫存、會員狀態）放在 Suspense
  - fallback 先出，使用者感覺不會「整頁卡住」

### C) GET API（Route Handlers）同樣可納入快取模型
- GET Route Handlers 可以走一致的 prerender / caching 策略
- 好處
  - 頁面與 API 統一快取/失效邏輯
  - 降低重複計算與尖峰壓力

---

## 2) POST / DELETE 變更：UI 快回 + 後端慢做
### A) UI 不等 Server：Optimistic UI + Transition
- 按下送出/刪除
  - UI 先立即反映（optimistic）
  - 背後再送 POST/DELETE
  - 失敗才 rollback + toast
- 目的
  - 讓「等待時間」不阻塞互動
  - 使用者體感不卡

### B) 變更後一致性：tag 失效，而不是整頁重刷
- POST/DELETE 完成後
  - 精準 invalidation（tag / revalidate）
  - 讓下一次讀取只重算必要區塊
- 避免
  - 整個 route 重跑 SSR
  - 全頁 reload 造成卡頓

### C) 高流量/慢操作：Queue/Job 背景化
- 何時需要
  - 多表交易、外部 API
  - 寄信、報表、索引更新
  - 檔案/影像處理
- 做法
  - 先回 202/OK + job id
  - 重活交給 worker（BullMQ / SQS / RabbitMQ / Cloud Tasks）
  - UI 觀察進度
    - polling
    - SSE / websocket（視需求）

---

## 3) Server 端「不被打爆」的必做點
### A) 連線管理：DB 連線池 / pooler
- 大量 request 常死在 DB 連線爆掉（不是 CPU）
- Serverless/Functions 更容易踩到連線上限
- 需確保
  - pool 設計合理
  - 連線回收與上限控制

### B) 限流與保護：Rate limit / circuit breaker
- 對登入、查詢、寫入做 rate limiting
- 下游保護
  - timeout
  - retry policy（有上限）
  - circuit breaker

### C) 部署層：concurrency / region / 系統限制
- Functions/Edge 會 scale，但有 concurrency / 上限概念（依平台/方案）
- 常見限制
  - 檔案描述符（FD）
  - 連線數上限
  - 單 instance 資源瓶頸
- 建議
  - 監控（APM/metrics/log）
  - 壓測與容量規劃

---

## 4) 快慢拆分決策表（最短結論）
- GET
  - 能 cache 就 cache（use cache + tag）
  - 能 PPR 就 PPR（先殼後肉）
- POST/DELETE
  - UI optimistic + 背景 job（慢的丟 queue）
  - 用 tag invalidation 精準刷新
- Server
  - DB pool、限流、timeout、監控