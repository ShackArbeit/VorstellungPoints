# Next.js 16 Cache（Cache Components / PPR）重點整理 + 與 2024 Blogger 文章差異（Markmap 版）

## 目標（你要記住的 1 句話）
- Next.js 16 開啟 **Cache Components** 後：  
  - **預設：所有動態資料都在 Request time 跑**  
  - **想快取：你必須顯式標註（opt-in）**  
  - **一條 route 內可混合：Static（直接進 shell）+ Cached（進 shell）+ Dynamic（Suspense 串流）**

---

## Next.js 16（Cache Components）核心概念

### 1) 這是 opt-in 功能（要先開啟）
- 在 `next.config.*` 設定：
  - `cacheComponents: true`
- 目的：讓「**資料抓取/動態工作**」在 prerender 時 **預設被排除**，避免你以為它被快取、實際卻是另一套隱式規則

### 2) 新的渲染心智模型：Static shell + Partial Prerendering（PPR）
- Build time（prerender）會先生成：
  - **static HTML shell**（先讓畫面可立即顯示）
  - **序列化的 RSC payload**（用於 client-side navigation）
- 遇到「無法在 prerender 完成」的內容（例如：network、DB、runtime data），你必須明確處理：
  - A. 用 `<Suspense>`：延後到 request time，先顯示 fallback（fallback 會進 shell）
  - B. 用 `use cache`：讓其結果可被快取並納入 shell（前提：不依賴 request context）

### 3) 什麼會「自動」進 static shell？
- 可在 prerender 完成且不需要 request context 的工作，例如：
  - 同步 I/O、純計算、import、可在 build 階段決定的內容
- 直覺：**不碰網路、不碰 runtime APIs、不需要每個請求都不同的值** → 容易進 shell

### 4) 什麼一定要「顯式」處理？
- Dynamic content（外部系統 / 網路 / DB / async I/O）
  - 選擇：
    - `<Suspense>`：request time 串流（想要每次都最新）
    - `use cache`：快取納入 shell（允許短時間內一致）
- Runtime data（依賴 request context）
  - 例如：`cookies()`、`headers()`、`searchParams`、`params`（動態路由在特定條件下）
  - 規則：**Runtime data 不能在同一個 scope 內直接 `use cache`**
  - 正確做法：先在非 cached scope 取 runtime data → 再把值「傳入」cached function/component

### 5) 非決定性（non-deterministic）操作要「表態」
- `Math.random()` / `Date.now()` / `crypto.randomUUID()` 這類：
  - 想要每個 request 都不同：必須先「明確延後到 request time」
  - 想要大家都看到同一份（直到 revalidate）：可放進 `use cache` scope

---

## Next.js 16 的 Cache API（你實務上最常用的）

### 1) `use cache`（核心主角）
- 可標註：
  - route（檔案層級）
  - component（函式層級）
  - function（函式層級）
- 作用：把「回傳結果」快取起來（包含 render 結果 / 資料抓取 / DB query / 計算）

### 2) `cacheLife()`（時間型快取策略）
- 配合 `use cache` 使用
- 你可以：
  - 用 profile（hours/days/weeks…）
  - 或自訂 `{ stale, revalidate, expire }`

### 3) Tag-based（事件型快取策略）
- `cacheTag(tagName)`：替 cached scope 標籤
- `revalidateTag(tagName)`：讓「被標籤的快取」進入 stale-while-revalidate（容忍延遲更新）
- `updateTag(tagName)`：同一個 request 內「立刻過期 + 立刻刷新」（常用在 Server Actions 寫入後希望馬上讀到新資料）

### 4) `<Suspense fallback>`（把動態區塊切開）
- 原則：**Boundary 盡量靠近真正需要 request time 的區塊**
- 好處：
  - shell 最大化（頁面秒開）
  - 多個動態區塊可並行串流（不要互相阻塞）

### 5) Navigation：Activity（狀態保留）
- 開啟 Cache Components 後，Next.js client navigation 會使用 React 的 Activity 機制：
  - route 可能被「hidden」而非立刻 unmount
  - state（表單、展開狀態）更容易保留
  - effects 會在 hidden 時清理、恢復可見時重建

---

