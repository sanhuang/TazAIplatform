# N8n / Infra / OpenClaw — 服務架構、權責與安全邊界

> **狀態：** 對應目前地端部署（2026-07）；**隨部署／網路／權責變更持續更新**  
> **作品集去敏版：** [docs/02-system-architecture.md](./docs/02-system-architecture.md)  
> **階段：** Phase 0～1 完成；平台敘事進度見 [docs/03-roadmap.md](./docs/03-roadmap.md)  
> **入口：** TazInfra → TazClaw / TazN8n（及其他應用）  
> **互補：** 知識／目錄四層概念圖與互動矩陣見 [personal_ai_platform_architecture.md](./personal_ai_platform_architecture.md)（圖：[personal_ai_platform_1786241579432.jpg](./personal_ai_platform_1786241579432.jpg)）  
> **相關：** [README.md](./README.md) · 應用／Infra 各 repo README · [`../TazInfra/docs/caddy-netbird.md`](../TazInfra/docs/caddy-netbird.md) · [`../TazInfra/docs/taz-shared-network.snippet.yml`](../TazInfra/docs/taz-shared-network.snippet.yml)

---

## 1. 總覽（三 repo 權責）

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         Host: M2 Max（地端）                              │
│                                                                          │
│  ┌─ TazInfra ─────────────────────────────────────────────────────────┐  │
│  │  共用基礎設施（最先啟動）                                            │  │
│  │  • taz-shared network                                               │  │
│  │  • taz-postgres / taz-redis                                         │  │
│  │  • taz-caddy（HTTPS 邊緣 :80/:443）                                  │  │
│  │  • NetBird 腳本（主機級 VPN，非 container）                           │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│           │ 提供 network / DB / Redis / TLS 終止                          │
│           ▼                                                              │
│  ┌─ TazClaw ──────────────┐    ┌─ TazN8n ─────────────────────────────┐ │
│  │  OpenClaw 應用          │    │  n8n Queue Mode 應用                 │ │
│  │  • openclaw-gateway     │    │  • n8n (main UI/API)                 │ │
│  │  • openclaw-cli (opt)   │    │  • n8n-worker × N（--scale）         │ │
│  │  Agent / workspace / cron│    │  Workflow 編排與排程執行              │ │
│  └────────────────────────┘    └──────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────┘
```

| Repo | 擁有什麼 | 不擁有什麼 |
|------|----------|------------|
| **TazInfra** | `taz-shared`、Postgres、Redis、Caddy、NetBird 連線腳本 | 業務工作流、Agent prompt、n8n credentials 內容 |
| **TazClaw** | OpenClaw gateway/cli、workspace、agent 設定、API keys（應用側） | 共用 DB/Redis 實例、邊緣 TLS、VPN |
| **TazN8n** | n8n main + workers、本機 `n8n_data` volume、encryption key | Postgres/Redis 容器本體（連 TazInfra） |

---

## 2. 網路拓撲（Docker + 遠端存取）

```
                    ┌─────────────────────┐
                    │  M1 Pro（遠端）      │
                    │  Browser / curl     │
                    └──────────┬──────────┘
                               │ NetBird VPN（WireGuard）
                               │ DNS Custom Zone（僅 VPN 內解析）
                               │   tazclaw.personalwork.tw → M2 peer
                               │   n8n.personalwork.tw     → M2 peer
                               ▼
