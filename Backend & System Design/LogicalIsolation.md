# Multi-tenant（多租戶）資料隔離取捨：Separate DB vs DB Schema vs Row-level
> 目標：用 **Next.js / FastAPI** 做企業系統時，讓「多個客戶（Tenant / 租戶）」共用同一套應用程式，但資料互不干擾。

---

## 1. 什麼是 Multi-tenant（多租戶）？
- **Tenant（租戶）**：一個客戶/組織/公司，例如「A 公司」「B 公司」。
- **Multi-tenant（多租戶架構）**：
  - **同一套系統（Same app）** 服務多個租戶
  - 每個租戶的 **資料（Data）**、**設定（Config）**、**權限（Authorization）** 必須隔離（Isolation）
- 用生活化比喻：
  - 一棟大樓（你的系統）有很多住戶（Tenant）
  - 每戶的房間（資料）要有門禁（隔離），不能走錯門就看到別人家

---

## 2. 三種常見資料隔離策略（Data Isolation Strategies）
下面三種不是「誰一定最好」，而是 **安全需求、成本、維運能力** 的取捨。

### A) Separate Database（獨立資料庫）
**做法**：每個 Tenant 一個獨立 DB（甚至獨立 DB Server）。  
- 👍 優點
  - **隔離最強（Strongest isolation）**：天生不容易「查到別人資料」
  - **備份/還原（Backup/Restore）**、**資料搬移（Migration）**、**刪除客戶（Tenant offboarding）** 很直覺
  - 合規（Compliance）/ 高敏感資料（High sensitive data）較容易達成
- 👎 缺點
  - **成本最高（Highest cost）**：DB 數量多、監控/備份/升級都放大
  - **維運較重（Operational overhead）**：連線池、權限、版本、指標都要照顧很多套
- 適合
  - 大客戶（Enterprise）
  - 金融/醫療/司法等高敏感或合規要求高
  - 要支援「客戶自帶 DB / on-prem（內網部署）」

---

### B) Separate Schema（同一 DB、不同 Schema / 命名空間）
**做法**：同一個 DB 裡，每個 Tenant 一個 Schema（例如 PostgreSQL schema）。  
- 👍 優點
  - **隔離比 Row-level 強**（結構上分開）
  - 成本比 Separate DB 低：只有一套 DB 服務，但有多個 Schema
  - 每個租戶可做「較容易的備份/匯出」（針對 schema）
- 👎 缺點
  - DB 還是共用一套：**資源競爭（Resource contention）** 仍存在
  - Migrations（資料庫版本變更）會更複雜：每個 schema 都要跑一次
  - 不同資料庫對 schema 支援差異大（例如 MySQL 沒有真正的 schema 概念，多用 database 當 schema）
- 適合
  - 中型 SaaS
  - 需要「比 Row-level 更強隔離」但又不想爆炸式 DB 數量
  - PostgreSQL 為主的團隊較舒服

---

### C) Row-level（同一套表，用 tenant_id 做列級隔離）
**做法**：所有租戶共用同一套 Table（表格），每筆資料都有 `tenant_id`，查詢時永遠帶入條件：`WHERE tenant_id = ?`。  
- 👍 優點
  - **成本最低（Lowest cost）**：只有一套 DB、表、索引
  - **最容易做報表/跨租戶統計**（如果你允許）
  - 上線/擴租戶最快：新增租戶通常只是新增一筆設定資料
- 👎 缺點（這些是最容易出事的地方）
  - **資料外洩風險最高（Highest data leak risk）**：只要某段查詢漏加 tenant filter 就可能看到別人資料
  - **刪除租戶資料**、**獨立備份** 較麻煩
  - 大客戶可能要求更高隔離，Row-level 不一定能滿足
- 適合
  - 早期產品、MVP、租戶數多但每個租戶資料量不大
  - 團隊能嚴格落實「所有查詢都自動加 tenant 條件」的工程紀律

> 延伸：PostgreSQL 有 **RLS（Row Level Security）**，可用 DB 規則強制隔離（更安全）。

---

## 3. 一張表看取捨（Decision Matrix）
- 你可以把它當「挑房型」：安全越高通常越貴、越難維運。

| 策略（Strategy） | 隔離強度（Isolation） | 成本（Cost） | 維運（Ops） | 擴展性（Scale） | 最常見風險（Risk） |
|---|---:|---:|---:|---:|---|
| Separate DB（獨立資料庫） | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐～⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐～⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | DB 數量爆炸、維運成本高 |
| Separate Schema（獨立 Schema） | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | migration 複雜、資源仍共用 |
| Row-level（列級隔離） | ⭐⭐～⭐⭐⭐ | ⭐ | ⭐～⭐⭐ | ⭐⭐⭐⭐⭐ | 忘記加 tenant filter 導致外洩 |

---

## 4. 怎麼選？用「需求等級」倒推（Rule of Thumb）
### 先問 3 個問題（3 Key Questions）
1. **資料敏感度（Data sensitivity）**：有沒有個資、金流、司法案件、醫療？
2. **客戶等級（Customer tier）**：是否有「大客戶願意付更高費用」？
3. **維運能力（Operational maturity）**：你們有沒有能力維運大量 DB / schema？（監控、備份、升級、權限）

### 快速建議（Practical Recommendations）
- **早期產品 / 小團隊 / 快速迭代**：先用 **Row-level + 嚴格防呆**  
- **中型 SaaS / 要平衡隔離與成本**：用 **Separate Schema**（偏 PostgreSQL）  
- **高敏感/合規/大客戶**：用 **Separate DB**（甚至 per-tenant K8s namespace + DB）

