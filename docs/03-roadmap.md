# 03 — 開發路線圖（Roadmap）

> **文件狀態：** Phase 2a 中後期（知識作業系統）；Phase 2b 準備中  
> **最後更新：** 2026-08-14  
> **節奏：** 2 週一個成果，每 3 天檢查一次  
> **相關文件：** [01-business-goals.md](./01-business-goals.md) · [02-system-architecture.md](./02-system-architecture.md) · [04-knowledge-operating-system.md](./04-knowledge-operating-system.md)

---

## 階段定位（對照現況）

| 來源 | 結論 |
|------|------|
| README Done | Compose、多 Agent、workspace 管線、Infra 拆分、VPN + TLS 遠端 UI |
| 三 repo 架構 | TazInfra（network／DB／Redis／Caddy／NetBird）→ TazClaw／TazN8n 已落地 |
| 知識本體 | **TazKnowledges**：`rawdata → aigen → obsidian → rag`；kb-ID 治理與 keyword／chunk ingest 已跑通 |
| 本路線圖 | **Phase 0～1 完成**；目前位於 **Phase 2a 中後期**（非入口）；向量 RAG 屬 Phase 2b |

```
Phase 0 文件+基線     ✅
Phase 1 MVP 資料層    ✅（經 TazInfra；n8n Queue Mode 一併就緒）
Phase 2a Knowledge OS 🔄 ← 你在這裡（中後期）
Phase 2b 向量 RAG     ⏳（chunks／keyword 有；embedding／vector-db 空）
Phase 3 Observability ⏳
Phase 4 Kubernetes    ⏳
Phase 5 Production    ⏳
```

對照「2 週一個成果」節奏圖：Day 1–9（環境／資料層／Agent 工作流）已完成；目前對應 **Day 10–12（RAG 與搜尋）前半**——有 ingest／chunks，尚無向量搜尋 Demo。

---

## 總覽

| Phase | 時間（規劃） | 主題 | 狀態 |
| ----- | ------------ | ---- | ---- |
| 0 | 2026-06 ~ 07 | 證照衝刺 + 文件架構 + Agent 基線 | ✅ 完成 |
| 1 | 2026-07 | AI Platform MVP（共用資料層 + 應用拆分） | ✅ 完成 |
| 2a | 2026-08 ~ | Personal Knowledge OS（Vault、治理、匯入） | 🔄 中後期 |
| 2b | 接續 | 正式 RAG／向量庫 | ⏳ 準備中 |
| 3 | 待排 | Observability | ⏳ 待開始 |
| 4 | 待排 | Kubernetes Migration | ⏳ 待開始 |
| 5 | 待排 | Production Ready | ⏳ 待開始 |

---

## Phase 0：證照衝刺期 — ✅ 完成

**目標：** 通過 DVA-C02／Terraform Associate（證照時程另追）；建立 OpenClaw 專案文件架構；驗證 Agent 基線。

**已完成：**

| 項目 | 說明 |
|------|------|
| OpenClaw 環境 | Gateway Compose 運行；workspace 任務專案就緒 |
| docs 目錄 | `01`／`02`／`03` 核心文件；後續作品集重組 |
| 多 Agent + cron | 分工與盤後情報類排程驗證 |
| 文件搜尋 | workspace `sources/`／`projects/` 讀取流程 |

**完成標準：** OpenClaw Docker Compose 運作；AI Agent 基本工作流驗證 — **已滿足**。

---

## Phase 1：AI Platform MVP — ✅ 完成

**目標：** 資料層從規劃落地為可運行服務；應用與基礎設施權責分離。

**實際落地（相對原規劃的調整）：**

| 原規劃（單 repo Compose） | 實際（三 repo） |
|---------------------------|-----------------|
| 在 OpenClaw compose 加 postgres／redis | **TazInfra** 擁有 Postgres／Redis／Caddy／`taz-shared`／NetBird |
| 單堆疊 | **TazClaw**（OpenClaw）+ **TazN8n**（Queue Mode）掛入共用網路 |
| 僅本機 | 遠端：VPN → TLS 邊緣 → 反代至 Gateway／n8n |

**已完成：**

| 項目 | 說明 |
|------|------|
| 共用資料層 | Postgres、Redis healthy；n8n 使用 DB + Redis queue |
| Infra 拆分 | 應用不擁有共用 DB／邊緣 TLS／VPN |
| 遠端 Control UI | VPN + Caddy HTTPS；應用埠僅本機 publish |
| 作品集 v1 | README Done／HISTORY／ADR／Demo；運維細節與私人網域不進公開敘事 |
| n8n Queue Mode | main + workers；與 OpenClaw 並列為應用層 |

**完成標準對照：**

| 標準 | 狀態 |
|------|------|
| `postgres`／`redis` healthy | ✅（TazInfra） |
| 文件查詢（workspace） | ✅ |
| Agent Workflow（cron／手動） | ✅ |
| 基本知識庫 | ✅ 過渡完成 → Phase 2a（Vault＋keyword index；向量庫屬 2b） |

詳見 [02-system-architecture.md](./02-system-architecture.md)「現況架構」。

---

## Phase 2：RAG 與自動化工作流 — 🔄 進行中

**時間：** 2026-08 起  

**目標：** 建立真正有價值的 AI 助理 — **先知識資產，再向量庫**。

