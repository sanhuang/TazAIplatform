# 04 — Personal Knowledge Operating System（知識作業系統）

> **文件狀態：** Phase 2a 中後期  
> **最後更新：** 2026-08-14  
> **來源整併：** 原 `howto_AIuser_to_AIPlaform_builder/` 筆記（2026-08-01 併入本文件）；入口摘要見 [README.md](../README.md#來源與轉型脈絡ai-使用者--platform-builder)  
> **相關：** [03-roadmap.md](./03-roadmap.md) · [workflows/knowledge-import.md](../workflows/knowledge-import.md) · [SKILLS.md](../SKILLS.md)

![PKOS 總覽](./assets/pkos-overview.png)

> 上圖為作品集**邏輯／習慣**資訊圖。實體儲存在獨立 repo **TazKnowledges**（生命週期目錄 + Obsidian Vault），見下方「實體 vs 邏輯」。

---

## 為什麼現在寫這份文件？

這是 **AI 使用者 → AI Platform Builder** 的分水嶺：重點不再是「如何問出更好的 Prompt」，而是 **如何累積 AI 可重用資產**。

| 具體目標 | 說明 | 現況 |
|----------|------|------|
| knowledge-builder | TazInfra Skill；inbox → staging draft → 策展 | Skill ✅；E2E 習慣 🔄 |
| 格式轉換 | 地端素材轉 md／json 等可索引格式 | 進行中 |
| 習慣 | 重要成果提煉成 Markdown，而非保存整段對話 | 進行中 |
| Vault／治理 | TazKnowledges + kb-ID + verified／rag-include | ✅ |
| Keyword ingest | chunks + keyword index（非向量） | ✅ |
| 向量 RAG | embedding + vector-db | ⏳ Phase 2b |

路線圖位置：見 [03-roadmap.md](./03-roadmap.md) Phase 2a。

---

## 實體 vs 邏輯

| 層 | 是什麼 | 在哪 |
|----|--------|------|
| **邏輯域**（作品集敘事） | `aws`／`k8s`／`sre`／`stocks`／`career`／`workflow`／`projects`／`adr` | 本文件與場景描述用的分類 |
| **實體儲存**（權威） | `rawdata → aigen → obsidian → rag` | **TazKnowledges** repo |
| **發布／模板** | 少數 published 資產 | TazKnowledges `knowledge/published/`（≠ 整個扁平知識樹） |
| **消費端** | OpenClaw bind mount（Vault／raw 唯讀；`aigen/openclaw` 可寫） | **TazClaw** |

### 邏輯域 → 實體路徑（對照）

| 邏輯域 | 實體落點（TazKnowledges） |
|--------|---------------------------|
| 技術（aws／k8s／sre…） | `obsidian/Knowledge/01-Tech/…` |
| 投資／stocks | `obsidian/Knowledge/02-Finance/…`（個股研究仍偏薄） |
| career | `obsidian/Career/…`（目前最厚） |
| workflow | `obsidian/…` 流程筆記 + `automation/` 腳本／工作流說明 |
| projects | `obsidian/Projects/…` |
| adr | 平台 ADR 在本作品集 `docs/adr/`；長期可同步可檢索副本進 Vault |
| 原始素材 | `rawdata/`（勿直接當 RAG 語料） |
| 執行期索引 | `rag/`（chunks／keyword 已有；embedding／vector-db 待 2b） |

**原則：** 不必為了對齊資訊圖而複製一套扁平 `knowledge/aws|…` 目錄；Agent 與 RAG 以 **策展後、帶 `kb-*` 的 Vault 筆記** 為準。

---

## Q1：RAG 友善格式——該改儲存習慣嗎？

**A：是，但不要為了 RAG 而 RAG。**

重點不是換格式本身，而是讓知識 **可讀、可版本控制、可被 Agent 重用**。

### 五類知識資產

#### 1. 技術知識（最適合 Markdown）

範例：AWS、Kubernetes、CKAD、DVA、Observability  

建議：`.md` — 可讀、可 Git、可 RAG  

邏輯路徑範例（敘事用）：

```
knowledge/aws/lambda-best-practice.md
knowledge/k8s/ckad/deployment.md
knowledge/observability/grafana-label-design.md
```

實體則寫入 Vault 對應主題樹，並發 `kb-*` ID。

#### 2. 投資研究

範例：個股、法說會、財報  

建議：Markdown + CSV  

```
knowledge/stocks/adata/
├─ 2026Q2-report.md
├─ earnings.csv
└─ news-summary.md
```

| 格式 | 適合內容 |
|------|----------|
| CSV | EPS、營收、股利 |
| Markdown | 分析、觀點、結論 |

#### 3. 技術決策（高履歷價值）

建議：ADR（Architecture Decision Record）  

本作品集已有：[`adr/0001-orchestrator-openclaw.md`](./adr/0001-orchestrator-openclaw.md)。  

長期可同步一份到 Vault 供 Agent 檢索。

#### 4. 個人流程

範例：求職流程、證照學習、股票分析流程  

建議：流程類 Markdown — 未來 Agent 可直接讀取。

#### 5. 原始素材

範例：PDF、圖片、簡報  

原則：不要丟掉；PDF **是知識來源，不是知識庫**。  

```
resume.pdf  →  resume-summary.md  →  再進 RAG
```

實體：保留在 `rawdata/`，精煉後進 `obsidian/`。

### 格式優先級

| 優先級 | 格式 |
|--------|------|
| ★★★★★ | Markdown |
| ★★★★☆ | CSV |
| ★★★☆☆ | JSON |
| ★★☆☆☆ | PDF |
| ★☆☆☆☆ | Word |

**結論：** 長期第二大資產（僅次於可運轉的平台本身）是 **Markdown Knowledge Base**。

---

## Q2：過去大量 GPT／Gemini／Cursor 問答怎麼用？

**A：不要存對話，要提煉知識。**

RAG ≠ 把對話全部丟進去。Chat 裡混有問題、回答、修正、閒聊、錯誤答案——Agent 需要的是提煉後的知識。

| 錯誤 | 正確 |
|------|------|
| 保存 100 輪「EKS 升級」對話 | 提煉成一篇 Upgrade Runbook（Goal／Steps／Risks／Rollback） |

### OpenClaw 應該吃什麼？

不是 Chat History，而是 Knowledge（策展後的 Vault／發布資產）：

```
邏輯域（敘事）          實體（TazKnowledges）
aws / k8s / sre    →   obsidian/Knowledge/01-Tech/…
stocks             →   obsidian/Knowledge/02-Finance/…
career             →   obsidian/Career/…
workflow           →   流程筆記 + automation/
projects           →   obsidian/Projects/…
adr                →   docs/adr/（作品集）+ 可選 Vault 副本
```

### Agent 如何運作（目標）

任務：「請幫我分析某檔個股是否值得加碼」

1. 搜尋知識庫中該標的相關筆記／CSV（經 RAG 或檔案檢索）
2. 找到歷史分析、財報、法說、持股策略
3. 再交給 LLM 綜合判斷

---

## 匯入工作流（本專案）

| 步驟 | 工具／路徑 |
|------|------------|
| Skill | `knowledge-builder`（權威：`TazInfra/skills/knowledge-builder/`，見 [SKILLS.md](../SKILLS.md)） |
| 工作流說明 | [workflows/knowledge-import.md](../workflows/knowledge-import.md) |
| 輸入 | Skill／工作流約定的 inbox（例：ChatGPT export；實體常落在 TazKnowledges `rawdata/` 或約定 inbox） |
| 產出 | staging draft（審過前不寫向量庫、不直接當正式 Vault） |
| 定庫 | 審閱通過 → Obsidian（`kb-*`）→ keyword ingest；Phase 2b 再 Embedding |

```
Export → 過濾分類 → 提煉結構化 → staging draft
                                      │ review
                                      ▼
                               obsidian/（kb-*）
                                      │ ingest
                                      ▼
                         rag/chunks + keyword  →（Phase 2b）vector
                                      │
                                      ▼
                                   Agent
```

---

## 當前兩週行動（Phase 2a 中後期）

不是 Kubernetes、不是「從空目錄開始」——而是把已有 Vault／ingest **用滿**：

1. **補精煉資產**：技術（AWS／K8s／SRE）與 stocks 筆記加厚；維持 Career 品質
2. **跑通 knowledge-builder E2E**：一份真實 inbox → staging draft → 策展進 Vault
3. **養成習慣**：Cursor／Chat 重要成果 → 一篇 Markdown，不存對話
4. **準備 Phase 2b**：ADR-003 向量庫選型；在既有 chunks 上規劃 embedding

> **如果只能做一件事：** 把下一個真實問題的結論寫成帶 `kb-*` 的 Markdown，並標 `rag-include`。長期報酬率通常高於再學一個 AI Framework。

---

## 與平台其他文件的關係

| 文件 | 關係 |
|------|------|
| [01-business-goals.md](./01-business-goals.md) | 技術文件搜尋、求職、台股場景依賴本知識層 |
| [02-system-architecture.md](./02-system-architecture.md) | Phase 2 資料流接在三 repo + TazKnowledges 之上 |
| [03-roadmap.md](./03-roadmap.md) | Phase 2a／2b 待辦與完成標準 |
| howto 目錄 | 原始筆記與圖；**以本檔為專案內權威版本** |
