# Skills（本專案）

列出本專案會用到的 Skill。Agent 執行前讀對應 `SKILL.md`；**勿把 Skill 本體複製進本專案**。權威目錄在 **TazInfra** `skills/`（symlink／bind 取用）。

| Skill | 權威路徑 | 本專案用途 |
|-------|----------|------------|
| knowledge-builder | `TazInfra/skills/knowledge-builder/` | 非結構化 inbox → staging draft；審過後策展進 TazKnowledges Vault（見 [workflows/knowledge-import.md](./workflows/knowledge-import.md)） |
| grill-me | `TazInfra/skills/grill-me/` | 以提問收斂需求，再產出／ refinement Skill 規格（OpenClaw／Cursor 變體） |
| create-taz-project | `TazInfra/skills/create-taz-project/` | 依範本建立新 Taz 專案骨架（可選；作品集敘事較少直接引用） |

新增 Skill：在本表加一列，並在 `workflows/` 增加或更新對應工作流。
