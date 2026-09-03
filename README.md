<img width="2048" height="1089" alt="image" src="https://github.com/user-attachments/assets/84923aec-b6b4-4dfe-8080-3f6252f4276c" />
<img width="2048" height="1089" alt="image" src="https://github.com/user-attachments/assets/ad6874ea-5afb-4a3a-8284-063fa2feec22" />
# GlazMarkets
Built and shipped a real-time financial intelligence platform solo — delivering market-moving events instantly vs. the up-to-10-minute lag on incumbents like Benzinga and Briefing.com.

---

## Overview

GLAZ Markets (product name: MarketSignal) aggregated 100+ real-time financial data sources — SEC filings, earnings releases, analyst ratings, institutional ownership, and macroeconomic events — and used an AI pipeline to classify, summarize, and prioritize them for delivery.

I designed, built, and ran the entire system solo: architecture, backend, data pipeline, and infrastructure. Recruited beta users to validate the product. Ran it at my own expense for several months, then made the call to conclude the project after determining it didn't have a path to sustainable unit economics as a solo venture.

## Screenshots
<img width="942" height="2046" alt="image" src="https://github.com/user-attachments/assets/9be9fe0a-deaf-4b1d-af16-671f6bc3b926" />
<img width="2048" height="1087" alt="image" src="https://github.com/user-attachments/assets/49232302-3d4a-4e76-af08-d7f6d5958d5a" />
<img width="942" height="2046" alt="image" src="https://github.com/user-attachments/assets/b4135bdc-a596-44e6-ab36-23e60a982315" />
<img width="2048" height="1089" alt="image" src="https://github.com/user-attachments/assets/a7acdd99-0d3a-4b02-a3b5-f21a080ac52d" />


## Technical Highlights

- **Real-time ingestion at scale** — aggregated 100+ live sources including EDGAR filings, press releases, and market data feeds, processing thousands of items a day and pushing market-moving events to users in under 15 seconds
- **Coverage benchmarking** — built a benchmark against a ground-truth market feed using semantic matching and residual analysis; used categorized root-cause diagnosis (thin filing content, filter exclusions, classifier bugs) to prioritize fixes, lifting coverage from ~60% to ~89% of market-moving events
- **Fixed a filter-layer regression** in EDGAR 424B5/424B4 offering filing classification that was causing misclassified content
- **Reverse-engineered a data access block** — unlocked full BusinessWire article bodies (previously assumed blocked by Akamai) via a Chrome mobile user-agent switch, adding meaningful real coverage volume
- **Cut AI processing cost significantly** with prompt caching and a template-render bypass for Form 4 filings — skipping unnecessary LLM enrichment calls with no loss in output quality
- **Structured event taxonomy** — materiality filters surfaced analyst ratings (upgrades/downgrades, price targets), earnings, 13F institutional holdings, and macro releases (FOMC, CPI, GDP) as screenable, structured data

## Example Coverage

Caught and summarized USA Rare Earth (USAR)'s $1.6B U.S. Department of Commerce backing — $277M in federal funding plus $1.3B in CHIPS Act loan capacity — within seconds of it hitting.

## Stack

TypeScript monorepo · Next.js 14 · React · Tailwind · Node.js · PostgreSQL · Prisma · Redis · BullMQ · Claude API (Sonnet for enrichment, Haiku as fallback) · deployed across Railway, Vercel, and Cloudflare

~480 commits, 55 releases, built and run hands-off.

## Status

Concluded August 2026. Solo-founded, self-funded, and validated with beta users before winding down.
