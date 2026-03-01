# SMC Commander SaaS Platform — Implementation Plan

## Context
The SMC Commander trading engine is complete (20 source files, 196 tests, 4,500+ lines). It detects ICT/SMC trade setups, scores them 0-100, calculates entries with risk management, and has a real-time event-driven engine. Now we wrap it in a full SaaS platform: API backend, database, auth, payments, notifications, WebSocket feed, and a Next.js dashboard.

**Engine code at `src/` is untouched.** SaaS layer imports from it.

## Architecture

```
User Browser  ←→  Next.js Frontend (web/)
                        ↕ REST + WebSocket
                  Fastify API Server (saas/)
                        ↕
              ┌─────────┼─────────┐
         PostgreSQL   EnginePool   Market Data
         (Drizzle)    (per-user    (Binance WS)
                      RealtimeEngine
                      instances)
```

## Directory Structure

```
SMC TRADING SAAS/
├── src/                          # EXISTING ENGINE (UNTOUCHED)
├── tests/                        # EXISTING TESTS (UNTOUCHED)
├── saas/                         # NEW: API BACKEND
│   ├── package.json
│   ├── tsconfig.json
│   ├── drizzle.config.ts
│   ├── .env.example
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── src/
│       ├── server.ts             # Fastify bootstrap
│       ├── config.ts             # Zod env validation
│       ├── db/
│       │   ├── schema.ts         # All tables (Drizzle pgTable)
│       │   ├── client.ts         # pg Pool + drizzle instance
│       │   └── migrate.ts        # Migration runner
│       ├── auth/
│       │   ├── password.ts       # argon2 hash/verify
│       │   ├── jwt.ts            # jose access+refresh tokens
│       │   ├── api-keys.ts       # crypto key gen + SHA-256
│       │   ├── middleware.ts     # Fastify onRequest hooks
│       │   └── routes.ts        # /auth/register, login, refresh, logout
│       ├── users/
│       │   ├── service.ts
│       │   └── routes.ts
│       ├── subscriptions/
│       │   ├── tiers.ts          # Tier limits + feature gates
│       │   └── service.ts
│       ├── billing/
│       │   ├── stripe.ts         # Stripe client init
│       │   ├── checkout.ts       # Checkout session
│       │   ├── portal.ts         # Customer portal
│       │   └── webhooks.ts       # Stripe webhook handler
│       ├── engine-manager/
│       │   ├── pool.ts           # EnginePool: manages RealtimeEngine per user
│       │   ├── bridge.ts         # EngineBridge: events → DB + WS + notifications
│       │   ├── routes.ts         # /engine/start, stop, status
│       │   └── data-providers/
│       │       ├── interface.ts  # MarketDataProvider abstract
│       │       ├── binance.ts    # Binance WS (crypto)
│       │       └── aggregator.ts # Tick→candle aggregation
│       ├── trades/
│       │   ├── service.ts        # Replaces TradeJournal (DB-backed)
│       │   └── routes.ts
│       ├── signals/
│       │   ├── service.ts
│       │   └── routes.ts
│       ├── instruments/
│       │   ├── service.ts
│       │   └── routes.ts
│       ├── notifications/
│       │   ├── dispatcher.ts     # Central dispatch to all channels
│       │   ├── telegram.ts
│       │   ├── discord.ts
│       │   ├── email.ts          # Resend
│       │   ├── webhook.ts        # Custom webhooks with HMAC
│       │   └── routes.ts
│       ├── ws/
│       │   ├── handler.ts        # WebSocket upgrade + registry
│       │   ├── protocol.ts       # WS message types
│       │   └── auth.ts           # WS JWT verification
│       ├── middleware/
│       │   ├── rate-limit.ts
│       │   ├── error-handler.ts
│       │   └── request-id.ts
│       ├── shared/
│       │   ├── errors.ts         # AppError, UnauthorizedError, etc.
│       │   ├── pagination.ts
│       │   └── logger.ts         # Pino config
│       └── health/
│           └── routes.ts
├── web/                          # NEW: NEXT.JS FRONTEND
│   ├── package.json
│   ├── next.config.ts
│   ├── tailwind.config.ts
│   └── src/
│       ├── app/
│       │   ├── layout.tsx
│       │   ├── page.tsx          # Landing page
│       │   ├── pricing/page.tsx
│       │   ├── (auth)/login/page.tsx
│       │   ├── (auth)/register/page.tsx
│       │   └── (dashboard)/
│       │       ├── layout.tsx    # Dashboard shell
│       │       ├── dashboard/page.tsx
│       │       ├── trades/page.tsx
│       │       ├── signals/page.tsx
│       │       ├── settings/page.tsx
│       │       └── billing/page.tsx
│       ├── lib/
│       │   ├── api.ts            # Fetch wrapper + token refresh
│       │   ├── ws.ts             # WS client with reconnect
│       │   └── auth-context.tsx
│       └── components/
│           ├── signal-card.tsx
│           ├── trade-table.tsx
│           ├── stats-panel.tsx
│           └── price-chart.tsx
└── package.json                  # Root workspace config
```

