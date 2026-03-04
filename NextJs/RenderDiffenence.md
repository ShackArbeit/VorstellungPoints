# Next.js 16（最新版）大型電商：哪些用 SSR / CSR / ISR（以及為什麼）（Markmap 版）

## 先建立 Next.js 16 的「新心智模型」
- Next.js 16 開啟 **Cache Components** 後（opt-in）
  - prerender 會先產出 **static HTML shell**（秒開）
  - 遇到「網路/DB/runtime data」時，**你必須明確選擇**
    - 用 `<Suspense>`：延後到 **request time**（等同 SSR 串流）
    - 用 `use cache` + `cacheLife`：把結果 **快取並可納入 shell**（等同 ISR/可重用的 cached SSR）
- 結論：在 Next 16，你更像是在做「區塊級別」的 SSR / ISR / CSR 組合，而不是整頁只能選一種
  - 這個模式是 PPR（Partial Prerendering）核心 :contentReference[oaicite:0]{index=0}

---

## 大型電商的頁面/模組拆解（推薦作法）

### A) 首頁（Home）
- 建議：**ISR + 局部 SSR/CSR**
- 哪些用 ISR（`use cache`）
  - 首頁主視覺/行銷版位（CMS 內容）
  - 熱門品、排行榜、主題館（更新頻率可控：分鐘~小時）
  - SEO 需要，且希望「所有人看到大致一致」→ 非個人化內容適合快取
- 哪些用 SSR（`<Suspense>`）
  - 個人化區塊：你可能喜歡、最近瀏覽、會員等級價（依 cookie/session）
  - 即時庫存警示（若你真的要每次都最新）
- 哪些用 CSR
  - 客戶端互動：輪播、追蹤曝光、A/B test（不影響 SEO 的互動）
- 為什麼
  - 首頁流量最大：ISR 讓 TTFB 穩、成本低；個人化再用 Suspense/CSR 補上

### B) 分類頁 / 列表頁（PLP: Category / Collection）
- 建議：**ISR（列表骨架+預設排序） + CSR（篩選/排序/分頁）**
- ISR（`use cache`）
  - 「分類頁初始列表」：例如前 24 筆、預設排序、分類描述（SEO 重要）
  - 可設 `cacheLife('minutes'|'hours')`，並用 tag 在商品上下架時 invalidation
- CSR
  - 篩選器（price slider / facet）、排序、無限捲動、快速切換 filter
  - 原因：filter 組合爆炸，全部用 SSR/ISR 會造成快取 key 失控、命中率很差
- SSR（可選）
  - 若你需要「每次請求都即時反映庫存/價格」且 SEO 很重要，可把「價格/庫存」做成 Suspense 子區塊（局部 SSR），列表本體仍 ISR

### C) 商品頁（PDP: Product Detail）
- 建議：**ISR（商品主資訊） + SSR（價格/庫存/促銷） + CSR（加入購物車互動）**
- ISR（`use cache`）
  - 商品名稱、描述、規格、圖片、FAQ（變動較低，SEO 關鍵）
  - 內容變更時用 tag 觸發 revalidate/update
- SSR（`<Suspense>`）
  - 即時價格（會員價/地區稅/活動折扣）
  - 即時庫存/可配送時間（履約需要「準」）
  - 這些通常依 request context 或變動頻繁，不適合直接進 shell
- CSR
  - variant 切換 UI、圖片 zoom、加入購物車動畫、收藏
- 為什麼
  - PDP 既要 SEO，又要正確交易資訊：用 PPR 把「可快取的」先塞進 shell，再把「一定要新鮮的」串流補上 :contentReference[oaicite:1]{index=1}

### D) 搜尋頁（Search）
- 建議：**SSR（或 SSR + CSR）為主**
- SSR（`<Suspense>` 或不使用 `use cache`）
  - 搜尋結果高度依賴 query（`searchParams`），而且常常要新鮮（熱門/庫存/價格）
  - query 組合也多，不適合全面 ISR
- CSR（可選）
  - 搜尋框即時建議、debounce、輸入時即時刷新（更好體驗）
