# GlazMarkets
Built and shipped a real-time financial intelligence platform solo, delivering market-moving events in seconds versus the up-to-10-minute lag on incumbents like Benzinga and Briefing.com.

---

## Overview

GLAZ Markets (product name: MarketSignal) aggregated 100+ real-time financial data sources, including SEC filings, earnings releases, analyst ratings, institutional ownership, and macroeconomic events, and used an AI pipeline to classify, summarize, and prioritize them for delivery.

I designed, built, and ran the entire system solo: architecture, backend, data pipeline, and infrastructure. I recruited beta users to validate the product, ran it at my own expense for several months, then made the call to conclude the project after determining it didn't have a path to sustainable unit economics as a solo venture.

## Product Surface

Beyond the real-time feed, GLAZ Markets included:
- **Auto-generated market summaries** — narrative after-hours/closing briefings synthesized from the day's feed activity, not just raw data
- **Ticker profile pages** — company fundamentals, sector/HQ/employee data, price history, analyst consensus
- **Fixed income desk** — live Treasury yield curve (3M–30Y) with day-over-day shape comparison, upcoming auction calendar
- **Portfolios & hedge fund tracking** — 13F institutional holdings, custom watchlists
- **Cross-asset dashboard** — equities, crypto (BTC and ETH), FX, and commodities in one live header


## Screenshots

<img width="2048" height="1087" alt="Live feed desktop view" src="https://github.com/user-attachments/assets/9d741f6f-c4c8-4dfd-b49d-8bf1805cb915" />

*Desktop — real-time feed with sentiment tagging, structured metrics, and portfolio alerts.*

<img width="2048" height="1089" alt="Market summaries desktop view" src="https://github.com/user-attachments/assets/84923aec-b6b4-4dfe-8080-3f6252f4276c" />

*Desktop — auto-generated market summary with cross-asset overnight data.*

<img width="942" height="2046" alt="Live feed mobile view" src="https://github.com/user-attachments/assets/9be9fe0a-deaf-4b1d-af16-671f6bc3b926" />

*Mobile — the Rocket Lab item from the example below, as delivered in the live feed.*

<img width="942" height="2046" alt="Hedge fund short interest mobile view" src="https://github.com/user-attachments/assets/b4135bdc-a596-44e6-ab36-23e60a982315" />

*Mobile — FINRA short interest rankings by ticker, updated semi-monthly.*

<img width="471" height="1024" alt="Ticker profile mobile view" src="https://github.com/user-attachments/assets/3e424db4-1136-43a0-baa8-d617c1dec2c1" />

*Mobile — company profile page with price history and fundamentals.*

<img width="471" height="1024" alt="Bonds desk mobile view" src="https://github.com/user-attachments/assets/4cbb5952-3591-4dff-bf7f-1d5c82e9f3c6" />

*Mobile — live Treasury yield curve and auction calendar.*

## Technical Highlights

- **Real-time ingestion at scale** — aggregated 100+ live sources including EDGAR filings, press releases, and market data feeds, processing thousands of items a day and pushing market-moving events to users in under 15 seconds
- **Coverage benchmarking** — built a benchmark against a ground-truth market feed using semantic matching and residual analysis; used categorized root-cause diagnosis (thin filing content, filter exclusions, classifier bugs) to prioritize fixes, lifting coverage from ~60% to ~89% of market-moving events
- **Fixed a filter-layer regression** in EDGAR 424B5/424B4 offering filing classification that was causing misclassified content
- **Reverse-engineered a data access block** — unlocked full BusinessWire article bodies (previously assumed blocked by Akamai) via a Chrome mobile user-agent switch, adding meaningful real coverage volume
- **Cut AI processing cost significantly** with prompt caching and a template-render bypass for Form 4 filings — skipping unnecessary LLM enrichment calls with no loss in output quality
- **Structured event taxonomy** — materiality filters surfaced analyst ratings (upgrades/downgrades, price targets), earnings, 13F institutional holdings, and macro releases (FOMC, CPI, GDP) as screenable, structured data

## Example: Raw Filing → Delivered Signal

**Source (May 21, 2026, Rocket Lab press release, ~600 words):**
> "Rocket Lab Corporation... today announced it has been awarded a $90 million contract by the U.S. Space Force's Space Systems Command (SSC) to design, manufacture, integrate, and operate two geostationary (GEO) satellites hosting the Heimdall space domain awareness (SDA) payload... Rocket Lab will serve as prime contractor and end-to-end mission provider, responsible for spacecraft design and manufacture, integration of the in-house Heimdall optical payload... [continues for 5 more paragraphs on the Lightning bus, GEOST acquisition history, and production facilities]"

**Delivered on GLAZ Markets, 15 minutes later:**

> **RKLB** · BULLISH · PRODUCT
> **Rocket Lab awarded $90 mln U.S. Space Force contract to build 2 GEO satellites hosting Heimdall space domain awareness payload**
> `satellites: 2` `contract_value: $90 mln`
> - Contract covers design, build, and operation of 2 GEO satellites hosting Heimdall space domain awareness payload
> - Awarded by U.S. Space Force; contract value is $90 mln

The pipeline read the full release, tagged the ticker, classified sentiment and category, extracted the two numbers that actually mattered (deal size, satellite count) out of six paragraphs of corporate background, and compressed it to two scannable bullets — all without a human in the loop.

## Stack

TypeScript monorepo · Next.js 14 · React · Tailwind · Node.js · PostgreSQL · Prisma · Redis · BullMQ · Claude API (Sonnet for enrichment, Haiku as fallback) · deployed across Railway, Vercel, and Cloudflare

~480 commits, 55 releases, built and run hands-off.

## Status

Concluded August 2026. Solo-founded, self-funded, and validated with beta users before winding down.
