# 03 — 開發路線圖（Roadmap）

> **文件狀態：** Phase 0 進行中  
> **最後更新：** 2026-07-05  
> **節奏：** 2 週一個成果，每 3 天檢查一次  
> **相關文件：** [01-business-goals.md](./01-business-goals.md) · [02-system-architecture.md](./02-system-architecture.md)

---

## 總覽


| Phase | 時間        | 主題                   | 狀態     |
| ----- | --------- | -------------------- | ------ |
| 0     | 現在 ~ DVA  | 證照衝刺 + 文件架構          | 🔄 進行中 |
| 1     | 第 1~2 週   | AI Platform MVP      | ⏳ 待開始  |
| 2     | 第 3~4 週   | RAG 與自動化工作流          | ⏳ 待開始  |
| 3     | 第 5~6 週   | Observability        | ⏳ 待開始  |
| 4     | 第 7~10 週  | Kubernetes Migration | ⏳ 待開始  |
| 5     | 第 11~12 週 | Production Ready     | ⏳ 待開始  |


---

## Phase 0：證照衝刺期（現在 ~ DVA）

**目標：**

- 通過 DVA-C02
- 通過 Terraform Associate（DVA 之後）
- 建立 OpenClaw 專案文件架構

**Deliverables：**

```
docs/OpenClaw_AI_Platform_Roadmap/
├── 01-business-goals.md      ✅
├── 02-system-architecture.md ✅
└── 03-roadmap.md             ✅
```

**已完成：**


| 項目                 | 說明                                                              |
| ------------------ | --------------------------------------------------------------- |
| OpenClaw 環境整理      | Gateway + Caddy Compose 運行中；workspace 任務專案就緒                    |
| 建立 docs 目錄         | `docs/OpenClaw_AI_Platform_Roadmap/` 三份核心文件                     |
| 系統架構圖 v1 / v2      | `02-system-architecture.md` 初版與技術堆疊對照                           |
| README 完成          | repo `README.md`、`docs/openclaw-summary.md` 操作指南                |
| 第一個 Agent Workflow | 多 Agent 分工（`main` / `research` / `analyst` / `memory`）與 cron 驗證 |
| 文件搜尋               | workspace `sources/` 與 `projects/` 讀取流程驗證                       |


**完成標準：**

- OpenClaw Docker Compose 運作
- AI Agent 基本工作流驗證

**前 14 天節奏：**


| 區間        | 任務                                | 狀態      |
| --------- | --------------------------------- | ------- |
| Day 1–3   | OpenClaw 環境整理、建立 docs 目錄、系統架構圖 v1 | ✅       |
| Day 4–6   | README 完成、架構圖 v2                  | ✅       |
| Day 4–6   | Redis / Postgres 資料流整理            | ⏳ 見下方待辦 |
| Day 7–9   | 第一個 Agent Workflow、文件搜尋           | ✅       |
| Day 7–9   | ADR 技術決策紀錄                        | ⏳ 見下方待辦 |
| Day 10–12 | RAG 資料匯入、向量搜尋驗證、Dashboard 需求設計    | ⏳ 見下方待辦 |
| Day 13–14 | Demo v1 完成                        | ⏳ 見下方待辦 |


**待完成（具體工作）：**


| 待辦                         | 要做什麼                                                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Redis / Postgres 資料流整理** | 在 `02-system-architecture.md` 補上資料流圖：OpenClaw ↔ Redis（快取／Session）↔ PostgreSQL（Metadata）；定義各 workspace 專案哪些狀態寫 DB、哪些仍用檔案；作為 Phase 1 Compose 服務設計依據 |
| **ADR 技術決策紀錄**             | 建立 `adr/` 目錄；針對待決策項（向量庫 Qdrant vs pgvector、物件儲存 MinIO vs S3、排程 Cron vs Temporal）各寫一份 ADR，記錄選項、決策與理由                                               |
| **RAG 資料匯入**               | 選定首批知識來源（如 AWS / CKAD 筆記、`tech-knowledge` 來源）；建立匯入腳本或流程；將文件 chunk 化並寫入向量庫（或過渡方案：檔案索引）                                                             |
| **向量搜尋驗證**                 | 以 3~5 個代表性查詢測試召回率；確認 Agent 能透過搜尋結果回答技術問題；記錄失敗案例供 Phase 2 調整                                                                                       |
| **Dashboard 需求設計**         | 列出 Phase 3 Observability 需監控的指標（Agent 延遲、cron 成功率、token 用量、服務 health）；草擬 Grafana panel 清單與 alert 條件                                               |
| **Demo v1 完成**             | 錄製或撰寫 5 分鐘內可重現的 Demo 腳本：啟動 Compose → 觸發 Agent Workflow → 展示文件搜尋結果；產出 `docs/demo-v1.md`                                                            |