┌──────────────────────────────────────────────────────────────────────────┐
│  M2 Max                                                                  │
│                                                                          │
│   NetBird IP :443/:80 ──► Local Forwarding ──► taz-caddy                 │
│                                                                          │
│   ┌─ Docker network: taz-shared ──────────────────────────────────────┐  │
│   │                                                                    │  │
│   │   taz-caddy                                                        │  │
│   │     │ reverse_proxy                                                │  │
│   │     ├─ CADDY_DOMAIN  ──► openclaw-gateway:18789                    │  │
│   │     ├─ N8N_DOMAIN    ──► n8n:5678                                  │  │
│   │     └─ IMMICH_DOMAIN ──► immich-server:2283（其他 repo）            │  │
│   │                                                                    │  │
│   │   openclaw-gateway ◄──┐                                            │  │
│   │   n8n / n8n-worker    │  TCP                                       │  │
│   │   taz-postgres        │                                            │  │
│   │   taz-redis           │                                            │  │
│   │                                                                    │  │
│   └────────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   Host loopback（僅本機，不經 NetBird）:                                  │
│     127.0.0.1:18789  → openclaw-gateway                                  │
│     127.0.0.1:18790  → openclaw bridge                                   │
│     127.0.0.1:5432   → taz-postgres                                      │
│     127.0.0.1:6379   → taz-redis                                         │
│     127.0.0.1:15678  → n8n（宿主發布埠；容器內仍為 5678）                 │
└──────────────────────────────────────────────────────────────────────────┘
```

**原則：** 公開網際網路**不**解析這些個人網域到 NetBird IP；遠端一律走 VPN + Caddy HTTPS。

---

## 3. 元件級 ASCII（職責）

### 3.1 TazInfra

```
┌────────────────────────── TazInfra ──────────────────────────┐
│                                                              │
│  taz-caddy          TLS 終止 / ACME（Cloudflare DNS-01）      │
│                     依 Host 反代至各應用容器                   │
│                                                              │
│  taz-postgres       共用 PostgreSQL（分 database）            │
│                     預設 DB=n8n；未來 openclaw 可另開 DB       │
│                     publish: 127.0.0.1:5432                   │
│                                                              │
│  taz-redis          共用 Redis（分 DB index / 密碼）          │
│                     n8n queue（Bull）使用                     │
│                     publish: 127.0.0.1:6379                   │
│                                                              │
│  taz-shared         bridge network；跨 compose 唯一信任平面   │
│                                                              │
│  scripts/           NetBird up / forwarding / HTTPS 測試      │
│  （主機程序）        非 Docker service                        │
└──────────────────────────────────────────────────────────────┘
```

### 3.2 TazClaw（OpenClaw）

```
┌────────────────────────── TazClaw ───────────────────────────┐
│                                                              │
│  openclaw-gateway                                            │
│    • Control UI / API / Agent 編排                            │
│    • 掛載 .openclaw + workspace                              │
│    • Documents 唯讀掛載 → sources/documents                   │
│    • 接 LLM（Codex / 可選 API keys）                          │
│    • networks: default + taz-shared                          │
│    • host publish: 127.0.0.1:18789/18790                     │
│                                                              │
│  openclaw-cli（profile: cli）                                │
│    • network_mode: service:openclaw-gateway                  │
│    • cron add/edit、一次性管理指令                            │
│    • cap_drop NET_RAW/NET_ADMIN；no-new-privileges           │
│                                                              │
│  應用資料邊界                                                 │
│    • 設定 / 記憶 / projects 在 OPENCLAW_*_DIR（主機路徑）      │
│    • 目前不依賴 taz-postgres（可選未來接入）                   │
└──────────────────────────────────────────────────────────────┘
```

### 3.3 TazN8n

```
┌────────────────────────── TazN8n ────────────────────────────┐
│                                                              │
│  n8n (main)                                                  │
│    • UI / API / Webhook 入口                                  │
│    • EXECUTIONS_MODE=queue                                   │
│    • DB → taz-postgres；Queue → taz-redis                    │
│    • volume: tazn8n_n8n_data（本機檔案，非 DB）               │
│    • host publish: :15678 → 容器 :5678（見 .env N8N_PORT）   │
│                                                              │
│  n8n-worker × N                                              │
│    • 執行任務；不對外開 port                                  │
│    • --scale n8n-worker=2|4                                  │
│    • OFFLOAD_MANUAL_EXECUTIONS_TO_WORKERS=true               │
│                                                              │
│  僅加入 taz-shared（external）；不建立共用網路 / 不帶 DB 容器  │
└──────────────────────────────────────────────────────────────┘
```

### 3.4 請求路徑（遠端 → 應用）

```
  HTTPS Client
       │
       ▼
  NetBird ACL（允許 peer → M2 TCP 443）
       │
       ▼
  taz-caddy（SNI / Host）
       │
       ├── Host: tazclaw.personalwork.tw ──► openclaw-gateway:18789
       │         + gateway.auth.token / device approve
       │
       └── Host: n8n.personalwork.tw ────► n8n:5678
                 + n8n 自身登入 / credentials
```

---

## 4. 權責對照（誰管什麼）

```
                    建置/擁有          設定檔位置           運轉依賴
─────────────────────────────────────────────────────────────────────
taz-shared          TazInfra          compose networks     所有應用先決
Postgres/Redis      TazInfra          TazInfra/.env        TazN8n（必）
Caddy / TLS         TazInfra          Caddyfile + .env     遠端 HTTPS
NetBird VPN         TazInfra scripts  .env.netbird         遠端存取
OpenClaw gateway    TazClaw           TazClaw/.env         Caddy 反代可選
OpenClaw agents     TazClaw           .openclaw/           gateway
n8n main/worker     TazN8n            TazN8n/.env          Infra DB/Redis
n8n encryption key  TazN8n            N8N_ENCRYPTION_KEY   憑證解密
密碼對齊            兩側 .env 一致     POSTGRES_/REDIS_     連線成功
```

### 啟動順序

```
1) cd TazInfra && docker compose up -d --build
2) （主機）cd TazInfra && ./scripts/netbird-run-watchdog.sh
3) cd TazClaw  && docker compose up -d
4) cd TazN8n   && docker compose up -d --scale n8n-worker=2
```

Infra **不** `depends_on` 應用：Caddy 可先起；上游未就緒時該 site 暫不可用，Caddy 本身仍應 healthy。

---

## 5. 安全邊界

### 5.1 信任分層

```
┌─ Layer 0：公開網際網路 ─────────────────────────────────────┐
│  不承載 Control UI / n8n；網域勿在公開 DNS 指到 NetBird IP   │
└─────────────────────────────────────────────────────────────┘
        │ 僅 NetBird peer 可達
        ▼
