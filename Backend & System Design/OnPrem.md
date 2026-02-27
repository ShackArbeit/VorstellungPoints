# 企業 On‑Prem AI 服務部署方案（NestJS / FastAPI）
> 目的：用**內網（on‑premises）**環境，把「聊天/問答、RAG、模型推論」這類 AI 服務安全、可維運地跑起來。  
>（英文對應：On‑Premises Deployment Plan for AI Services）

## 1) 你要先決定的三件事
- **環境型態**（Environment Type）
  - 單機 / VM（Single Server / VM）
  - 多節點叢集（Cluster）
- **網路限制**（Network Constraints）
  - 可上網（Internet‑connected）
  - 只允許白名單（Allowlist / Whitelist）
  - 完全離線（Air‑gapped / Offline）
- **模型策略**（Model Strategy）
  - 離線自建模型（On‑prem Model / Self‑hosted Model）
  - 外部 LLM 代理（External LLM via Gateway / Proxy）

---

## 2) 推薦的整體架構（高層、好理解）
- **API 服務層**（API Service Layer）
  - NestJS 或 FastAPI：負責登入、權限、請求驗證、串接 RAG/模型、回傳結果  
  -（Auth / RBAC / Validation / Orchestration）
- **模型推論層**（Inference Layer）
  - 跑你自己的模型（GPU/CPU）或連到企業允許的外部模型
- **知識與資料層**（Knowledge & Data Layer）
  - 文件庫（Document Store）：原始檔（PDF/Word/HTML）
  - 向量資料庫（Vector DB）：做語意檢索（Semantic Search）
  - 關聯資料庫（Relational DB）：存使用者、權限、對話紀錄、稽核
- **觀測與稽核層**（Observability & Audit Layer）
  - 日誌（Logs）、指標（Metrics）、追蹤（Tracing）、稽核（Audit Log）

> 你可以把它想成：**API 是總機**、**模型是專家**、**知識庫是圖書館**、**監控是保全與儀表板**。

---

## 3) 部署方案選擇：Docker vs K8s
### A. Docker（單機/小規模最常見）
- 適合（Fit For）
  - PoC / 小團隊 / 低併發（Low Concurrency）
  - 只有 1–3 台主機
- 做法（How）
  - 使用 `docker compose` 管理：API、DB、Vector DB、Inference、監控
- 優點（Pros）
  - 上線快、成本低、好理解
- 缺點（Cons）
  - 高可用（HA, High Availability）與彈性擴縮（Autoscaling）較弱

### B. Kubernetes（K8s，多節點/企業級標準）
- 適合（Fit For）
  - 中大型組織、需要 HA、需要彈性擴縮、需要標準化治理
- 做法（How）
  - 用 Helm / Kustomize 部署：API、推論服務、向量庫、監控、Ingress
- 優點（Pros）
  - 可靠性高（Resilience）、可滾動更新（Rolling Update）、資源調度好
- 缺點（Cons）
  - 學習與維運成本較高（Ops Cost）

> **選擇建議（Rule of thumb）**：  
> - 你們只有少量使用者 + 先要能跑：**Docker** ✅  
> - 你們要做企業內部平台、多人使用、要穩：**K8s** ✅

---

## 4) 內網限制下的關鍵處理（On‑prem / Air‑gapped）
- **鏡像來源**（Container Image Source）
  - 建議設內部映像倉庫（Private Registry），例如 Harbor / Nexus
  - 離線環境用「搬運」方式：匯出映像（Image Export）→ 內網匯入（Import）
- **套件與依賴**（Dependencies）
  - Node（NestJS）與 Python（FastAPI）建議使用鎖版（Lockfile）與內部鏡像站（Mirror）
- **模型檔案**（Model Artifacts）
  - 模型權重、Tokenizer、Embedding 模型、Reranker：都要可離線部署
- **時間同步**（Time Sync）
  - 稽核與憑證很依賴時間：內網 NTP（Network Time Protocol）要設好
- **憑證與 TLS**（Certificates & TLS）
  - 內網 CA（Internal CA）與憑證更新流程要可控（Rotation）

---

## 5) LLM 策略：離線模型 vs 外部 LLM Gateway
### A. 離線模型（On‑prem Model）
- 適合（Fit For）
  - 資安敏感資料不能外流（Data Residency / Confidentiality）
  - 內網完全離線（Air‑gapped）
- 典型組合（Typical Stack）
  - 推論服務（Inference Server）：vLLM / TGI / llama.cpp（依硬體）
  - Embedding（向量化）模型：本地部署
- 你要面對的現實（Operational Reality）
  - GPU 資源、版本更新、效能調校（Tuning）、成本

