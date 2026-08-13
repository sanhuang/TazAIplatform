# 個人 AI 平台架構概念圖與互動矩陣

> **狀態：** 隨平台現況持續更新（知識層／Agent／目錄慣例變更時同步修訂）  
> **圖檔：** [personal_ai_platform_1786241579432.jpg](./personal_ai_platform_1786241579432.jpg)  
> **互補：** 地端服務／三 repo 運維邊界見 [architecture-n8n-infra-openclaw.md](./architecture-n8n-infra-openclaw.md)；作品集去敏敘事見 [docs/02-system-architecture.md](./docs/02-system-architecture.md)  
> **入口：** [README.md](./README.md) 文件地圖

本文件描述「資料 → 處理 → 編排 → 知識庫」四層概念與目錄互動矩陣；**不**取代 Infra／OpenClaw／n8n 的部署細節。

![個人 AI 平台架構概念圖](./personal_ai_platform_1786241579432.jpg)

---

## 核心層級結構解析

### 1. 原始資料層 (Data Input & Raw Storage)
* **主要目錄**：`rawdata/`
* **包含類型**：`pdf/`、`images/`、`heic/`、`audio/`、`video/`、`chatgpt-export/` 等。
* **特性**：作為系統最底層的原始數據源，供上層處理工具（OCR、Transcript、RAG、Agents、IDE）進行讀取。

---

### 2. AI 處理與服務層 (AI Processing & Feature Services)
* **OCR 服務**：讀取 `rawdata/pdf/`、`images/`、`heic/` $\rightarrow$ 寫入 `aigen/ocr/`（保留來源 ID 與頁碼）。
* **Transcript 逐字稿**：讀取 `rawdata/audio/`、`video/` $\rightarrow$ 寫入 `aigen/transcript/`（保留時間碼與說話者）。
* **OpenClaw Agent**：參照 `rawdata/`、`obsidian/`、`knowledge/` $\rightarrow$ 寫入 `aigen/openclaw/`（不直接覆寫正式知識）。
* **Hermes Agent**（未啟用）：參照 `rawdata/`、`obsidian/`、`knowledge/` $\rightarrow$ 寫入 `aigen/hermes/`（草稿需人工審核）。
* **RAG Pipeline**：參照 `rawdata/`、`obsidian/`、`knowledge/` $\rightarrow$ 建置並寫入 `rag/`（索引可重建，不回寫來源）。

---

### 3. IDE 開發環境與自動化編排層 (AI IDEs & Orchestration)
* **Antigravity IDE / Gemini**：讀取 `rawdata/chatgpt-export/`、`obsidian/`、`automation/gemini/` $\rightarrow$ 寫入 `automation/gemini/`（統一 Git 版控）與 `aigen/`。
* **Cursor IDE**：讀取 `rawdata/`、`obsidian/`、`automation/` $\rightarrow$ 寫入 `automation/`（Rules/Prompt/Skill）與 `aigen/` 或 `obsidian/`。
* **n8n / Automation Scripts**：根據 Workflow 授權 $\rightarrow$ 寫入 `aigen/`、`rag/`、`tmp/` 與 `obsidian/`（嚴禁機密寫入 Repo）。
* **Git 版本控制**：版控 `obsidian/`、`knowledge/`、`automation/` 與根目錄指南，不納管大型或可重建數據。

---

### 4. 知識庫與產出暫存層 (Knowledge Vault & Staging Output)
* **Obsidian Vault**：`obsidian/`（終端筆記與正式知識呈現，僅開啟 Vault 目錄）。
* **Knowledge Base**：`knowledge/`（結構化知識庫）。
* **AI Generated Staging**：`aigen/`（包含 ocr, transcript, openclaw, hermes 等 AI 產出草稿與暫存區）。
