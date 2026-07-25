# 01 — 業務目標（Business Goals）

> **文件狀態：** Phase 0 初始化
> **最後更新：** 2026-07-05
> **相關文件：** [02-system-architecture.md](./02-system-architecture.md) · [03-roadmap.md](./03-roadmap.md)

---

## 專案定位

**Personal AI Operations Platform** — 以 OpenClaw 為核心的個人 AI 營運平台，整合 Agent 工作流、知識庫搜尋、自動化排程與 Production 級基礎設施，作為履歷與面試可展示的成果。

---

## 最終目標

建立可展示於履歷與面試的：


| 成果                                    | 說明                             |
| ------------------------------------- | ------------------------------ |
| AI Agent Platform                     | OpenClaw 驅動的多 Agent 工作流        |
| Docker Compose Production Environment | 本地可重現的完整服務堆疊                   |
| Kubernetes Migration Project          | Compose → K8s 遷移與 CKAD 實戰      |
| Observability Platform                | Prometheus / Grafana / Loki 監控 |
| GitOps / CI-CD Workflow               | GitLab CI + Runner             |


---

## 同步證照目標


| 證照                  | 角色                                      |
| ------------------- | --------------------------------------- |
| AWS DVA-C02         | 開發者 Associate，與平台 AWS 整合對齊                |
| Terraform Associate | IaC 與基礎設施自動化，銜接 Compose → K8s 遷移          |
| CKAD                | Kubernetes 部署與維運實戰                        |


---

## 核心使用場景

OpenClaw 平台需支援以下高價值工作流：


| 場景                 | 專案名稱                                    | 優先級 | 說明                                      |
| ------------------ | --------------------------------------- | --- | --------------------------------------- |
| 財報整理 / 台股追蹤        | `stock-infos`                           | 高   | 每日新聞、法說、財報分析                            |
| 技術文件搜尋             | —                                       | 高   | AWS / K8s / CKAD / DVA 知識庫（`tech-knowledge` 待建） |
| 新聞摘要與分類            | `stock-infos`                           | 高   | 利多 / 利空 / 資金輪動 / 風險                       |
| 求職管線（職涯／廣搜／面試）     | `career` · `job-search` · `interview`   | 高   | 經歷 STAR、104 廣搜、面試 brief                  |
| PDF 歸檔             | `pdf-converter`                         | 中   | OCR + 結構化入庫                             |
| 基礎設施監控與通知          | `line-notify`                           | 中   | Gateway / Docker health、incident 推送     |
| AWS 成本分析           | —                                       | 中   | 雲端資源與費用洞察（待建）                           |
| Grafana Alert 根因分析 | —                                       | 中   | Observability 整合後啟用                     |


---

## Agent 分工

> 設定範本：`.openclaw/openclaw.json.example`；實際狀態見 `docs/openclaw-summary.md`。

| Agent id          | 職責              | 主要模型                          |
| ----------------- | --------------- | ----------------------------- |
| `main`（預設）        | 唯一聊天對話          | `openai/gpt-5.5`              |
| `deep`            | 任務／對話混用（對話優先）   | `openai/gpt-5.5`              |
| `research`        | 任務搜集（cron、廣搜）  | `google/gemini-2.5-flash`     |
| `analyst`         | 任務分析／推論         | `google/gemini-2.5-pro`       |
| `memory`          | 整理歸檔            | `google/gemini-2.5-flash-lite` |


---

## 成功標準（Definition of Done）

### Phase 0（現在 ~ DVA）

- OpenClaw Docker Compose 可正常運作
- 本文件目錄架構建立完成
- AI Agent 基本工作流驗證通過
- 通過 DVA-C02
- 通過 Terraform Associate

### 最終交付（Phase 5）

- 可 Demo 的 AI Platform（文件查詢、Agent Workflow、知識庫）
- 完整架構圖與技術決策文件（ADR）
- Kubernetes 部署與 Observability Dashboard
- 履歷可引用的英文 / 中文成果描述

---

## 履歷成果描述（草稿）

**英文版：**

> Designed and built a Personal AI Operations Platform using OpenClaw, integrating RAG search, workflow automation, Docker Compose, Kubernetes deployment, and Grafana-based observability with GitOps CI/CD.

**繁體中文版：**

> 以 OpenClaw 建構個人 AI 營運平台，整合 RAG 知識搜尋、工作流自動化、Docker Compose 本地環境、Kubernetes 部署，以及 Grafana 監控與 GitOps CI/CD 流程。

---

## 待決策事項


| 項目    | 選項                | 狀態    |
| ----- | ----------------- | ----- |
| 向量資料庫 | Qdrant / pgvector | 待 ADR |
| 物件儲存  | MinIO / S3        | 待 ADR |
| 排程引擎  | Cron / Temporal   | 待 ADR |


> 決策紀錄請存放於 `adr/` 目錄（後續 Phase 建立）。

