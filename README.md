# Personal AI Operations Platform

> 以 **OpenClaw** 為核心的個人 AI 營運平台：多 Agent 工作流、workspace 任務管線、共用基礎設施（Docker network），並持續演進至 RAG／可觀測性／Kubernetes。  
> **本目錄為公開作品集敘事（v1）**；本機運維細節不在此檔。

## 30 秒故事

| | |
|--|--|
| **問題** | 個人知識、求職、投資情報分散；需要可重複執行的 Agent 管線，而非一次性聊天 |
| **做法** | OpenClaw Gateway 編排 Agent；任務以 workspace 專案落地；基礎設施拆到獨立 Infra repo（共用 DB／Redis／HTTPS 邊緣） |
| **現況** | Compose 可運行；多 Agent；求職／台股等管線已驗證；遠端 HTTPS 經 VPN + 反向代理；**Phase 2：知識作業系統／RAG 準備** |
| **下一步** | `knowledge/` 匯入與向量搜尋、Observability、Kubernetes（見 [docs/03-roadmap.md](./docs/03-roadmap.md)） |

## Done / In progress / Planned

| 狀態 | 項目 |
|------|------|
| **Done** | OpenClaw Docker Compose Gateway；多 Agent（`main`／`deep`／`graph`／`task`）；workspace 任務（求職管線、台股情報等）；Infra 拆分（共用 `taz-shared`、Postgres、Redis、Caddy）；n8n Queue Mode；遠端 Control UI（VPN + TLS）；作品集文件與 ADR |
| **In progress** | Personal Knowledge OS（Markdown 知識資產、knowledge-builder 匯入）；RAG／向量庫準備；預算與模型分級 |
| **Planned** | 向量庫正式上線、Prometheus／Grafana／Loki、Compose → Kubernetes、GitOps CI/CD |

**階段一句話：** Phase 0～1 已完成；目前在 **Phase 2a（知識作業系統）**，尚未進入 K8s。

## 來源與轉型脈絡（AI 使用者 → Platform Builder）

原 `howto_AIuser_to_AIPlaform_builder/` 筆記已整併至本 repo 文件（2026-08-01）；**權威版本見下方文件地圖**，不再保留獨立 howto 目錄。

| Phase | 狀態 |
|-------|------|
| 0 文件 + Agent 基線 | ✅ |
| 1 MVP（Infra／DB／Redis／Caddy／n8n） | ✅ |
| **2a Knowledge OS** | **🔄 目前焦點** |
| 2b 正式 RAG／向量庫 | ⏳ |
| 3～5 Observability／K8s／GitOps | ⏳ |

下一步不是先上 Kubernetes，而是 **累積可重用知識資產**（PKOS 全文見 [docs/04-knowledge-operating-system.md](./docs/04-knowledge-operating-system.md)，資訊圖見 [docs/assets/pkos-overview.png](./docs/assets/pkos-overview.png)）。

## 我負責什麼（vs 上游）

| 範圍 | 說明 |
|------|------|
| **我建置／維護** | 部署與設定、Agent 分工、workspace 專案與腳本、Infra／應用拆分、遠端存取架構、知識匯入工作流、本作品集文件與 ADR |
| **上游 OpenClaw** | Gateway／CLI／Agent runtime（映像與核心能力） |
| **不宣稱已完成** | 完整 K8s 叢集、生產級 Observability 全套、公開可一鍵重現的私人資料、正式向量 RAG 上線 |

## 架構（現況簡圖）

```
User / Cron / CLI
        │
        ▼
OpenClaw Gateway（應用 repo）
  Agents: main · deep · graph · task
        │
        │  Docker network: taz-shared
        ├──────────────────┐
        ▼                  ▼
Infra（獨立 repo）      n8n（應用 repo）
  Postgres · Redis       Queue Mode
  Caddy（HTTPS 邊緣）
        │
        ▼
Remote access: VPN → Caddy → Gateway / n8n
```

詳見 [docs/02-system-architecture.md](./docs/02-system-architecture.md)（現況 + 目標 + 信任邊界）。

## 履歷可用描述（與 Done 對齊）

**English**

> Built a personal AI operations platform on OpenClaw: multi-agent workflows, Docker Compose deployment, shared infrastructure (Postgres/Redis/HTTPS edge via a dedicated infra repo), and remote Control UI over VPN—documented as an evolving portfolio with clear Done vs Planned scope.

**繁中**

> 以 OpenClaw 建構個人 AI 營運平台：多 Agent 工作流、Docker Compose 部署、獨立基礎設施 repo（Postgres／Redis／HTTPS 邊緣）與 VPN 遠端 Control UI；並以文件區分已完成與規劃中範圍，作為可驗證的作品集敘事。

> 目標敘述（含 RAG／K8s／GitOps）見 [docs/01-business-goals.md](./docs/01-business-goals.md)，**面試時請先引用上方 Done 版**。

## 文件地圖

| 文件 | 用途 |
|------|------|
| [HISTORY.md](./HISTORY.md) | 建造時間線（作品集核心） |
| [docs/01-business-goals.md](./docs/01-business-goals.md) | 定位、場景、GoD |
| [docs/02-system-architecture.md](./docs/02-system-architecture.md) | 現況（三 repo）+ 目標架構 |
| [docs/03-roadmap.md](./docs/03-roadmap.md) | Phase 路線圖與階段定位 |
| [docs/04-knowledge-operating-system.md](./docs/04-knowledge-operating-system.md) | 知識資產習慣與 RAG 準備 |
| [docs/demo-v1.md](./docs/demo-v1.md) | 5 分鐘 Demo 講稿 |
| [docs/adr/0001-orchestrator-openclaw.md](./docs/adr/0001-orchestrator-openclaw.md) | 為何選 OpenClaw |
| [SKILLS.md](./SKILLS.md) · [workflows/](./workflows/) | knowledge-builder 等 Skill 接線 |

### 持續更新架構（Living docs）

這組文件隨平台狀態一起修訂（目錄慣例、Agent／知識層、地端部署變更時同步），**非**一次性定稿：

| 文件 | 用途 | 與另一份的關係 |
|------|------|----------------|
| [personal_ai_platform_architecture.md](./personal_ai_platform_architecture.md) | 四層概念圖＋目錄互動矩陣（rawdata → 處理 → IDE／n8n → vault） | 概念／資料流視角 |
| [personal_ai_platform_1786241579432.jpg](./personal_ai_platform_1786241579432.jpg) | 上列概念圖資產 | 由概念文件嵌入 |
| [architecture-n8n-infra-openclaw.md](./architecture-n8n-infra-openclaw.md) | 地端運維級：三 repo、網路、權責與安全邊界（非公開敘事正文） | 部署／Infra 視角；去敏摘要見 docs/02 |

## Portfolio DoD（v1 合格線）

- [x] README 區分 Done / In progress / Planned
- [x] 架構反映 Infra + 應用拆分
- [x] HISTORY ≥ 4 個真實里程碑
- [x] 無私人網域／完整運維 runbook（公開文件去敏）
- [x] 履歷句與 Done 一致
- [x] 至少一則 ADR + Demo 講稿

## License

MIT License. See [LICENSE](./LICENSE).