### 常見的「混合策略（Hybrid）」
- 大多數租戶：Row-level
- VIP/高敏感租戶：Separate DB
- 同一套程式碼支援多模式，靠設定（Tenant tier / Plan）切換

---

## 5. 用 Next.js / FastAPI 時，關鍵不是框架，而是「Tenant 解析 + DB 路由」
### 5.1 Tenant Resolution（租戶辨識）怎麼做？
常見來源（你選一個主方案，其他當備援）：
- **Subdomain（子網域）**：`acme.yourapp.com` → tenant = `acme`
- **Custom Domain（自訂網域）**：`app.acme.com` → tenant = `acme`
- **Header（自訂標頭）**：`X-Tenant-Id: acme`（多用於內網或 API）
- **JWT Claims（Token 內的租戶欄位）**：`tenant_id`

建議：
- 對外 Web 產品常用 **Subdomain + JWT**（雙重確認）
- 內部系統/整合 API 常用 **Header + JWT**

---

## 6. 三種隔離策略在 FastAPI 的落地方式（Implementation Patterns in FastAPI）
### A) Row-level：用 DB Session / ORM 自動加 tenant filter
核心做法：
- Request 進來 → 解析 tenant → 放進 `request.state.tenant`
- ORM 查詢時自動加 `tenant_id = current_tenant`
- **最重要：把 tenant filter 做成「預設行為」**，不要靠每個工程師記得加

常見配套：
- 所有表都有 `tenant_id`（或 `organization_id`）
- Index（索引）要包含 `(tenant_id, ...)`，否則查詢會慢
- PostgreSQL 可啟用 **RLS**，讓 DB 層強制隔離（建議有能力就上）

---

### B) Separate Schema：每個 Request 切換 schema / search_path
核心做法：
- Request 進來 → tenant → 對應 schema 名稱（例如 `tenant_acme`）
- DB 連線建立後設定 `search_path`（PostgreSQL）或用不同 DB 名稱（MySQL）
- Migration：要對每個 schema 跑一次（可自動化）

風險提醒：
- 連線池（Connection Pool）要注意：不能把「切過 schema 的連線」借給另一個 tenant
  - 通常做法是：每次借出連線後先設定 schema，歸還前重置

---

### C) Separate DB：用「Tenant → Connection string」做連線路由
核心做法：
- Tenant 設定表（Tenant registry）存每個租戶的 DB 連線資訊
- Request 進來 → tenant → 取得對應 DB → 建立/取得連線池
- Migration：每個 DB 一次，但可以批次自動跑

風險提醒：
- 連線池數量會變多，需要監控（Monitoring）與上限（Limit）
- 租戶數非常大時，要做「惰性建立（Lazy init）」與 LRU cache

---

## 7. Next.js 在 multi-tenant 常做什麼？
> Next.js 多半處理「租戶辨識、UI 設定、路由」，資料隔離主要在後端 DB。

常見做法：
- Middleware 讀取 Host header 解析 subdomain / domain
- 把 tenant info 放到：
  - cookies（例如 `tenant=acme`）
  - request headers（轉發到 API）
  - 或在 Next.js server actions / API route 直接呼叫後端時帶上 tenant

UI 常見差異（Per-tenant customization）：
- Logo、主題色（Theme）、功能開關（Feature flag）
- 權限（RBAC/ABAC）與方案等級（Plan/Tier）

---

## 8. 最容易踩雷的 8 件事（Common Pitfalls）
1. 忘記 tenant filter（Row-level 最致命）
2. 權限（Authorization）只做 user，沒做 tenant（要 user + tenant + role）
3. 背景工作（Background job / Celery / RQ）沒帶 tenant context
4. Cache（Redis / CDN）key 沒包含 tenant（會串資料）
5. Log/Trace 沒帶 tenant_id（出事很難查）
6. Migration 沒策略（Schema/DB 會痛苦）
7. 備份/還原流程沒演練（Disaster recovery drill）
8. 測試沒有做「跨租戶隔離測試」（Isolation tests）

---

## 9. 實務落地的「建議清單（Checklist）」### 必備（Must-have）
- [ ] Tenant resolution 有單一權威來源（Host/Header/JWT）
- [ ] 後端每個 request 都能取得 current tenant（例如 dependency / middleware）
- [ ] DB 層有保護：RLS（PostgreSQL）或至少 ORM 層自動 tenant filter
- [ ] Cache key 一律包含 tenant_id
- [ ] 所有 audit log / tracing 會記 tenant_id

### 可選加強（Nice-to-have）
- [ ] VIP tenant 切 Separate DB（Hybrid）
- [ ] 每個 tenant 的資源配額（Quota / Rate limit）
- [ ] per-tenant encryption key（KMS / Envelope encryption）

---

## 10. 名詞對照（Glossary: 中文 / English）
- 多租戶：Multi-tenant
- 租戶：Tenant
- 資料隔離：Data isolation
- 獨立資料庫：Separate Database (Separate DB)
- 獨立 Schema：Separate Schema
- 列級隔離：Row-level isolation
- 列級安全：Row Level Security (RLS)
- 權限：Authorization
- 身份驗證：Authentication
- 子網域：Subdomain
- 自訂網域：Custom Domain
- 連線池：Connection Pool
- 資源競爭：Resource contention
- 資料外洩：Data leak
- 混合策略：Hybrid strategy
- 功能開關：Feature flag
- 方案等級：Plan / Tier
- 備份/還原：Backup / Restore
- 遷移：Migration

---

## 11. 一句話總結（One-liner Summary）
- **Row-level**：最快最省，但要工程紀律（最怕漏加 tenant filter）  
- **Schema**：隔離與成本折衷，migration 較麻煩  
- **Separate DB**：隔離最強、合規最好，但維運成本最高
