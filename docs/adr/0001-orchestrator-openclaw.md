# ADR-0001：選用 OpenClaw 作為 Orchestrator

- **狀態：** Accepted
- **日期：** 2026-07-25
- **決策者：** 專案維護者

## 背景

需要一個可長期運轉的個人 AI 營運層：多 Agent、排程、workspace 檔案管線、可接外部工具，且能以 Docker 部署並逐步補齊資料層與可觀測性。

## 選項

| 選項 | 優點 | 缺點 |
|------|------|------|
| **A. OpenClaw** | 內建 Gateway／Agent／cron／skills；適合「營運平台」而非單次 notebook | 上游演進快，需跟版與設定治理 |
| B. 自研 LangGraph／自寫 orchestrator | 完全可控 | 維運成本高，分心做框架 |
| C. 僅 Chat UI + 手動腳本 | 上手快 | 難形成可重複管線與作品集敘事 |

## 決策

採 **選項 A：OpenClaw** 作為 Orchestrator；業務邏輯與知識以 **workspace 專案 + 腳本** 落地；基礎設施（DB／Redis／HTTPS）拆到獨立 Infra，避免單 repo 膨脹。

## 後果

- **正向：** 快速驗證多 Agent 與排程；面試可講「平台 + 管線」而非單次 demo。
- **代價：** 需追上游版本；設定與 secret 必須與公開作品集分離。
- **待決（不在本 ADR）：** 向量庫（Qdrant vs pgvector）、物件儲存（MinIO vs S3）、排程是否升級 Temporal。

## 參考

- [01-business-goals.md](../01-business-goals.md)
- [02-system-architecture.md](../02-system-architecture.md)
- [03-roadmap.md](../03-roadmap.md)
- [04-knowledge-operating-system.md](../04-knowledge-operating-system.md)

