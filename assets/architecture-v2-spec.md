# Architecture v2 — 圖面規格（圖像 AI 交付用）

> **用途：** 產出公開作品集圖 `assets/architecture-v2.png`  
> **對應階段：** Phase 2a 中後期（Compose + 三 repo + TazKnowledges + keyword ingest）  
> **權威敘事：** `docs/02-system-architecture.md`  
> **禁止出現：** 真實網域、FQDN、IP、埠號清單、密鑰、私人主機名（勿寫 M2 Max／個人 DNS）

---

## 1. 圖的一句話目標

畫一張**現況運行架構圖**（不是求職節奏圖、不是四層知識概念圖）：  
「遠端經 VPN → TLS 邊緣進入地端；TazInfra 提供共用資料層與網路；TazClaw／TazN8n 為應用；TazKnowledges 以 bind mount 掛入；向量庫／監控／K8s 僅標 Planned。」

---

## 2. 畫布與版式

| 項目 | 規格 |
|------|------|
| 比例 | 16:9 橫向（建議 1920×1080 或 2400×1350） |
| 背景 | 深色技術風（深藍黑／石墨），非純白簡報風 |
| 佈局 | **上：遠端存取與信任邊界** → **中：Host 內三 repo + 知識本體** → **右下或底：Planned 區（虛線）** |
| 語言 | 圖內標籤用 **英文為主、關鍵處可中英並列**（作品集面試友善） |
| 標題 | 左上：`Personal AI Operations Platform — Architecture v2` |
| 副標 | `Current: Phase 2a · Compose + Knowledge OS (keyword ingest)` |
| 圖例 | 右上固定圖例（見 §5） |

**禁止：** 卡片堆疊過多、紫粉霓虹、emoji、真實 logo 海報牆、把 K8s／Grafana 畫成已上線實線連線。

---

## 3. 分層（由外到內／由上到下）

### Layer A — Access & Trust Boundary（圖上方橫帶）

從左到右：

1. **Public Internet**  
   - 標籤：`Public Internet`  
   - 狀態：紅色叉或「No direct Control UI / n8n」  
2. **VPN Peer**  
   - 標籤：`VPN (WireGuard-class)`  
   - 狀態：Done（實線綠／青）  
   - 說明小字：`Peer identity + ACL · private DNS`  
3. **Edge TLS**  
   - 標籤：`Caddy (TLS termination)`  
   - 歸屬：TazInfra  
   - 狀態：Done  
   - 說明：`reverse proxy → apps`（**不要**寫真實 Host 名）  
4. 箭頭進入下方 Host 內的 `taz-shared`

### Layer B — Host Runtime（圖中央大框）

外框標題：`Host (on-prem)`

內部分三欄 + 底部一橫列：

| 區域 | 位置 | 內容 |
|------|------|------|
| **B1 TazInfra** | 上橫跨或左上寬框 | 共用基礎設施（最先啟動） |
| **B2 TazClaw** | 中左 | OpenClaw 應用 |
| **B3 TazN8n** | 中右 | n8n Queue Mode |
| **B4 TazKnowledges** | 底部橫帶 | 知識本體（**非容器**，檔案樹視覺） |

虛線環繞 B1～B3 內元件，標註：`Docker network: taz-shared`

### Layer C — Planned（圖右下或底右虛線框）

標題：`Planned (not in current runtime)`  
全部虛線邊框 + 灰色／琥珀色，**不得**用實線接到 Done 服務當成已上線。

---

## 4. 元件清單

### 4.1 Done（實線、實心邊框、主色）

| ID | 顯示名稱 | 所屬 | 圖上必寫的子標籤 | 備註 |
|----|----------|------|------------------|------|
| D1 | `TazInfra` | Infra repo | `shared infra · starts first` | 外框 |
| D2 | `taz-shared` | TazInfra | `Docker network` | 可用虛線橢圓包住應用+DB |
| D3 | `PostgreSQL` | TazInfra | `metadata · n8n state` | |
| D4 | `Redis` | TazInfra | `queue / cache` | |
| D5 | `Caddy` | TazInfra | `HTTPS edge` | 可與 Layer A 同一元件，勿重複兩顆 |
| D6 | `VPN agent` | Host-level | `not a container` | 小字即可；可用「主機」圖示 |
| D7 | `skills/` | TazInfra | `knowledge-builder (source of truth)` | 可畫成小資料夾徽章 |
| D8 | `TazClaw` | App repo | `OpenClaw application` | 外框 |
| D9 | `OpenClaw Gateway` | TazClaw | `orchestrator` | 核心節點，視覺權重最高 |
| D10 | `Agents` | TazClaw | `main · deep · graph · task · m1pro · m2max` | 一列 chip，勿再展開 |
| D11 | `workspace` | TazClaw | `projects · cron pipelines` | |
| D12 | `TazN8n` | App repo | `n8n Queue Mode` | 外框 |
| D13 | `n8n main` | TazN8n | `UI / API` | |
| D14 | `n8n workers × N` | TazN8n | `scaled workers` | |
| D15 | `TazKnowledges` | Knowledge repo | `not a container · bind mount` | 外框強調「檔案本體」 |
| D16 | Lifecycle path | TazKnowledges | `rawdata → aigen → obsidian → rag` | 水平流程 |
| D17 | `rag/` | TazKnowledges | `chunks + keyword index` | 明確寫 keyword，**不要**寫 vector |
| D18 | Entry actors | 外部 | `User / Cron / CLI` | 指向 Gateway |