┌─ Layer 1：NetBird VPN ──────────────────────────────────────┐
│  • Peer 身分 + ACL（TCP 443 等）                             │
│  • Custom Zone DNS（私人解析）                               │
│  • macOS Local Forwarding（M2 將 NB IP:443 → 本機 Caddy）   │
└─────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ Layer 2：邊緣 TLS（taz-caddy）──────────────────────────────┐
│  • Let's Encrypt（Cloudflare DNS-01）                        │
│  • HTTP → HTTPS 轉址                                        │
│  • 依 Host 反代；終止 TLS 後走 taz-shared 明文 HTTP 至上游   │
└─────────────────────────────────────────────────────────────┘
        │ 僅 Docker 內網
        ▼
┌─ Layer 3：taz-shared（跨專案信任平面）───────────────────────┐
│  • openclaw-gateway / n8n / postgres / redis / caddy 互通    │
│  • 不應把不可信容器隨意加入此 network                        │
└─────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ Layer 4：應用身分 ─────────────────────────────────────────┐
│  OpenClaw：OPENCLAW_GATEWAY_TOKEN + Control UI 裝置核准       │
│  n8n：使用者登入 + N8N_ENCRYPTION_KEY（credentials 加密）    │
│  Postgres/Redis：密碼（.env；宿主僅 127.0.0.1 publish）      │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 埠口與暴露面

| 服務 | 監聽 / 發布 | 邊界含義 |
|------|-------------|----------|
| `taz-caddy` `:80/:443` | 宿主 `0.0.0.0`（經 NetBird 使用） | **唯一**建議遠端入口 |
| `openclaw-gateway` | `127.0.0.1:18789/18790` | 不直連遠端；由 Caddy 反代 |
| `n8n` | 宿主 `N8N_PORT`（例 15678） | 本機除錯用；遠端應走 `N8N_DOMAIN` HTTPS |
| `postgres` / `redis` | `127.0.0.1:5432/6379` | 僅本機工具；容器間用服務名 |
| `n8n-worker` | 無 host port | 只經 Redis queue 接單 |

### 5.3 資料與密鑰邊界

```
┌─ 主機檔案系統 ──────────────────────────────────────────────┐
│  TazClaw/.openclaw/**          Agent 設定、記憶、workspace   │
│  TazClaw/.env                  Gateway token、通知 URL 等    │
│  Documents（ro 掛載）          個人文件；容器內唯讀           │
│  TazN8n volume n8n_data        n8n 本機狀態檔                │
│  TazInfra volumes              Postgres/Redis 持久層         │
│  tazclaw_caddy_data（external）LE 憑證（自舊 TazClaw 沿用）  │
└─────────────────────────────────────────────────────────────┘

規則：
  • .env / API token / DB 密碼 不進 git
  • TazInfra 與 TazN8n 的 POSTGRES_*/REDIS_* 必須一致
  • 更換 N8N_ENCRYPTION_KEY 會使既有 credentials 無法解密
  • Immich 自有 Postgres：不要併入 TazInfra（見 TazInfra README）
```

### 5.4 明確「不跨越」的邊界

| 從 → 到 | 是否允許 | 說明 |
|---------|----------|------|
| 公開 Internet → Gateway/n8n | ❌ | 無公開 A 記錄；靠 VPN |
| Peer → M2:18789 直連 | ⚠️ 不建議 | 應只開 ACL 443，走 Caddy |
| 任意 compose → `taz-shared` | ⚠️ 謹慎 | 等同進入共用 DB/邊緣信任域 |
| OpenClaw → 宿主 Docker socket | ❌（預設關） | sandbox 需顯式掛載才開 |
| `docker compose down -v`（Infra） | ⚠️ | 會清共用 DB/Redis；勿未備份執行 |
| n8n `down -v` | ✅ 相對安全 | 只清 `n8n_data`，不動 Postgres/Redis |

---

## 6. 故障切分（依邊界排查）

```
遠端打不開 tazclaw / n8n？
  │
  ├─ DNS（Custom Zone）對不對？     → TazInfra docs/caddy-netbird.md 問題 1
  ├─ NetBird ACL 放行 TCP 443？    → 問題 3
  ├─ M2 Local Forwarding？         → 問題 2
  ├─ taz-caddy healthy？上游起來？ → TazInfra compose / Caddyfile Host
  ├─ OpenClaw：token / device？    → TazClaw scripts approve
  └─ n8n：連得上 taz-postgres/redis？→ 密碼對齊、Infra 是否先起
```

---

## 7. 變更紀錄

| 日期 | 項目 |
|------|------|
| 2026-07-16 | 初版：以現況 compose / Caddyfile 繪製三 repo 架構、權責與安全邊界 |
| 2026-07-25 | NetBird 腳本路徑改 TazInfra；遠端連線全文見 TazInfra docs；OpenClaw 步驟見 README |
| 2026-08-01 | 去敏摘要併入 `docs/02-system-architecture.md`；路線圖校準為 Phase 2a |
