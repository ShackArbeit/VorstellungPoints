# NestJS + FastAPI Architecture Responsibility Map

---

## 1. API Gateway Layer (NestJS)

### Role
- 作為單一入口 (Single Entry Point)
- 負責:
  - Routing
  - JWT/OAuth2 驗證
  - RBAC (Role-based Access Control)
  - CORS
  - Rate Limiting
  - Swagger API 文件

### Why NestJS
- TypeScript 型別安全
- 模組化架構
- Guard + Interceptor 設計清晰
- 適合作為 API Gateway / BFF

### Risks
- 單點瓶頸 (Single Point Bottleneck)
- 必須：
  - Horizontal Scaling
  - 使用 Load Balancer
  - 開啟 Cluster Mode

📚 Source:
- NestJS Official Docs
- Node.js Cluster Docs

---

## 2. Auth / Security Layer

### NestJS
- 使用 Passport + JWT Strategy
- Guard 控制權限
- 可實作 RBAC

### FastAPI
- 可快速驗證 API Key
- OAuth2PasswordBearer
- 適合做模型服務驗證

### Risks
- 跨語言 session 管理
- Token format 必須統一
- 建議使用：
  - Stateless JWT
  - 共用 Redis Session Store

📚 Source:
- NestJS Security Docs
- FastAPI Security Tutorial

---

## 3. AI Inference Service (FastAPI)

### Role
- LLM 推論
- 向量檢索
- RAG Pipeline
- 知識圖查詢

### Why FastAPI
- Python AI 生態：
  - PyTorch
  - Transformers
  - LangChain
  - Pandas
- ASGI + Async 支援高併發 I/O

### Risks
- GIL 限制 CPU-bound 任務
- 模型初始化時間長
- 建議：
  - 使用 Multiprocess Workers
  - 使用 Celery 處理 heavy task
  - 模型 preload

📚 Source:
- FastAPI Docs
- Python GIL 官方文件

---

## 4. Data Processing / ETL Layer (FastAPI)

### Role
- 批次資料清洗
- 向量化處理
- 圖譜構建
- Embedding Pipeline

### Why Python
- Pandas
- NumPy
- PyTorch
- HuggingFace
- Neo4j Driver

### NestJS Role
- 觸發 ETL 任務
- 發送 Message Queue
- 控制排程

### Risks
- 長時間阻塞
- 需避免阻塞 API thread
- 建議：
  - Background Worker
  - Celery + Redis
  - 任務 Queue 分離

📚 Source:
- Celery Official Docs
- FastAPI Background Tasks

---

## 5. Database Layer

### NestJS
- RDBMS (Postgres/MySQL)
- TypeORM / Prisma
- Transaction 管理清晰

### FastAPI
- Vector DB:
  - Pinecone
  - Weaviate
- Graph DB:
  - Neo4j

### Risks
- 異質資料庫 (Polyglot Persistence)
- 需統一：
  - Connection Pool
  - Timeout 設定
  - Retry Policy

📚 Source:
- TypeORM Docs
- Prisma Docs
- Pinecone Docs
- Neo4j Docs

---

## 6. Batch Processing

### NestJS
- Cron Module
- BullMQ (Redis Queue)

### FastAPI
- Celery
- APScheduler
- Multiprocessing

### Risks
- 批次任務與 API 爭搶資源
- 建議：
  - Separate Worker Node
  - Resource Limitation
  - Kubernetes Job

📚 Source:
- NestJS Schedule Module
- Celery Docs

---

## 7. Monitoring / Logging

### NestJS
- Winston
- Morgan
- Prometheus Exporter

### FastAPI
- Python logging
- Prometheus client
- OpenTelemetry

### Risks
- 指標格式不同
- Log 格式不統一
- 建議：
  - 統一 Log Schema (JSON)
  - 使用集中式監控：
    - Prometheus
    - Grafana
    - ELK

📚 Source:
- Prometheus Docs
- OpenTelemetry Docs

---

## 8. CI/CD & Deployment

### NestJS
- Dockerize
- CI Pipeline (GitHub Actions)
- Build → Test → Release

### FastAPI
- Dockerize + ML Model
- 需考慮：
  - Model Size
  - Cold Start
  - Memory Usage

### Risks
- FastAPI 容器體積大
- 模型初始化慢
- 建議：
  - Multi-stage build
  - 模型分離存儲
  - 使用 Volume Mount

📚 Source:
- Docker Official Docs
- Kubernetes Docs

---

# Architecture Summary

## Why This Split Architecture?

NestJS:
- 適合做 API Gateway
- 負責流量控制與安全
- Type-safe + 模組化

FastAPI:
- 適合 AI / Data-heavy 任務
- Python 生態完整
- 快速實驗與推論

---

# High-Level Risk

- Cross-language token validation
- GIL 限制
- 異質資料庫管理
- Monitoring 標準不一致
- 資源競爭 (CPU/Memory)

---

# Recommended Pattern

User → NestJS Gateway → FastAPI Inference Service → Vector DB / Graph DB