### 4.2 In progress（半實線或雙色邊框）

| ID | 顯示名稱 | 標示 | 位置建議 |
|----|----------|------|----------|
| I1 | `knowledge-builder E2E` | `In progress` | skills → aigen／obsidian 旁小徽章 |
| I2 | `Vault curation habit` | `In progress` | obsidian 節點旁 |

### 4.3 Planned（虛線框，與現況隔離）

| ID | 顯示名稱 | 小字 |
|----|----------|------|
| P1 | `Vector DB` | `Qdrant / pgvector · ADR TBD` |
| P2 | `Embedding` | `on existing chunks` |
| P3 | `Observability` | `Prometheus · Grafana · Loki` |
| P4 | `Kubernetes` | `Compose → K8s` |
| P5 | `Object storage` | `MinIO / S3 · ADR TBD` |
| P6 | `GitOps CI/CD` | `GitLab CI` |

可選：從 `rag/` 畫一條**虛線箭頭**到 P1／P2，標籤 `Phase 2b`。

### 4.4 明確不要畫進圖的元件

- Hermes Agent、Antigravity、Immich、真實域名  
- 具體埠號（`:443` 可只寫在圖例「edge only」，不要列 DB 埠）  
- Prometheus／Grafana 實線連線到 Gateway  
- 單 repo 大一統 Compose（必須呈現三 repo 拆分）

---

## 5. 連線（箭頭語意）

| 從 → 到 | 線型 | 標籤 |
|---------|------|------|
| User/Cron/CLI → OpenClaw Gateway | 實線 | `control / tasks` |
| VPN → Caddy | 實線 | `remote HTTPS` |
| Public Internet → VPN | 實線但標 ❌ 旁路 | `blocked to apps`（Internet 不直連 Gateway） |
| Caddy → Gateway / n8n main | 實線 | `reverse proxy` |
| Gateway ↔ Postgres / Redis | 實線（細） | `via taz-shared`（若 OpenClaw 尚未重度用 DB，可畫較淡） |
| n8n main/workers ↔ Postgres / Redis | 實線 | `DB + queue` |
| Gateway → TazKnowledges | **粗實線** | `bind mount`；旁註 `ro: vault/raw · rw: aigen` |
| skills/ → Gateway | 虛線或細實線 | `mounted skills` |
| obsidian → rag/ | 實線（知識帶內） | `ingest` |
| rag/ → Vector DB (Planned) | **虛線** | `Phase 2b` |
| 任何 Done → Kubernetes / Observability | **禁止實線** | 僅放在 Planned 框內 |

---

## 6. Done / In progress / Planned 視覺圖例（必須印在圖上）

| 狀態 | 邊框 | 填色建議 | 角標文字 |
|------|------|----------|----------|
| **Done** | 實線 | 青藍／青綠半透明 | `DONE` |
| **In progress** | 半實線或點劃線 | 琥珀半透明 | `WIP` |
| **Planned** | 虛線 | 灰／紫灰，低對比 | `PLANNED` |

圖例三格橫排於標題右側。  
可另加一小標：`Portfolio-safe · no private FQDN`

---

## 7. 建議座標草圖（給生成模型的空間描述）

