# CreatePair（瑞佩資訊）Next.js 前端工程師（Next.js 16）可能面試題庫（依官網推測）

> 依 CreatePair 官網資訊：主產品 **MEDITRUSS**，主打診所管理「人資 + 物料（進銷存）」的一站式系統，強調即時數據、流程自動化與提升效率，並提到支援 App、報表分析、以及以 LINE 自動通知採購等流程整合。 :contentReference[oaicite:0]{index=0}

---

## 1) 你可能會被期待「先講清楚產品」的問題（Domain Fit）

- 你怎麼理解 MEDITRUSS 的核心價值主張？
  - 對象（院長/助理/醫師）各自的痛點是什麼？你會如何對應到 UI/Flow？ :contentReference[oaicite:1]{index=1}
- 「人資模組」與「物管模組」在前端資訊架構（IA）上，你會怎麼拆頁面/導覽？
  - 哪些是高頻操作（助理）？哪些是決策頁（院長看報表）？ :contentReference[oaicite:2]{index=2}
- 如果要做「三步驟建置完成 / 到場教學 / 匯入系統」的 onboarding，你會怎麼設計前端流程？
  - 匯入檔案、欄位 mapping、錯誤回饋、回滾、進度條、重試策略 :contentReference[oaicite:3]{index=3}

---

## 2) Next.js 16 核心能力題（App Router / RSC / Cache / Data Fetching）

- 你在 Next.js 16 會怎麼選擇：
  - Server Components vs Client Components？判斷依據是什麼？
  - 什麼情況你「一定」要用 client（互動/狀態/瀏覽器 API），什麼情況 server 更划算（資料取得/SEO/權限 gating）？
- 資料取得策略（重點：效能與一致性）
  - `fetch()` 在 App Router 的 cache 行為你怎麼理解？
  - 你會如何設計「報表頁（偏靜態/可快取）」與「庫存即時查詢（偏動態）」在同一路由的呈現？
- ISR / revalidate 與一致性
  - 報表（每日/每週）適合多久 revalidate？
  - 庫存異動後要怎麼做到「使用者看到最新」又不把整站快取打爆？（tags / paths 的思路）
- Streaming / Suspense
  - 你會如何讓「報表卡片先出骨架、數據逐塊串流進來」，降低首屏等待？
- Route Handlers / Server Actions
  - 你會怎麼選？（表單送出、審核流程、領料掃碼、採購確認…）
  - 如何做錯誤處理（domain error vs system error）與 UI 回饋？

---

## 3) 針對「診所系統」的資安/隱私與權限（很可能被問）

> 官網 FAQ 直接提到「資料隱密性」、「資料會不會被代理商看到」等疑慮，這類通常會延伸成面試題。 :contentReference[oaicite:4]{index=4}

- 前端你能做的資安措施有哪些？
  - XSS / CSRF / Clickjacking / Token storage（cookie vs localStorage）取捨
- 權限與資料隔離（Multi-tenant / RBAC）
  - 「不同診所」資料如何確保前端不會拿錯？你會怎麼設計 query key / cache key？
  - 權限變更（例如助理升主管）後，前端如何同步刷新可見功能？
- 稽核與可追溯性（Auditability）
  - 庫存領出、盤點、報廢、排班審核等操作，前端如何協助留下「可用」的操作紀錄（可讀性、可搜尋、可匯出）？ :contentReference[oaicite:5]{index=5}

---

## 4) 高互動業務頁的狀態管理與 UI 設計題

- 表單與流程（常見於：排班、考勤審核、領料/盤點、採購）
  - 你會如何做：
    - 表單驗證（同步/非同步）
    - 草稿保存
    - 送出防重（double-submit）
    - 失敗重試與補償（compensation）
- 大量表格資料（庫存清單、異動紀錄、工時報表）
  - 你會怎麼做 pagination / infinite scroll / virtualization？
  - 篩選、排序、搜尋要放前端還是後端？依據是什麼？
- 即時性需求
  - 「即時呈現營運數據與分析報表」你會用 polling、SSE、WebSocket、還是 background revalidate？為什麼？ :contentReference[oaicite:6]{index=6}

---

## 5) 效能題（Web Vitals / p95 / UX）

- 你會怎麼找出 LCP / INP / CLS 的瓶頸？
- 圖表頁（數據分析圖表化）如何避免：
  - 初次載入過重（bundle 太大）
  - 圖表重算造成卡頓
  - 切換篩選條件時的多餘 re-render :contentReference[oaicite:7]{index=7}
- 你會如何設計「快取層級」：
  - 靜態資源（CDN）
  - 路由層（Next cache）
  - API 層（etag / cache-control）
  - 前端狀態層（React Query / SWR 的 cache）

---

## 6) 與後端協作、API 契約與錯誤模型（很常見）

- 你希望 API 回傳的錯誤格式是什麼？（error code / i18n message / field errors）
- 你會如何做型別安全？
  - OpenAPI/Swagger → 產型別？或 zod schema？如何避免前後端 drift？
- 上線後「需求常變」怎麼維護？
  - feature flag / gradual rollout
  - backward compatibility（舊 App/舊頁面仍在用）

---

## 7) 跟產品特性高度相關的情境題（命中率高）

> 這些題目直接對應官網描述的功能與模組。 :contentReference[oaicite:8]{index=8}

- 情境：庫存不足要「智慧化計算購買量」，並「透過 LINE 通知廠商採購項目」
  - 你會怎麼把「計算結果」做成可解釋 UI？（為什麼建議買這些、依據區間）
  - 通知送出前要怎麼做確認/預覽/撤回？失敗怎麼補送？
- 情境：助理使用意願低（官網 FAQ）
  - 你會怎麼用前端設計降低學習成本？（空狀態、引導、快捷操作、預設值）
- 情境：到院教學前先匯入資料（物料/員工/班別）
  - 你會怎麼設計「匯入成功率」與「錯誤可修復」的體驗？
- 情境：院長要看「提升物料使用率/減少行政時間」等 KPI
  - 你會如何設計 dashboard 的資訊層級與 drill-down？ :contentReference[oaicite:9]{index=9}

---

## 8) 你可以反問面試官的問題（顯得你很懂交付與維運）

- 前端目前主要痛點是：
  - 報表效能？表單流程複雜？權限/租戶隔離？還是 onboarding/匯入？
- Next.js 16 的使用深度到哪？
  - App Router / RSC / Server Actions 是否已落地？還是仍在 pages router？
- 資料即時性需求是什麼等級？
  - 哪些頁面需要秒級？哪些可以分鐘級？
- 有沒有既定的 design system / UI 規範？是否要支援多裝置（診所現場平板/手機）？ :contentReference[oaicite:10]{index=10}

---

## 9) 快速準備方向（你面試前 1–2 天可以做）

- 用「診所助理」視角把流程講順：
  - 領料 → 盤點 → 採購 → 進貨 → 報表
- 準備 2 個你最拿手的 Next.js 16 實戰案例：
  - 1 個偏「動態即時」頁（庫存查詢/審核）
  - 1 個偏「可快取」頁（報表/統計）
- 準備你如何處理：
  - 權限（RBAC）+ 租戶隔離（multi-tenant）
  - 錯誤回饋（表單 field error / 全域 error / retry）
  - Web Vitals 與 bundle 控制

---