# 建造時間線（HISTORY）

> 作品集用里程碑。證據類型標註「可展示什麼」，不含密鑰、私人網域或職缺原文。

| 日期 | 里程碑 | 說明 | 證據類型 |
|------|--------|------|----------|
| 2026-06 | 環境落地 | OpenClaw Gateway Compose 可運行；文件目錄與架構圖 v1 | Compose、roadmap 初稿 |
| 2026-06 | 多 Agent 與第一條排程 | Agent 分工與盤後情報類 cron 驗證 | Agent 設定範本、cron 列表截圖（去敏） |
| 2026-06～07 | 求職管線 | career／job-search／interview 三分流；手動觸發與產出格式 | 管線說明文件、腳本名稱（無 JD 內容） |
| 2026-07 | 雙平台職缺擷取 | 104（REST 繞過）+ Cake（頁面資料擷取） | FETCH 說明、manual-run 流程 |
| 2026-07 | Infra 拆分 | 共用 network／Postgres／Redis／Caddy 遷至獨立 Infra；應用以 external network 加入 | Infra README、network 成員示意 |
| 2026-07 | 遠端 Control UI | VPN + TLS 終止於邊緣，反代至 Gateway（僅本機埠） | 架構方塊圖（無真實 FQDN） |
| 2026-07 | n8n Queue Mode | 與 OpenClaw 並列掛入共用 Infra；main + workers | 架構文件三 repo 權責 |
| 2026-07 | 作品集 v1 | 本目錄重組：README／HISTORY／ADR／Demo；運維手冊移出公開範圍 | 本 repo 文件樹 |
| 2026-08 | 階段校準 + PKOS | 路線圖確認 Phase 0～1 完成、Phase 2a 進行中；howto 整併入 docs | [docs/03-roadmap.md](./docs/03-roadmap.md)、[docs/04-knowledge-operating-system.md](./docs/04-knowledge-operating-system.md) |

## 下一里程碑（預告）

| 目標 | 對應 Phase |
|------|------------|
| `knowledge/` + knowledge-builder 匯入跑通 | Phase 2a |
| RAG 匯入與向量搜尋驗證 | Phase 2b |
| Grafana／Loki 分層看板 | Phase 3 |
| Compose → Kubernetes | Phase 4 |

進度細節見 [docs/03-roadmap.md](./docs/03-roadmap.md)。