```
[Title ........................] [Legend: DONE | WIP | PLANNED]

[ Public Internet ✕ ]──►[ VPN ]──►[ Caddy / TLS ]──┐
                                                   │
┌────────────── Host (on-prem) ────────────────────▼──────────────┐
│  ┌─ TazInfra ─────────────────────────────────────────────────┐ │
│  │  Postgres │ Redis │ Caddy* │ skills/ (knowledge-builder)   │ │
│  │            [ taz-shared network ellipse ]                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌─ TazClaw ──────────────┐   ┌─ TazN8n ────────────────────┐  │
│  │ OpenClaw Gateway ★     │   │ n8n main                    │  │
│  │ Agents chips           │   │ workers × N                 │  │
│  │ workspace / cron       │   │ Queue Mode                  │  │
│  └───────────┬────────────┘   └──────────────┬──────────────┘  │
│              │ bind mount                    │ DB/queue        │
│              ▼                               ▼                  │
│  ┌─ TazKnowledges ──────────────────────────────────────────┐  │
│  │ rawdata → aigen → obsidian → rag (chunks + keyword)      │  │
│  │            [WIP: curation / knowledge-builder E2E]       │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ┊ Phase 2b
                              ▼
              ┌─ PLANNED ─────────────────────────────┐
              │ Vector DB · Embedding · Observability │
              │ Kubernetes · MinIO/S3 · GitOps        │
              └───────────────────────────────────────┘
```

\* Caddy 若已在上方 Access 層畫過，Host 內 TazInfra 用「同屬 Infra」引用即可，避免兩顆重複大框。

---

## 8. 圖像 AI 主提示詞（可直接貼上）

```text
Create a clean technical architecture diagram, 16:9 landscape, dark graphite-blue background, neon cyan accents (not purple). Title top-left: "Personal AI Operations Platform — Architecture v2". Subtitle: "Current: Phase 2a · Compose + Knowledge OS (keyword ingest)".

Layout top-to-bottom:
1) Access strip: Public Internet (blocked to apps) → VPN (WireGuard-class, DONE) → Caddy TLS edge (DONE) → into host.
2) Large "Host (on-prem)" frame containing:
   - TazInfra box: PostgreSQL, Redis, skills/knowledge-builder, shared Docker network "taz-shared".
   - TazClaw box: OpenClaw Gateway (visual center), agent chips (main, deep, graph, task, m1pro, m2max), workspace/cron.
   - TazN8n box: n8n main + workers × N, Queue Mode.
   - Bottom knowledge strip TazKnowledges (NOT a container): horizontal flow rawdata → aigen → obsidian → rag labeled "chunks + keyword index"; bind-mount arrow from Gateway with note "ro vault/raw · rw aigen".
3) Separate dashed "PLANNED" panel (lower-right): Vector DB, Embedding, Observability (Prometheus/Grafana/Loki), Kubernetes, MinIO/S3, GitOps — no solid lines treating them as live.

Legend top-right: DONE (solid cyan), WIP (amber dashed), PLANNED (gray dashed). Mark knowledge-builder E2E and vault curation as WIP badges.

Constraints: portfolio-safe; no real domains, FQDNs, IPs, ports, or hostnames; no Kubernetes/monitoring drawn as currently running; English primary labels; crisp boxes and arrows; high readability for interview screenshots; flat vector infographic style, not photorealistic.
```

### 負面提示（Negative）

```text
purple glow, emoji, real company logos collage, white PowerPoint template, newspaper layout, cream paper theme, realistic photos, cluttered stat cards, fake domain names, port numbers list, Hermes agent, Immich, claiming RAG vector search is live, Kubernetes cluster as current state
```

---

## 9. 產出驗收清單（生成後人工檢查）

- [ ] 一眼能看出 **三 repo + 知識本體**（不是單堆疊）  
- [ ] **VPN → Caddy → apps** 信任邊界清楚；Internet 不直連 Control UI  
- [ ] TazKnowledges 標成 **非容器 / bind mount**  
- [ ] `rag` 寫的是 **keyword／chunks**，不是 vector search 已上線  
- [ ] Planned 區塊與 Done **視覺分離**  
- [ ] 無私人 FQDN／IP／密鑰  
- [ ] 檔名建議存成：`assets/architecture-v2.png`  
- [ ] 嵌入位置：`docs/02-system-architecture.md`（v1 改為「演進／求職路徑圖」、v2 為「現況運行架構」）

---

## 10. 與既有圖的分工（生成時勿混題）

| 圖 | 回答什麼問題 |
|----|----------------|
| `architecture-v1.png` | 2 週節奏＋求職演進路徑 |
| **`architecture-v2.png`（本規格）** | 現在機器上怎麼跑、權責怎麼拆 |
| `personal_ai_platform_*.jpg` | 知識目錄四層概念流 |
| `docs/assets/pkos-overview.png` | PKOS 習慣／邏輯域 |

---

## 11. 文件回填（出圖後）

出圖並放入 `assets/architecture-v2.png` 後，建議同步：

1. `docs/02-system-architecture.md`：新增「架構圖（v2）現況」區塊  
2. `README.md`：架構連結改以 v2 為主、v1 為演進補充  
3. `docs/02` 架構版本紀錄加一列 v0.4／圖資 v2  
