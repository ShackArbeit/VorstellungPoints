# 🔐 Secure RAG：保護敏感資料的 RAG 架構模板（README + Mermaid）

> 目的：在 RAG（Retrieval-Augmented Generation）中，降低敏感資料（PII/PHI/內部機密）外洩風險，同時保留可用性與合規性（GDPR/HIPAA/企業政策）。

---

## 📌 目標與原則

- **最小暴露（Least Exposure）**：能不進向量庫就不進
- **最小權限（Least Privilege）**：每個角色只能看到「該看到的」
- **可稽核（Auditable）**：所有檢索與回傳都能追蹤
- **可替換（Composable）**：可替換 LLM / Vector DB / DLP / IAM

---

## 🧩 核心元件（Components）

- **Client / UI**：使用者提問介面
- **API Gateway / BFF**：統一入口，做鑑權、速率限制、審計
- **IAM / AuthN/AuthZ**：身分驗證、角色/權限核發（JWT/OIDC/RBAC/ABAC）
- **DLP / PII Detector**：敏感資訊偵測與分類
- **Redactor / Masker**：遮罩、移除或替換敏感欄位
- **Embedding Service**：向量化（可自託管或雲服務）
- **Vector DB**：向量資料庫 + Metadata Filter
- **Retriever**：檢索（包含 metadata 過濾策略）
- **LLM**：生成答案
- **Audit Log / SIEM**：記錄與告警

---

## 🏗️ 架構總覽（Mermaid：Flowchart）

```mermaid
flowchart LR
  U[Users] -->|Ask| UI[Client/UI]
  UI --> API[API Gateway / BFF]

  API --> IAM[IAM / AuthN/AuthZ]
  IAM -->|JWT/Claims| API

  API --> DLP[DLP / PII Detector]
  DLP -->|Sensitive? yes/no + labels| API

  subgraph Ingestion[資料匯入（Ingestion）]
    DS[(Data Sources)] --> PRE[Preprocess/Clean]
    PRE --> DLP2[DLP / PII Detector]
    DLP2 --> REDACT[Redactor/Masker]
    REDACT --> EMB[Embedding Service]
    EMB --> VDB[(Vector DB + Metadata)]
  end

  API --> RET[Retriever]
  RET -->|Query + Claims| VDB
  VDB -->|TopK Chunks| RET

  RET --> LLM[LLM]
  LLM --> API
  API --> UI
  UI -->|Answer| U

  API --> LOG[Audit Log / SIEM]
  RET --> LOG
  VDB --> LOG