# 阿爾發投顧 AI Engineer 面試速讀

## 公司背景
### 公司定位
- FinTech 公司
- 核心產品：Robo Advisor 機器人理財
- 使用 AI + 金融演算法提供投資建議
- 提供自動化資產配置與投資組合管理

### 公司產品
- AI 智能理財顧問
- 投資組合推薦
- ETF 投資配置
- 自動再平衡
- 退休資產規劃

### 公司成立
- 成立時間：2017
- FinTech WealthTech
- 台灣 Robo Advisor 平台

### Robo Advisor 概念
- 根據使用者
  - 年齡
  - 風險承受度
  - 投資目標
- 自動建立 ETF 投資組合
- 持續監控並調整資產配置
- 降低投資決策的人性偏誤

---

# AI Model Integration

## AI 在 Robo Advisor 的角色
### Risk Profiling
- 分析使用者風險承受度
- 建立投資風險模型

### Portfolio Recommendation
- 建立投資組合
- ETF Allocation

### Market Analysis
- 市場資料分析
- 預測市場風險

### Rebalancing Decision
- 自動再平衡
- 維持資產配置比例

---

## AI Integration Architecture

User Request
↓
Backend API
↓
AI Model Service
↓
Prediction Result
↓
Portfolio Recommendation

---

## AI Integration 實務架構

Frontend
- Next.js
- Web App

Backend
- NestJS / FastAPI

AI Service
- Python Model Server
- REST API

Model
- Machine Learning
- Portfolio Optimization

---

# Data Pipeline

## Financial Data Sources
- Stock Market Data
- ETF Data
- Economic Indicators
- User Portfolio Data

---

## Data Pipeline Architecture

Market Data
↓
Data Ingestion
↓
Data Cleaning
↓
Feature Engineering
↓
Model Training
↓
Model Inference
↓
Investment Recommendation

---

## Feature Engineering (金融特徵)

- Returns(收益維度)
- Volatility(風險維度)
- Risk Score(綜合維度)
- Asset Correlation(分散維度)
- Market Trend(環境維度)

---

# AI Service Deployment

## AI Deployment Architecture

Frontend
↓
API Gateway
↓
Backend Service
↓
AI Service
↓
Model Server
↓
Database

---

## 技術 Stack

Frontend
- Next.js

Backend
- NestJS
- FastAPI

Database
- PostgreSQL
- MySQL

AI Model
- Python
- Scikit-learn
- PyTorch

---

## Deployment

Containerization
- Docker

CI/CD
- GitHub Actions

Infrastructure
- Cloud Server
- Kubernetes (optional)

---

# Robo Advisor System Design

## System Architecture

User App
↓
API Gateway
↓
Backend Services
↓
Risk Assessment Service
↓
Portfolio Optimizer
↓
Market Data Pipeline
↓
Database

---

## 核心金融演算法

### Efficient Frontier
- 最佳風險報酬配置

### Black-Litterman
- 投資觀點結合市場數據

### Monte Carlo Simulation
- 模擬未來投資結果

---

# 常見面試問題

## What is Robo Advisor
- 自動化投資顧問
- 使用 AI 與演算法
- 根據風險承受度建立投資組合

---

## How does AI help investment
- 分析市場資料
- 建立投資組合
- 自動再平衡

---

## How would you deploy an AI model
- 將模型部署成 API Service
- Backend 呼叫 AI service
- 回傳 prediction result

---

## How to build AI data pipeline
1. 收集市場資料
2. 清洗資料
3. 特徵工程
4. 模型預測
5. 投資建議