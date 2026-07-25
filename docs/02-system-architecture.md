# 02 — 系統架構（System Architecture）

> **文件狀態：** Phase 0 初始化（v0.1）  
> **最後更新：** 2026-06-13  
> **相關文件：** [01-business-goals.md](./01-business-goals.md) · [03-roadmap.md](./03-roadmap.md)

---

## 架構概覽

```
┌─────────────────────────────────────────────────────────────┐
│                     User / Cron / CLI                        │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                   OpenClaw (Orchestrator)                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Research   │  │   Analyst   │  │      Memory         │  │
│  │   Agent     │  │   Agent     │  │      Agent          │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└──────────────────────────┬──────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼───────┐  ┌───────▼───────┐  ┌───────▼───────┐
│  PostgreSQL   │  │     Redis     │  │ Vector DB     │
│  (Metadata)   │  │   (Cache)     │  │ (Qdrant/TBD)  │
└───────────────┘  └───────────────┘  └───────────────┘
        │
┌───────▼───────┐  ┌───────────────┐
│  MinIO / S3   │  │  OpenAI API   │
│  (Documents)  │  │  (LLM)        │
└───────────────┘  └───────────────┘
```

---

## 技術堆疊

| 層級 | 技術 | 用途 |
|------|------|------|
| Orchestrator | OpenClaw | Agent 編排、工作流 |
| Workflow | Python | 業務邏輯、RAG Pipeline |
| AI | OpenAI API | LLM 推論 |
| 關聯式 DB | PostgreSQL | Metadata、結構化資料 |
| 快取 | Redis | Session、快取 |
| 向量庫 | Qdrant（待確認） | Embedding 搜尋 |
| 物件儲存 | MinIO / S3（待確認） | PDF、原始文件 |
| 容器化 | Docker Compose → Kubernetes | 部署 |
| 監控 | Prometheus + Grafana + Loki | Observability |
| CI/CD | GitHub Actions + ArgoCD | GitOps |

---

## 核心元件

### OpenClaw

- **角色：** 平台 Orchestrator，負責 Agent 調度與工作流執行
- **部署：** `docker-compose.yml`（profile: cli / gateway）
- **Workspace：** `.openclaw/workspace/` — 任務專案與知識來源

### 資料層

| 元件 | 職責 | Phase |
|------|------|-------|
| PostgreSQL | Agent 狀態、知識 Metadata、結構化紀錄 | Phase 1 |
| Redis | 快取、Session、Rate Limit | Phase 1 |
| Vector DB | RAG Embedding 儲存與相似度搜尋 | Phase 2 |
| MinIO / S3 | PDF、圖片等原始素材 | Phase 2 |

### 外部整合

| 整合 | 用途 |
|------|------|
| OpenAI API | LLM 推論 |
| FinMind / Yahoo | 台股資料 |
| Telegram Bot | 通知與摘要推送 |

---

## 資料流（Phase 2 目標）

```
原始素材 (PDF / 新聞 / CSV)
        │
        ▼
   Parser / OCR
        │
        ▼
  Chunk + Embedding
        │
        ├──────────────┐
        ▼              ▼
   Vector DB      PostgreSQL
   (語意搜尋)      (Metadata)
        │
        ▼
   OpenClaw Agent
        │
        ▼
   分析結果 / 摘要 / 通知
```

---

## 目錄結構（目標）

```
openclaw-platform/
├── README.md
├── docs/
│   └── OpenClaw_AI_Platform_Roadmap/
│       ├── 01-business-goals.md
│       ├── 02-system-architecture.md
│       ├── 03-roadmap.md
│       ├── 04-observability.md      # Phase 3
│       ├── 05-kubernetes.md         # Phase 4
│       └── adr/                     # 技術決策紀錄
├── docker/
│   └── compose.yml
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── secret.yaml
├── scripts/
└── monitoring/
```

---

## 已知架構難點

| 問題 | 真正難點 | 緩解方向 |
|------|----------|----------|
| 成本暴增 | Token 用量 | 模型分級、快取、Prompt 優化 |
| 資料重複 | Embedding 去重 | Dedup + Metadata 設計 |
| Agent 幻覺 | Prompt 品質 | RAG Grounding、驗證步驟 |
| Workflow 混亂 | Orchestration | OpenClaw 明確分工 |
| 長期記憶失控 | Metadata 管理 | PostgreSQL 結構化 |
| PDF 品質差 | OCR | 預處理 + 人工抽查 |
| 新聞洗版 | Dedup | 時間窗口 + 相似度過濾 |

---

## Observability 架構（Phase 3 預覽）

```
OpenClaw / API Service
        │
        ├── Metrics → Prometheus → Grafana
        ├── Logs    → Loki       → Grafana
        └── Traces  → OTel        → Grafana
```

**Tracing 必須回答：**

- Request 經過哪些服務？
- 卡在哪一段（Bottleneck）？
- Redis / MQ 是否造成延遲？
- Log 能否透過 TraceID 關聯？

---

## 架構版本紀錄

| 版本 | 日期 | 變更 |
|------|------|------|
| v0.1 | 2026-06-13 | 初始架構文件，Phase 0 建立 |

---

## 待補項目

- [ ] 架構圖 v1（Mermaid / PNG）
- [ ] ADR-001：Orchestrator 選型（OpenClaw）
- [ ] ADR-002：PostgreSQL vs 其他 DB
- [ ] ADR-003：向量庫選型
- [ ] 資料流圖 v1（RAG Pipeline）