## Database Schema (10 tables)

### users
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | gen_random_uuid() |
| email | VARCHAR(255) UNIQUE | NOT NULL |
| password_hash | VARCHAR(255) | argon2 |
| name | VARCHAR(255) | |
| role | VARCHAR(20) | FREE/STARTER/PRO/ELITE/ADMIN |
| stripe_customer_id | VARCHAR(255) UNIQUE | |
| created_at / updated_at | TIMESTAMPTZ | |

### subscriptions
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | CASCADE |
| stripe_subscription_id | VARCHAR(255) UNIQUE | |
| tier | VARCHAR(20) | FREE/STARTER/PRO/ELITE |
| status | VARCHAR(20) | active/past_due/canceled |
| current_period_start/end | TIMESTAMPTZ | |
| api_calls_used / api_calls_limit | INTEGER | |

### api_keys
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | |
| key_hash | VARCHAR(255) | SHA-256 |
| key_prefix | VARCHAR(8) | for display (smc_xxxx) |
| name | VARCHAR(100) | |
| revoked | BOOLEAN | |

### user_instruments
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | |
| symbol | VARCHAR(20) | ES, BTCUSD, etc. |
| capital | DECIMAL(14,2) | Account balance |
| max_risk_per_trade | DECIMAL(5,4) | default 0.01 |
| max_daily_drawdown | DECIMAL(5,4) | default 0.02 |
| auto_trade | BOOLEAN | default false |
| UNIQUE(user_id, symbol) | | |

### trades
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | |
| trade_id | VARCHAR(50) | Engine-generated (ES-1) |
| symbol, direction, entry_price, stop_loss, tp1/2/3 | | Matches TradeRecord |
| position_size, risk_amount, risk_percent, score, grade | | |
| result | VARCHAR(10) | WIN/LOSS/BE/NULL |
| exit_price, pnl, closed_at | | Nullable until closed |
| trade_date | DATE | |
| INDEX(user_id, symbol), INDEX(user_id, trade_date) | | |

### signals
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | |
| symbol, is_tradable, direction, score, grade | | Matches SignalLogEntry |
| disqualifications | TEXT[] | Postgres array |
| signal_data | JSONB | Full SignalResult for detail |
| INDEX(user_id, symbol, timestamp DESC) | | |

### session_states
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | |
| symbol | VARCHAR(20) | |
| consecutive_losses, daily_pnl, weekly_pnl, trades_count | | Matches SessionState |
| is_halted, halt_reason | | |
| UNIQUE(user_id, symbol) | | |

### notification_settings
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | |
| channel | VARCHAR(20) | telegram/discord/email/webhook |
| enabled | BOOLEAN | |
| config | JSONB | {chatId, token} / {webhookUrl} / {url, secret} |
| events | TEXT[] | signal, trade_entry, trade_exit, session_halt |

### refresh_tokens
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | |
| token_hash | VARCHAR(255) | |
| expires_at | TIMESTAMPTZ | 7 days |
| revoked | BOOLEAN | |

### audit_log
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| user_id | UUID FK→users | SET NULL |
| action | VARCHAR(100) | user.login, trade.entry, etc. |
| metadata | JSONB | |
| ip_address | INET | |
| created_at | TIMESTAMPTZ | INDEX DESC |

## Subscription Tiers

| Feature | FREE | STARTER $49 | PRO $129 | ELITE $299 |
|---------|------|-------------|----------|------------|
| Instruments | 1 | 3 | 5 | 7 (all) |
| WS connections | 1 | 2 | 5 | 10 |
| API calls/month | 100 | 5,000 | 50,000 | Unlimited |
| Notifications | Email | +Telegram | +Discord, Webhook | All |
| Auto-trade | No | No | Yes | Yes |
| Rate limit | 10/min | 60/min | 300/min | 1000/min |

## API Endpoints

