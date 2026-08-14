# ADR-0003：Phase 2b 向量庫選型（pgvector vs Qdrant）

- **狀態：** Proposed（建議採納）
- **日期：** 2026-08-14
- **決策者：** 專案維護者
- **相關 Phase：** Phase 2b（正式 RAG／向量搜尋）

---

## 背景

Phase 2a 已完成 TazKnowledges 生命週期與 **keyword／chunk ingest**（`rag/chunks`＋keyword index）。Phase 2b 要在既有 chunks 上產生 embedding，並讓 Agent 能做語意檢索。

約束與現況：

| 條件 | 說明 |
|------|------|
| 已有資料層 | TazInfra 已運行 **PostgreSQL**（metadata／n8n）與 Redis |
| 資料規模 | 個人知識庫；首批來源 AWS／CKAD／stocks／career |
| 權威來源 | 策展後 Vault（`kb-*`／`rag-include`）；向量索引須可重建 |
| 部署形態 | 目前 Compose＋三 repo；K8s 屬 Phase 4 |
| 作品集敘事 | 需能清楚講「為何選這個、何時該換」 |

待決問題：向量索引放在 **既有 Postgres（pgvector）**，還是另開 **專用向量服務（Qdrant）**。

---

## 選項

| 選項 | 優點 | 缺點 |
|------|------|------|
| **A. pgvector（PostgreSQL 擴充）** | 不新增長駐服務；metadata／filter／chunk 可同庫 JOIN；備份與權限沿用 TazInfra Postgres；個人規模足夠；運維面最小 | 超大規模 ANN、進階 payload／複雜過濾、獨立水平擴展不如專用庫；需確認 Postgres image／extension 啟用 |
| **B. Qdrant** | 向量檢索專用；過濾／payload／集合管理成熟；與 DB 負載隔離；作品集可講「專用向量層」 | 多一個 Compose 服務與磁碟／升級面；chunk metadata 需與 Postgres／檔案雙邊對齊；短期報酬低於「先出可 Demo 召回」 |
| C. 僅檔案／Chroma 等嵌入式 | 上手極快、少設定 | 與現有 TazInfra 權責不一致；難講清 production 路徑；易變成拋棄式實驗 |

不納入本期比較：Weaviate／Milvus（運維過重）、純雲端托管向量庫（綁定與成本敘事不符合本機＋VPN 架構）。

---

## 建議決策

採 **選項 A：pgvector（掛在 TazInfra PostgreSQL）** 作為 Phase 2b **預設向量庫**。

理由（對齊「先累積可重用資產、最短路徑驗證召回」）：

1. **摩擦最低**：共用 `taz-shared`、既有備份與網路邊界，不必為 Demo 再開服務拓撲。
2. **資料模型一致**：`kb-id`、domain、`rag-include`、chunk 文本與向量可同庫關聯，重建索引腳本單純。
3. **規模匹配**：個人 Vault 量級下，pgvector 足夠支撐 3～5 個代表性查詢驗證與 Agent 檢索。
4. **可逆**：chunk 檔與 embedding 產物以 TazKnowledges `rag/` 為可重建來源；若日後 Qdrant 更合適，可換寫入目標而不改策展／ingest 上游。

**Qdrant 保留為升級選項**，不在本期預設部署。觸發改選見下方「退出條件」。

---

## 後果

### 正向

- Phase 2b 可直接：embedding → 寫入 pgvector → 測召回 → Agent 回答。
- Infra 權責清楚：向量仍屬 **TazInfra 資料層**；應用（TazClaw）只消費檢索 API／SQL／薄封裝。
- 面試敘事：先 keyword ingest，再同庫語意檢索；專用向量服務是有依據的演進，而非一開始過度設計。

### 代價

- 需在 TazInfra 啟用／驗證 `vector` extension，並約定 schema（例：collection／table、embedding 維度、model 名稱、chunk_id ↔ kb-id）。
- n8n 與向量表共用同一 Postgres 實例時，要注意連線與維護視窗；必要時可另開 database／role 隔離，仍同容器。
- 極大量文件或高 QPS 時可能需遷移（見退出條件）。

### 實作邊界（本 ADR 範圍內）

| 做 | 不做（本期） |
|----|--------------|
| 在既有 chunks 上 embedding，寫入可重建的向量表 | 宣稱「向量 RAG 已上線」於作品集 Done（完成驗證前） |
| 3～5 個代表性查詢測召回 | 一開始就上 Qdrant「以防萬一」 |
| Agent 能依檢索結果回答 | 物件儲存 ADR、Observability、K8s |

### 退出條件（改採 Qdrant 或雙寫）

出現任一時，開後續 ADR 或修訂本決策：

- Postgres 資源／鎖競爭明顯影響 n8n 或檢索延遲；
- 需要 Qdrant 級 payload／多租戶集合／進階 ANN 調參，且 pgvector 驗證成本更高；
- 知識量或併發成長到個人 Compose 單庫難以維運。

遷移原則：**上游 Vault＋chunks 不變**；重跑 embedding／ingest 指向新後端。

---

## 待決（不在本 ADR）

- Embedding 模型與維度（本地／API）
- 物件儲存（MinIO vs S3）
- 檢索 API 形狀（直連 SQL vs 薄服務／Skill）
- Hybrid：keyword + vector 融合策略

---

## 參考

- [0001-orchestrator-openclaw.md](./0001-orchestrator-openclaw.md)（待決項：向量庫）
- [01-business-goals.md](../01-business-goals.md)
- [02-system-architecture.md](../02-system-architecture.md)
- [03-roadmap.md](../03-roadmap.md)
- [04-knowledge-operating-system.md](../04-knowledge-operating-system.md)