---

## Phase 1：AI Platform MVP

**時間：** 第 1~2 週（Phase 0 待辦完成後啟動）

**目標：** 建立第一個可展示成果，將資料層從「架構規劃」落地為可運行的 Compose 服務

**功能：**

- OpenClaw（既有 Gateway + Workspace）
- PostgreSQL — Metadata、結構化紀錄
- Redis — 快取、Session、Rate Limit
- OpenAI API（既有）

**實際操作（PostgreSQL + Redis 部署）：**


| 步驟            | 內容                                                                                   |
| ------------- | ------------------------------------------------------------------------------------ |
| 1. Compose 服務 | 在 `docker-compose.yml` 新增 `postgres`、`redis` 服務；設定 volume 持久化、`healthcheck`、資源 limit |
| 2. 環境變數       | `.env.example` 補上 `POSTGRES_`*、`REDIS_URL`；Gateway 容器透過 service name 連線              |
| 3. Schema 初版  | 建立 `scripts/db/` 或 migration 目錄；定義首批 table（如 agent_run_log、project_metadata）         |
| 4. 連線驗證       | `docker compose up` 後以 `psql`、`redis-cli` 確認連通；OpenClaw 或測試腳本寫入／讀取一筆資料               |
| 5. 資料流落地      | 依 Phase 0 整理的資料流圖，將至少一個 workspace 專案狀態改寫或雙寫至 PostgreSQL（其餘仍可用檔案過渡）                   |
| 6. 文件更新       | 更新 `README.md`、`02-system-architecture.md` 部署章節；`docker compose ps` 截圖或指令納入 Demo 素材  |


**產出：**

- Docker Compose（含 OpenClaw + PostgreSQL + Redis）
- Architecture Diagram（含資料層連線）
- README（含啟動與驗證步驟）

**完成標準：**

- `docker compose ps` 顯示 postgres、redis 皆 healthy
- 文件查詢（workspace 檔案搜尋，既有能力維持）
- Agent Workflow（cron 或手動觸發可寫入 DB 紀錄）
- 基本知識庫（檔案索引或向量庫過渡方案可查詢）

---

## Phase 2：RAG 與自動化工作流

**時間：** 第 3~4 週

**目標：** 建立真正有價值的 AI 助理

**功能：**

- 新聞收集
- 財報分析
- AWS 學習助理
- 知識庫搜尋

**產出：**

- RAG Pipeline
- Vector Database
- Data Flow Diagram

**完成標準：**

- 股票研究
- 技術文件搜尋
- AWS 證照知識整理

---

## Phase 3：Observability

**時間：** 第 5~6 週

**目標：** 建立 Production Monitoring

**功能：**

- Prometheus
- Grafana
- Loki

**監控項目：**

- CPU / Memory
- Request / Response Time
- Agent Workflow

**產出：**

- Dashboard
- Alert Design

**完成標準：**

- 能分析 Agent 效能
- 能定位系統瓶頸

---

## Phase 4：Kubernetes Migration

**時間：** 第 7~10 週

**目標：** 開始 CKAD 路線

```
Docker Compose  →  Kubernetes
```

**完成項目：**

- Deployment
- Service
- Ingress
- ConfigMap
- Secret
- PVC

**產出：**

```
k8s/
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── configmap.yaml
└── secret.yaml
```

---

## Phase 5：Production Ready

**時間：** 第 11~12 週

**目標：** 完成履歷級 Side Project

**完成項目：**

- Health Check / Probe
- Resource Limit
- Job / CronJob

**演進路徑：**

```
Observability  →  Kubernetes  →  GitLab (GitOps)
```

**完成標準：**

- 架構圖
- 完整文件
- 可 Demo
- 履歷內容就緒

---

## 最終履歷成果

**Project:** Personal AI Operations Platform

**Features:**

- OpenClaw Agent Platform
- RAG Search
- Workflow Automation
- Docker Compose
- Kubernetes Deployment
- Grafana Monitoring
- GitOps CI/CD

**Tech Stack:**

Python · OpenAI · OpenClaw · Redis · PostgreSQL · Docker · Kubernetes · Prometheus · Grafana · Loki · GitLab CI · Runner

---

## 進度追蹤


| 日期         | Phase | 里程碑            | 備註                                              |
| ---------- | ----- | -------------- | ----------------------------------------------- |
| 2026-06-13 | 0     | 文件架構初始化        | 建立 Roadmap 目錄與三份核心文件                            |
| 2026-07-05 | 0     | 環境與 Agent 基線完成 | Compose 運行、多 Agent 分工、文件搜尋驗證；待辦：ADR、RAG、Demo v1 |


> 每 3 天更新此表格，記錄實際進度與偏差。

