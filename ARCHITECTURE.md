# Architecture

GLAZ Markets (MarketSignal) is a pnpm monorepo I designed and built solo, delivering AI-processed market feed items to users in real time. This document covers the runtime components and how they connect.

## High-level overview

```
   << CONTENT SOURCES >>          << MARKET DATA VENDORS >>      << ANTHROPIC >>
   EDGAR · BusinessWire              Polygon · Yahoo                Claude
   PRNewswire RSS                    CoinGecko                      Sonnet
         │                                 │                          ▲
         │ poll (HTTPS/RSS)                │ HTTPS                     │ HTTPS
         ▼                                 ▼                           │
  ┌──────────────┐                  ┌──────────────┐                   │
  │ Press        │                  │ market-data  │                   │
  │ Release      │                  │ adapters     │                   │
  │ Poller       │                  │              │                   │
  └──────┬───────┘                  └──────┬───────┘                   │
         │                                 │ imported by               │
         │ enqueue       Commentary        │ (read-through,            │
         │               Engine            │  no queue)                │
         │                     │           │                           │
         │ enqueue ai_processor│           │                           │
         ▼                     ▼           ▼                           │
  ══════════════════════════════════════════════════════════════╗      │
  ║                          Redis                              ║      │
  ║     BullMQ queues  ·  pub/sub channels  ·  cache + dedup    ║      │
  ╚═════════════════════════════════════════════════════════════╝      │
         │                                 ▲                           │
         │ BRPOP ai_processor              │ PUBLISH feed:*            │
         ▼                                 │ + SETEX                   │
  ┌──────────────────────────────────────────────────────────┐         │
  │                  AI Processor Worker                     │─────────┘
  │          pulls job → Claude → Zod → persist              │ Anthropic SDK
  └──────────────────────────┬───────────────────────────────┘
                             │ INSERT FeedItem
                             ▼
  ╔══════════════════════════════════════════════════════════════╗
  ║              Postgres — source of truth, via Prisma          ║
  ║  User · FeedItem · FeedItemTicker · Portfolio · Commentary…  ║
  ╚══════════════════════════════════════════════════════════════╝
         ▲                                 ▲
         │ Prisma reads                    │ (no direct access;
         │ (SSR + API routes)              │  WS server reads only
         │                                 │  from Redis pub/sub)
  ┌──────┴─────────┐                  ┌────┴─────────────┐
  │ Web            │                  │ WS Server        │
  │ Next.js :3000  │                  │ Node :4000       │
  └──────┬─────────┘                  └────┬─────────────┘
         │ HTML + JWT                      │ WebSocket
         │                                 │ (pre-serialized fanout)
         ▼                                 ▼
              ┌────────────────────────────────┐
              │        End user — browser      │
              └────────────────────────────────┘
```

**Read top-to-bottom:**

1. **Ingress lane.** The press-release poller and commentary engine reach out to external content sources and push jobs onto Redis (BullMQ). The market-data adapters are a sibling lane: they don't enqueue anything — they're a library imported by other apps that makes on-demand HTTPS calls to Polygon/Yahoo/CoinGecko.
2. **Enrichment lane.** The AI processor worker is the only thing that pulls `ai_processor` jobs. It calls Claude, writes the finished `FeedItem` to Postgres, caches it in Redis, and publishes it on Redis pub/sub (`feed:all` + per-ticker channels).
3. **Delivery lane.** The Next.js web app reads Postgres directly via Prisma for server-rendered pages. The WS server never touches Postgres — it only subscribes to Redis pub/sub and fans pre-serialized payloads out to browser clients over WebSocket. The browser holds both connections (HTTP to Next.js, WS to ws-server) using the same signed JWT.

**Redis is the seam between ingress and enrichment. Postgres + Redis pub/sub together are the seam between enrichment and delivery.** Nothing talks worker-to-worker or worker-to-ws-server directly.

## Apps

| App | Runtime | Purpose |
|---|---|---|
| `apps/web` | Next.js 14 | React UI (feed, portfolios, calendar, hedge funds, search), auth, Prisma reads for pages |
| `apps/ws-server` | Node/tsx | WebSocket hub. Verifies JWT on upgrade, subscribes to Redis channels, fans out pre-serialized payloads to clients (per-ticker refcounted) |
| `apps/workers/ai-processor` | Node/tsx | BullMQ consumer for the `ai_processor` queue. Calls Claude to extract headline / bullets / metrics / sentiment, then publishes the finished `FeedItem` |
| `apps/workers/press-release` | Node/tsx | Polls EDGAR 8-K + BusinessWire + PRNewswire RSS (~45s). Dedupes via sha256 (Redis set + Postgres unique index), enqueues to `press_release` queue |
| `apps/workers/commentary-engine` | Node/tsx | ET-hours scheduler. Produces templated market commentary, enqueues to `commentary` queue, tracks idempotency |

## Packages

| Package | Provides |
|---|---|
| `@marketsignal/db` | Prisma schema + singleton client (`User`, `FeedItem`, `FeedItemTicker`, `Portfolio`, `CommentaryRun`, etc.) |
| `@marketsignal/types` | Shared TS types + Zod schemas — `FeedItem`, WebSocket message types, API DTOs |
| `@marketsignal/config` | Env loader, queue names, pub/sub channel names, ET calendar helpers |
| `@marketsignal/market-data` | Provider adapters for Polygon / Yahoo / CoinGecko, with deterministic mock fallbacks |

## Infrastructure

- **Postgres 16** — source of truth for users, feed items, portfolios, commentary idempotency. Every service reads/writes through a single shared Prisma client — no service opens its own DB connection.
- **Redis 7** — three roles: BullMQ job queues (`press_release`, `commentary`, `ai_processor`), pub/sub fanout (`feed:all` + per-symbol channels), and a cache + dedup layer.
- **WebSocket server** — one Redis subscriber per process, holding a refcounted map of `ticker → connected clients` so it only subscribes to per-ticker channels while at least one client is watching that symbol. Payloads are serialized once and written to every matching socket.

## Core data flow: a feed item's lifecycle

1. **Ingest** — poller fetches a source, computes a sha256 content hash, checks the dedup set/index to avoid reprocessing.
2. **Enqueue** — a new item is pushed onto the `ai_processor` queue with raw text + known tickers.
3. **AI processing** — the worker pulls the job, calls Claude, validates the response against a schema (repairing on drift).
4. **Persist** — the finished item is written to Postgres.
5. **Cache + publish** — the item is cached and published to Redis pub/sub channels.
6. **Fan-out** — the WS server relays the message to every connected client subscribed to that channel.
7. **Render** — the browser prepends and animates the new item into the live feed.