### B. 外部 LLM Gateway（外部模型的「企業代理」）
- 核心概念（Key Idea）
  - **你的系統永遠只連到「內部 Gateway」**，Gateway 再去管外部模型  
  -（LLM Gateway / Proxy / Broker）
- Gateway 需要做什麼（Gateway Responsibilities）
  - 供應商切換（Provider Switching）
  - 請求記錄與稽核（Audit）
  - 速率限制（Rate Limit）與配額（Quota）
  - 脫敏（Redaction / Masking）與 DLP（Data Loss Prevention）
  - 快取（Cache）與回退策略（Fallback）
- 適合（Fit For）
  - 公司允許特定外部連線（Allowlist）且想快速享受更強模型

> **一句話**：  
> - 離線模型 = 你自己養一位專家（成本高但資料最安全）  
> - Gateway = 你找外部顧問，但先經過公司門禁與記錄（更彈性）

---

## 6) Config 管理（Configuration Management）
> 目標：**同一套程式**在「開發/測試/正式/內網」用不同設定，且不改程式碼。

- **分層配置**（Layered Config）
  - Base（共用） + Env（環境） + Secret（機密）
- **常見做法**（Common Practices）
  - Docker：`.env` + `config.yaml`
  - K8s：ConfigMap + Helm values
- **需要明確列出的設定**（What to Configure）
  - DB 連線（DB Connection）
  - 向量庫連線（Vector DB Endpoint）
  - LLM / Embedding 端點（LLM/Embedding Endpoint）
  - 功能開關（Feature Flags）
  - 逾時與重試（Timeout / Retry）
  - 併發限制（Concurrency Limit）
  - 日誌等級（Log Level）

---

## 7) Secret 管理（Secrets Management）
> 原則：**機密不要寫進程式碼、不要寫進 Git、不要寫進映像**。

- **你會遇到的 Secret**（Typical Secrets）
  - DB 密碼（DB Password）
  - API Key（外部模型、SaaS）
  - JWT/Session 金鑰（Signing Keys）
  - TLS 私鑰（Private Keys）
- **存放方式（從簡到嚴）**
  - Docker：環境變數（Environment Variables）+ 受管控的 `.env`（權限鎖死）
  - K8s：Secret + RBAC + Encryption at rest
  - 企業更成熟：Vault（HashiCorp Vault）或雲端 KMS（Key Management Service）
- **輪替機制**（Rotation）
  - 設定到期/輪替週期 + 不中斷更新（Zero‑downtime Rotation）

---

## 8) 交付與維運：企業最在意的清單
- **可觀測性（Observability）**
  - Logs（應用日誌）、Metrics（CPU/RAM/GPU/QPS/Latency）、Tracing（跨服務追蹤）
- **稽核（Auditability）**
  - 誰在什麼時間問了什麼（Who/When/What）
  - 回答引用了哪些資料（Citations）
- **安全（Security）**
  - 身分驗證（Authentication）+ 權限（Authorization / RBAC）
  - 內網分區（Network Segmentation）
  - 輸入檢查（Input Validation）避免 Prompt Injection
- **版本與更新（Release & Patch）**
  - 映像版本（Image Tagging）
  - 回滾（Rollback）
  - 依賴漏洞掃描（Vulnerability Scanning）
- **備援與備份（Backup & DR）**
  - DB、向量庫、文件庫的備份策略（Backup Policy）
  - 災難復原（Disaster Recovery, DR）演練

---

## 9) 你可以直接照抄的「最小可行」落地方案（MVP Blueprint）
- Docker（Compose）
  - `api`（NestJS/FastAPI）
  - `db`（PostgreSQL/MySQL）
  - `vector`（Qdrant/Milvus/pgvector 擇一）
  - `inference`（vLLM/TGI/llama.cpp 擇一）
  - `monitoring`（Prometheus + Grafana）
  - `logs`（Loki 或 ELK 擇一）
- 內網配套
  - 私有映像倉庫（Private Registry）
  - `.env` 與 Secret 管理流程
  - 基本稽核（Audit Log）表格/索引

---

## 10) 名詞對照（Glossary, 中英）
- On‑Premises：地端/內部部署
- Air‑gapped：完全離線（與外網隔離）
- Private Registry：私有映像倉庫
- Ingress / Reverse Proxy：入口閘道/反向代理
- HA (High Availability)：高可用
- Rolling Update：滾動更新
- ConfigMap / Secret：設定/機密物件（K8s）
- LLM Gateway / Proxy：大模型代理/閘道
- DLP (Data Loss Prevention)：資料外洩防護
- Observability：可觀測性
- Audit Log：稽核日誌
- DR (Disaster Recovery)：災難復原