### Auth
- `POST /api/auth/register` — Create account
- `POST /api/auth/login` — Returns JWT + refresh token
- `POST /api/auth/refresh` — New token pair
- `POST /api/auth/logout` — Revoke refresh token

### Users
- `GET /api/users/me` — Profile
- `PATCH /api/users/me` — Update profile
- `GET/POST/DELETE /api/users/me/api-keys` — API key management

### Engine
- `POST /api/engine/start` — Start engine for symbol
- `POST /api/engine/stop` — Stop engine
- `GET /api/engine/status` — All running engines
- `POST /api/engine/session/reset` — Reset circuit breaker
- `PUT /api/engine/macro` — Update DXY/VIX/news

### Instruments
- `GET /api/instruments` — Available instruments (from INSTRUMENTS constant)
- `GET /api/instruments/mine` — User configs
- `PUT /api/instruments/mine/:symbol` — Upsert config
- `DELETE /api/instruments/mine/:symbol` — Remove

### Trades & Signals
- `GET /api/trades` — Paginated, filterable
- `GET /api/trades/:id` — Detail
- `POST /api/trades/:id/close` — Manual close
- `GET /api/trades/stats` — JournalStats
- `GET /api/signals` — Recent signals
- `GET /api/signals/:id` — Full SignalResult detail

### Notifications
- `GET/POST/PATCH/DELETE /api/notifications` — CRUD
- `POST /api/notifications/test` — Test send

### Billing
- `GET /api/billing/plans` — Tier info
- `POST /api/billing/checkout` — Stripe Checkout
- `POST /api/billing/portal` — Stripe Portal
- `POST /api/webhooks/stripe` — Webhook handler

### WebSocket Protocol
Connect: `wss://host/ws?token=JWT`

Client→Server: `subscribe {symbols}`, `unsubscribe {symbols}`, `ping`
Server→Client: `signal`, `trade_entry`, `trade_exit`, `session_halt`, `candle`, `pong`, `error`

## Build Order (75 steps, 14 phases)

### Phase 1: Scaffolding (steps 1-8)
1. Root `package.json` with workspaces `["saas", "web"]`
2. `saas/package.json` — fastify, @fastify/websocket, @fastify/rate-limit, @fastify/cors, drizzle-orm, pg, argon2, jose, zod, pino, stripe, nanoid
3. `saas/tsconfig.json` — paths: `@engine/* → ../src/*`
4. `saas/.env.example`
5. `saas/src/config.ts` — Zod env parsing
6. `saas/src/shared/logger.ts` — Pino
7. `saas/src/shared/errors.ts` — AppError hierarchy
8. `saas/src/shared/pagination.ts`

### Phase 2: Database (steps 9-12)
9. `saas/src/db/schema.ts` — All 10 tables with Drizzle pgTable
10. `saas/src/db/client.ts` — Pool + drizzle instance
11. `saas/drizzle.config.ts`
12. `saas/src/db/migrate.ts`

### Phase 3: Auth (steps 13-17)
13. `saas/src/auth/password.ts`
14. `saas/src/auth/jwt.ts`
15. `saas/src/auth/api-keys.ts`
16. `saas/src/auth/middleware.ts`
17. `saas/src/auth/routes.ts`

### Phase 4: Users & Subscriptions (steps 18-21)
18. `saas/src/users/service.ts`
19. `saas/src/users/routes.ts`
20. `saas/src/subscriptions/tiers.ts`
21. `saas/src/subscriptions/service.ts`

### Phase 5: Billing (steps 22-25)
22. `saas/src/billing/stripe.ts`
23. `saas/src/billing/checkout.ts`
24. `saas/src/billing/portal.ts`
25. `saas/src/billing/webhooks.ts`

### Phase 6: Engine Manager — Core Integration (steps 26-31)
26. `saas/src/engine-manager/data-providers/interface.ts`
27. `saas/src/engine-manager/data-providers/binance.ts`
28. `saas/src/engine-manager/data-providers/aggregator.ts`
29. `saas/src/engine-manager/bridge.ts` — Engine events → DB + WS + notifications
30. `saas/src/engine-manager/pool.ts` — Multi-tenant engine lifecycle
31. `saas/src/engine-manager/routes.ts`

### Phase 7: Trades & Signals (steps 32-35)
32. `saas/src/trades/service.ts` — DB-backed TradeJournal replacement
33. `saas/src/trades/routes.ts`
34. `saas/src/signals/service.ts`
35. `saas/src/signals/routes.ts`

