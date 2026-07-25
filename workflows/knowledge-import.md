# Workflow：knowledge-import

**Skill：** knowledge-builder（見根目錄 `SKILLS.md`）  
**權威：** 該 Skill 的 `SKILL.md`（步驟與規則以那邊為準）

## 何時用

要把 ChatGPT export／同類匯入轉成可審閱的知識 draft 時。

## 步驟

1. 確認可讀 `knowledge-builder/SKILL.md` 與 `references/`
2. 準備輸入路徑（例：`inbox/chatgpt/`；目錄不存在就建）
3. 依 Skill Workflow 跑 importer → 產出到 `staging/`（不存在就建）
4. 在 `staging/` review；通過前維持 draft，不寫向量庫

## 路徑慣例（本專案可改，改了要寫進本檔）

| 角色 | 預設 |
|------|------|
| 輸入 | `inbox/` |
| 產出 | `staging/` |

CLI 參數、category、Metadata DB 等細節：**只跟 Skill `SKILL.md`，不在本檔重複。**