- 為什麼
  - 搜尋的快取命中率通常差；SSR 能確保結果一致且即時
  - 注意：`searchParams` 是 runtime data，不能和 `use cache` 同 scope 混用（需把 runtime 值抽出再傳入 cached function） :contentReference[oaicite:2]{index=2}

### E) 購物車（Cart）
- 建議：**SSR / CSR（依架構）**
- SSR（推薦）
  - cart 通常依 cookie/session，且涉及價格計算與優惠
  - 用 `<Suspense>`：先出 shell（結帳按鈕、版面），cart items request-time 串流
- CSR（可行）
  - 若 cart 完全走 client storage + API（SPA 模式），可 CSR + SWR/React Query
- 為什麼
  - cart 是轉換關鍵：要「快」也要「準」
  - SSR 可減少 hydration 前的空白/閃爍

### F) 結帳流程（Checkout）
- 建議：**SSR 為主（不要 ISR）**
- SSR
  - 地址、運費、稅、付款方式、風控（都依 request / 交易狀態）
  - 避免 stale data：不使用 `use cache` 或只快取不影響交易的靜態資料（例如國家清單）
- CSR（輔助）
  - 表單互動、欄位驗證、分步 UI
- 為什麼
  - 高風險/高一致性區：stale = 交易錯誤 / 法規問題 / 客訴

### G) 會員中心（Account / Orders / Wishlist）
- 建議：**SSR + 少量 CSR**
- SSR
  - 訂單列表、個人資料（依 cookie/session）
  - `<Suspense>` 串流：頁面先顯示框架與導覽，資料再進來
- CSR
  - tab 切換、編輯個資、上傳頭像等互動
- 為什麼
  - 多為登入後內容：SEO 不重要，但體驗要穩定且安全

### H) 內容型頁面（Blog / Help / Policies / Landing）
- 建議：**ISR（甚至接近 SSG）**
- ISR（`use cache` + `cacheLife('hours'|'days')`）
  - 部落格、FAQ、退換貨政策、活動頁（非個人化）
- 為什麼
  - 幾乎全是 SEO + 穩定內容：快取命中率高、成本最低
  - ISR 的 revalidate/retry 行為也比較符合內容站需求 :contentReference[oaicite:3]{index=3}

---

## 「一頁三段式」範本（Next.js 16 最常見的電商最佳實務）

### 1) Static shell（自動 prerender）
- Header / Nav / Footer
- Skeleton / fallback UI（放在 Suspense fallback）

### 2) Cached blocks（ISR）
- `use cache` + `cacheLife`
- 典型：商品基本資料、分類初始列表、CMS 內容、推薦（非個人化）

### 3) Dynamic blocks（SSR / request time）
- `<Suspense>` 包住
- 典型：價格、庫存、配送 ETA、個人化推薦、cart、會員資料

---

## 決策規則（你可以拿去做系統設計審查）

### 1) 先問：SEO 重要嗎？
- 重要（首頁/PLP/PDP/內容頁）→ 優先讓「可快取的」進 shell（ISR）
- 不重要（會員中心/checkout）→ SSR/CSR 以正確性與互動為主

### 2) 再問：資料需要「每次請求都最新」嗎？
- 是 → SSR（不要 `use cache`）或 `<Suspense>` 串流
- 否 → ISR（`use cache` + `cacheLife`）+ tag invalidation

### 3) 最後問：快取 key 會爆炸嗎？
- 會（大量 filter / searchParams 組合）→ CSR 或 SSR（但不要全面 ISR）
- 不會（有限維度、命中率高）→ ISR

---

## Next.js 16 相關 API 對照（只放你會用到的）
- ISR（cached）
  - `use cache`
  - `cacheLife()`
  - `cacheTag()` + `revalidateTag()` / `updateTag()`
- SSR（request time）
  - 不使用 `use cache`
  - 或 `<Suspense>` 讓動態區塊 request-time 串流
- CSR
  - Client Component + SWR/React Query + router.refresh（視需求）

---

## 參考（建議搭配閱讀）
- Cache Components / PPR（Partial Prerendering）：:contentReference[oaicite:4]{index=4}
- `use cache` 指令：:contentReference[oaicite:5]{index=5}
- Caching & Revalidating：:contentReference[oaicite:6]{index=6}
- ISR Guide：:contentReference[oaicite:7]{index=7}