### Phase 8: Instruments (steps 36-37)
36. `saas/src/instruments/service.ts`
37. `saas/src/instruments/routes.ts`

### Phase 9: Notifications (steps 38-43)
38. `saas/src/notifications/telegram.ts`
39. `saas/src/notifications/discord.ts`
40. `saas/src/notifications/email.ts`
41. `saas/src/notifications/webhook.ts`
42. `saas/src/notifications/dispatcher.ts`
43. `saas/src/notifications/routes.ts`

### Phase 10: WebSocket (steps 44-46)
44. `saas/src/ws/protocol.ts`
45. `saas/src/ws/auth.ts`
46. `saas/src/ws/handler.ts`

### Phase 11: Middleware & Server (steps 47-52)
47. `saas/src/middleware/rate-limit.ts`
48. `saas/src/middleware/error-handler.ts`
49. `saas/src/middleware/request-id.ts`
50. `saas/src/health/routes.ts`
51. `saas/src/server.ts` — Full bootstrap
52. `saas/docker-compose.yml` + `saas/Dockerfile`

### Phase 12: Frontend Setup (steps 53-60)
53. `web/package.json` — next, react, tailwindcss, @tanstack/react-query, zustand, recharts
54. `web/tailwind.config.ts` + `web/next.config.ts` + `web/tsconfig.json`
55. `web/src/lib/api.ts` — Fetch wrapper + auto-refresh
56. `web/src/lib/ws.ts` — WS client with reconnect
57. `web/src/lib/auth-context.tsx`
58. `web/src/app/layout.tsx` — Root with providers
59. `web/src/app/page.tsx` — Landing page
60. `web/src/app/pricing/page.tsx`

### Phase 13: Frontend Dashboard (steps 61-70)
61. `web/src/app/(auth)/login/page.tsx`
62. `web/src/app/(auth)/register/page.tsx`
63. `web/src/app/(dashboard)/layout.tsx` — Sidebar + header
64. `web/src/components/signal-card.tsx`
65. `web/src/components/trade-table.tsx`
66. `web/src/components/stats-panel.tsx`
67. `web/src/app/(dashboard)/dashboard/page.tsx` — Active signals, P&L, engines
68. `web/src/app/(dashboard)/trades/page.tsx` — Trade journal
69. `web/src/app/(dashboard)/signals/page.tsx` — Real-time signal feed
70. `web/src/app/(dashboard)/settings/page.tsx` — Instruments, risk, notifications, API keys

### Phase 14: Tests & Verification (steps 71-75)
71. `saas/tests/setup.ts` — Test DB
72. `saas/tests/auth.test.ts`
73. `saas/tests/engine-pool.test.ts`
74. `saas/tests/trades.test.ts`
75. `saas/tests/ws.test.ts`

## Key Integration: How Engine Connects to SaaS

### EnginePool (multi-tenant core)
```
User calls POST /engine/start {symbol: "BTCUSD"}
  → EnginePool.startEngine(userId, "BTCUSD")
    → Load user_instruments config from DB
    → Check tier limits (max instruments)
    → new RealtimeEngine({symbol, capital, ...config})
    → new EngineBridge(engine, userId, symbol, {db, ws, notifications})
    → Connect BinanceProvider → engine.pushLTFCandle / pushHTFCandle
    → engine.start()
```

### EngineBridge (event adapter)
```
engine.on('signal')     → INSERT signals table + broadcast WS + notify if tradable
engine.on('trade_entry') → INSERT trades table + broadcast WS + notify
engine.on('trade_exit')  → UPDATE trades table + broadcast WS + notify
engine.on('session_halt') → UPDATE session_states + broadcast WS + notify
```

### Data Flow
```
Binance WS → BinanceProvider → CandleAggregator → engine.pushLTFCandle()
                                                 → engine.pushHTFCandle()
engine.analyze() → SignalResult → EngineBridge → PostgreSQL
                                              → WebSocket broadcast
                                              → Telegram/Discord/Email
                                              → Dashboard auto-updates
```

## Verification Plan

1. `cd saas && npm run build` — Zero TS errors
2. `docker-compose up` — Postgres starts, migrations run
3. `cd saas && npm test` — Auth, engine pool, trades, WS tests pass
4. Manual flow: register → login → configure instrument → start engine → receive signal via WS
5. Stripe test mode: checkout → webhook → tier upgrade verified
6. `cd web && npm run build` — Next.js builds without errors
7. Full E2E: Dashboard shows real-time signals from Binance feed