### 為什麼現在不是 K8s？

分水嶺是 **累積 AI 可重用資產**，不是再學一個框架。短期最高報酬仍是 Personal Knowledge Operating System（PKOS），見 [04-knowledge-operating-system.md](./04-knowledge-operating-system.md)。

### Phase 2a — Knowledge OS（當前焦點：中後期）

作品集敘事保留邏輯域（`aws`／`kubernetes`／`sre`／`stocks`／`career`／`workflow`／`projects`／`adr`）；**實體儲存**在 TazKnowledges（Obsidian 主題樹 + 生命週期目錄），對照見 [04](./04-knowledge-operating-system.md)。

| 狀態 | 項目 | 說明 |
|------|------|------|
| ✅ 已完成 | TazKnowledges 生命週期 | `rawdata → aigen → obsidian → rag`；OpenClaw bind 契約 |
| ✅ 已完成 | kb-ID／標籤治理 | `kb-*`、`status/verified`、`rag-include`；ledger |
| ✅ 已完成 | Keyword／chunk ingest | 高信任文件進 `rag/chunks`＋keyword index（**非**向量 RAG） |
| ✅ 已完成 | knowledge-builder Skill | 權威在 TazInfra；作品集接線見 [SKILLS.md](../SKILLS.md) |
| 🔄 進行中 | knowledge-builder E2E | 真實 inbox → staging draft → 策展進 Vault 習慣化 |
| 🔄 進行中 | 對話提煉習慣 | 重要成果 → Markdown／Runbook，不存整包 Chat |
| 🔄 進行中 | 精煉技術／stocks 資產 | Career 厚、技術知識與個股研究仍偏薄 |
| ⏳ 未完成 | 扁平邏輯域完整覆蓋 | 作品集域對照齊；實體目錄不必強求扁平複製 |

### Phase 2b — RAG Pipeline（接續）

| 待辦 | 要做什麼 |
|------|----------|
| **向量庫選型 ADR** | Qdrant vs pgvector（仍待決策 → ADR-003） |
| **Embedding** | 在既有 chunks 上產生 embedding；填滿 `rag/embedding/` |
| **Vector DB** | 寫入可重建的向量索引；首批來源：AWS／CKAD／stocks／career |
| **向量搜尋驗證** | 3～5 個代表性查詢測召回；Agent 能依結果回答 |
| **場景強化** | 股票研究、技術文件搜尋、證照知識整理、新聞摘要 |

**產出：** Knowledge 生命週期與匯入工作流；RAG Pipeline；Vector DB；Data Flow Diagram  

**完成標準：** 股票研究／技術文件搜尋／證照知識可經 Agent + 知識庫完成，而非單次聊天。

---

## Phase 3：Observability — ⏳

**目標：** Production Monitoring（Prometheus／Grafana／Loki）  

**監控項目：** CPU／Memory、Request／Response Time、Agent Workflow、token 用量、cron 成功率  

**完成標準：** 能分析 Agent 效能、定位瓶頸；Trace 能串 log。

---

## Phase 4：Kubernetes Migration — ⏳

**目標：** Compose → Kubernetes（對齊 CKAD）  

**產出：** `k8s/` Deployment／Service／Ingress／ConfigMap／Secret／PVC  

---

## Phase 5：Production Ready — ⏳

**目標：** 履歷級 Side Project 收斂  

**項目：** Probe、Resource Limit、Job／CronJob；Observability → K8s → GitLab GitOps  

**完成標準：** 架構圖、完整文件、可 Demo、履歷敘述就緒。

---

## 最終履歷成果（目標敘述）

**Project:** Personal AI Operations Platform  

**Features:** OpenClaw Agent Platform · RAG Search · Workflow Automation · Docker Compose · Kubernetes · Grafana · GitOps CI/CD  

**Tech Stack:** Python · OpenAI／多模型 · OpenClaw · n8n · Redis · PostgreSQL · Docker · Kubernetes · Prometheus · Grafana · Loki · GitLab CI  

> 面試時請先引用 README 的 **Done** 版敘述，再視需要展開 Planned。

---

## 進度追蹤

| 日期 | Phase | 里程碑 | 備註 |
|------|-------|--------|------|
| 2026-06-13 | 0 | 文件架構初始化 | Roadmap 三份核心文件 |
| 2026-07-05 | 0 | 環境與 Agent 基線 | Compose、多 Agent、文件搜尋 |
| 2026-07 | 1 | Infra 拆分 + 遠端 UI | TazInfra／TazClaw／TazN8n；VPN + TLS |
| 2026-07 | 1 | 作品集 v1 | README／HISTORY／ADR／Demo |
| 2026-08-01 | 2a | 階段校準 + PKOS 整併 | 確認位於 Phase 2a；文件併入 docs |
| 2026-08 | 2a | TazKnowledges 策展與 ingest | kb-ID、keyword／chunk index；非向量 RAG |
| 2026-08 | 2a | Skills／文件治理 | knowledge-builder；Infra docs → Knowledges |
| 2026-08 | 2a | Companion Node／Obsidian | m1pro／m2max；daily note 管線穩定化 |
| 2026-08-14 | 2a | 文件與現況對齊 | 定位改為 2a 中後期；架構圖／PKOS 圖資入庫 |

> 每 3 天更新此表格，記錄實際進度與偏差。
