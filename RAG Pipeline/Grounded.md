
# RAG 如何做到「可引用、可稽核」
（How RAG Becomes Grounded & Auditable）

---

# 一、先用生活例子理解

## 1️⃣ RAG 是什麼？（Retrieval-Augmented Generation）
簡單說：
👉 不是憑空回答  
👉 是「先查資料，再回答」

就像你寫報告：
- 先去圖書館找書
- 找到段落
- 再寫出結論
- 最後附上參考來源

這就叫做：
**Grounding（根據真實來源回答）**

---

# 二、什麼叫「可引用」？（Citations）

企業希望的是：

- 回答裡每一個重要結論
- 都可以指出「是哪份文件」
- 而且是「哪一段原文」

不是只說：
❌ 來源：公司政策

而是要說：

✅ 文件：資料外部分享作業規範 v3.2  
✅ 第 12 頁  
✅ 原文片段（Snippet）  
✅ 文字位置（Span）

---

# 三、關鍵名詞解釋（中英對照）

## Grounding（依據來源回答）
答案必須來自檢索到的文件，而不是模型自己猜。

## Citations（引用標註）
在答案旁標註 [1][2]，每個編號都能對應到來源。

## Source ID（來源識別碼）
每份文件都有一個唯一 ID。

## Snippet（引用片段）
被引用的「原文摘錄」。

## Span（文字範圍）
這段話在原文件的哪個位置（例如第幾頁、第幾個字元）。

## Manifest（版本清單）
這次建立索引時，用了哪些文件版本。

## Evidence Chain（證據鏈）
從「答案」一路追溯到「原始文件」的完整過程。

---

# 四、企業為什麼這麼在意？

因為企業怕三件事：

1️⃣ 模型亂說話  
2️⃣ 找不到來源  
3️⃣ 無法驗證是否被修改  

所以企業需要：

- 可追溯（Traceability）
- 可重現（Reproducibility）
- 可驗證（Integrity / Hash）
- 可稽核（Auditability）

---

# 五、RAG 的「證據鏈」長什麼樣？

一個完整流程應該是：

使用者提問  
→ 系統去資料庫找相關文件  
→ 找到幾個片段（Chunks）  
→ 用這些片段產生答案  
→ 每一段答案都標示引用  
→ 引用能對應到原文片段（Snippet + Span）  
→ 紀錄版本與時間（Manifest + Timestamp）  

這整條就叫做：

Evidence Chain（證據鏈）

---

# 六、回答的標準格式應該包含什麼？

一個「可稽核回答」至少要有：

- query_id（這次問題編號）
- answer（回答內容）
- citations（引用清單）
- source_id（來源文件）
- snippet（原文）
- span（原文位置）
- index_version（索引版本）
- manifest_id（資料版本）
- model_version（模型版本）

這樣才算企業級。

---

# 七、Next.js 與 NestJS 在這裡做什麼？

## 🟢 Next.js（前端顯示層）

角色：證據展示平台（Evidence Viewer）

負責：
- 顯示答案
- 顯示 [1][2] 引用
- 點擊後顯示原文 Snippet
- 跳轉到文件頁碼
- 高亮引用段落
- 權限不足時隱藏內容

---

## 🔵 NestJS（後端控制層）

角色：證據管理與稽核中心

負責：
- 強制所有回答必須附引用
- 過濾使用者權限（RBAC / ABAC）
- 記錄查詢日誌（Audit Log）
- 記錄索引版本（Index Version）
- 驗證文件完整性（Hash Check）
- 提供版本回滾（Rollback）

---

# 八、用一句話總結

RAG 的「可稽核」就像：

寫報告一定要附參考資料，  
而且還要：

- 告訴你哪本書  
- 哪一頁  
- 哪一段  
- 哪個版本  
- 什麼時間建立  

這樣主管或審查單位才能相信你。

---

# 九、給面試時可以說的關鍵句

- 我們把答案拆成段落，每段都綁定 citations。
- 每個 citation 包含 source_id、snippet、span 與版本資訊。
- 我們使用 manifest 來確保資料版本可回滾。
- NestJS 負責稽核與權限控管。
- Next.js 負責引用展示與高亮顯示。
- 整體形成 evidence chain。

---

# 十、核心概念心智圖關鍵詞（Markmap 友善）

RAG  
- Grounding  
- Citations  
- Snippet  
- Span  
- Source ID  
- Manifest  
- Evidence Chain  
- Audit Log  
- Integrity  
- Traceability  
