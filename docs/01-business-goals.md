# 01 — 業務目標（Business Goals）

> **文件狀態：** Phase 2a（知識作業系統）中後期  
> **最後更新：** 2026-08-14  
> **相關文件：** [02-system-architecture.md](./02-system-architecture.md) · [03-roadmap.md](./03-roadmap.md) · [04-knowledge-operating-system.md](./04-knowledge-operating-system.md)

---

## 專案定位

**Personal AI Operations Platform** — 以 OpenClaw 為核心的個人 AI 營運平台，整合 Agent 工作流、知識庫搜尋、自動化排程與 Production 級基礎設施，作為履歷與面試可展示的成果。

**能力分水嶺：** 從「會用 Prompt」演進到 **累積 AI 可重用資產（Knowledge OS）**，再接 RAG／Observability／Kubernetes。

---

## 最終目標

建立可展示於履歷與面試的：

| 成果 | 說明 | 現況 |
|------|------|------|
| AI Agent Platform | OpenClaw 驅動的多 Agent 工作流 | ✅ |
| Docker Compose + Infra 拆分 | 共用 DB／Redis／HTTPS 邊緣；應用獨立 repo | ✅ |
| Personal Knowledge OS | TazKnowledges Vault＋kb-ID 治理＋keyword／chunk ingest | 🔄 |
| RAG 知識搜尋 | 向量庫 + Agent 檢索 | ⏳ |
| Observability Platform | Prometheus／Grafana／Loki | ⏳ |
| Kubernetes Migration | Compose → K8s 與 CKAD 實戰 | ⏳ |
| GitOps／CI-CD | GitLab CI + Runner | ⏳ |

---

## 同步證照目標

| 證照 | 角色 |
|------|------|
| AWS DVA-C02 | 開發者 Associate，與平台 AWS 整合對齊 |
| Terraform Associate | IaC 與基礎設施自動化，銜接 Compose → K8s |
| CKAD | Kubernetes 部署與維運實戰 |

---

## 核心使用場景

| 場景 | 邏輯域／專案 | 優先級 | 說明 |
|------|--------------|--------|------|
| 財報整理／台股追蹤 | `stocks`／workspace `stock-infos` | 高 | 新聞、法說、財報；實體知識在 TazKnowledges Finance |
| 技術文件搜尋 | `aws`／`k8s`／`sre` 等邏輯域 | 高 | Vault `01-Tech`＋RAG 準備中（keyword 已有） |
| 新聞摘要與分類 | `stock-infos` | 高 | 利多／利空／資金輪動／風險 |
| 求職管線 | `career` · workspace `job-search`／`interview` | 高 | STAR、廣搜、面試 brief；Career Vault 最厚 |
| PDF 歸檔 | `pdf-converter` → rawdata／aigen | 中 | OCR → MD → 策展進 Vault |
| 基礎設施監控與通知 | `line-notify` 等 | 中 | Gateway／Docker health |
| AWS 成本分析 | — | 中 | 待建 |
| Grafana Alert 根因分析 | — | 中 | Observability 整合後 |

邏輯域與實體路徑對照見 [04-knowledge-operating-system.md](./04-knowledge-operating-system.md)。

---

## Agent 分工

> 作品集敘事以目前驗證過的分工為準；設定細節在應用 repo，不進公開密鑰。

| Agent id | 職責 |
|----------|------|
| `main` | 日常對話（預設） |
| `deep` | 研究／較深任務 |
| `graph` | 視覺／圖表類 |
| `task` | 背景任務 |
| `m1pro` | Companion Node（本機節點執行） |
| `m2max` | Companion Node（本機節點執行；部分 daily／長任務） |

---

## 成功標準（Definition of Done）

### Phase 0～1（已完成）

- [x] OpenClaw Docker Compose 可正常運作
- [x] 專案文件架構（含作品集 v1）
- [x] AI Agent 基本工作流驗證
- [x] Infra 拆分：Postgres／Redis／Caddy／VPN 遠端 UI
- [x] n8n Queue Mode 與共用網路

### Phase 2a（中後期）

- [x] TazKnowledges 生命週期（`rawdata → aigen → obsidian → rag`）與 OpenClaw bind
- [x] kb-ID／`status/verified`／`rag-include` 治理
- [x] Keyword／chunk ingest（高信任文件進索引；**非**向量 RAG）
- [ ] knowledge-builder：真實 inbox → staging draft → 策展進 Vault 習慣化
- [ ] 重要對話習慣改為提煉 Markdown（見 [04](./04-knowledge-operating-system.md)）
- [ ] 技術／stocks 精煉資產加厚（Career 已相對成熟）

### 最終交付（Phase 5）

- 可 Demo 的 AI Platform（文件查詢、Agent Workflow、知識庫）
- 完整架構圖與 ADR
- Kubernetes 部署與 Observability Dashboard
- 履歷可引用的英文／中文成果描述

---

## 履歷成果描述

**面試請先用 Done 版（與 README 對齊）：**

**English**

> Built a personal AI operations platform on OpenClaw: multi-agent workflows, Docker Compose deployment, shared infrastructure (Postgres/Redis/HTTPS edge via a dedicated infra repo), and remote Control UI over VPN—documented as an evolving portfolio with clear Done vs Planned scope.

**繁中**

> 以 OpenClaw 建構個人 AI 營運平台：多 Agent 工作流、Docker Compose 部署、獨立基礎設施 repo（Postgres／Redis／HTTPS 邊緣）與 VPN 遠端 Control UI；並以文件區分已完成與規劃中範圍。

**目標敘述（含 RAG／K8s／GitOps，面試勿與 Done 混淆）：**

> Designed and built a Personal AI Operations Platform using OpenClaw, integrating RAG search, workflow automation, Docker Compose, Kubernetes deployment, and Grafana-based observability with GitOps CI/CD.

---

## 待決策事項

| 項目 | 選項 | 狀態 |
|------|------|------|
| 向量資料庫 | Qdrant／pgvector | 待 ADR |
| 物件儲存 | MinIO／S3 | 待 ADR |
| 排程引擎 | Cron／n8n／Temporal | 部分已用 OpenClaw cron + n8n；Temporal 待 ADR |

> 決策紀錄：`docs/adr/`（已有 [0001](./adr/0001-orchestrator-openclaw.md)）。