## 與 2024 Blogger 文章（Web Dev Simplified）差異比較

### 1) 最大差異：從「隱式預設快取」→「顯式 opt-in 快取」
- Blogger 文章的核心框架是：
  - Request Memoization
  - Data Cache
  - Full Route Cache
  - Router Cache
- 但 Next.js 16（Cache Components）把重點往這裡移：
  - **prerender 遇到動態工作就停**
  - 你必須選：
    - `<Suspense>`（request time）
    - `use cache`（cache 並可納入 shell）
- 結果：Next 16 更像「你決定快取策略」，而不是「你被快取規則牽著走」

### 2) Blogger 的四層 cache 觀念：哪些仍然有用？
- Request Memoization（同一次 render pass 去重）：
  - 仍是重要概念（React/Server Components 層級），理解它仍有助於避免重複抓取
- Router Cache（client-side navigation 的 payload 快取）：
  - 仍存在，但 Next 16 額外要加上 **Activity / hidden 保狀態** 的理解

### 3) Blogger 文章中「需要更新/容易誤導」的地方（重點清單）
- A. 「server `fetch` 預設跨 request 持久快取」這種寫法要改得更精準  
  - Next 16 的重點不是“fetch 預設快取有多兇”，而是「你要不要把它納入 prerender shell」要用 `use cache` / `<Suspense>` 表態
- B. `unstable_cache` 的定位要降級  
  - Next 16 文件把 `use cache` 放到主角位置（而不是 `unstable_cache` 當主力）
- C. route segment configs（舊時代常用的）需要改寫/刪除或換新做法  
  - `dynamic = 'force-dynamic'`：在 Cache Components 模式下不再是核心手段（因為預設就偏 request time）
  - `revalidate`：應改成 `cacheLife()`
  - `fetchCache`：在 `use cache` scope 內不再需要用它控制
  - `runtime = 'edge'`：Cache Components 需要 Node runtime（Edge 不支援）
- D. Tag-based 的內容需要補上新版 API 與語意差異  
  - 補 `cacheTag`、`updateTag`，並釐清 `revalidateTag` vs `updateTag` 的使用時機
- E. Router Cache 章節要補上 Navigation Activity 對 UI state 的影響  
  - 否則讀者會困惑「為什麼回上一頁狀態還在 / effect 重新跑」

---

## 實務落地：你可以怎麼寫一個「又快又新鮮」的頁面（心智模板）

### 1) 先切三種區塊
- Static：永遠可 prerender → 直接進 shell（navbar、標題、layout…）
- Cached：可接受短時間一致 → `use cache` + `cacheLife`（產品清單、文章內容、排行榜…）
- Dynamic：每次 request 都要最新或依賴 runtime → `<Suspense>`（購物車、個人化偏好、即時狀態…）

### 2) runtime data 的正確流向
- runtime scope 取：cookies/headers/searchParams
- 把值作為參數丟給 cached function/component（避免同 scope 混用 runtime + use cache）

### 3) mutation 後更新策略（最常踩坑）
- 寫入後希望「同 request 立刻看到」：`updateTag`
- 寫入後可容忍延遲更新：`revalidateTag`
- 一律先養成習慣：cached scope 都打 `cacheTag`，不然你不知道自己在 invalidating 什麼

---

## Blogger 文章若要「更新成 Next.js 16 版」的建議章節結構

### 1) 先新增一章：Cache Components 的新心智模型
- opt-in：`cacheComponents: true`
- PPR：static shell + 串流
- 顯式處理：`<Suspense>` vs `use cache`

### 2) 重新編排原本四大快取
- 把「Full Route Cache」從主角降級成結果（Outcome）
  - 你的顯式選擇（Suspense/use cache）才是主因
- 把「Data Cache」與「use cache」關係講清楚
  - 尤其是：哪些內容能進 shell、哪些只能 request time

### 3) Migration 專區（直接對照舊 config）
- revalidate → cacheLife
- fetchCache → use cache scope
- force-* dynamic → 先拿掉，用錯誤提示引導你補 Suspense / use cache
- Edge runtime → 說明限制

---

## 你可以拿這份文件做什麼
- 直接存成 `README.md`
- 在 VSCode 用 markmap 把層級轉成心智圖