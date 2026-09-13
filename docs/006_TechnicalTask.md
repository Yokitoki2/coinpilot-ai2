# 006 — Technical Task & Development Roadmap

## 1. Core Modules Specification

### Module A: Data Ingestion & Normalization
- Integration with Binance API (Kline, OrderBook, Ticker Data).
- Integration with Coinglass API (Open Interest, Funding Rate, Liquidation Heatmap).
- Normalization pipeline converting raw websocket/REST data into structured float vectors.

### Module B: Consensus Intelligence Engine (CIE)
- Calculation of sub-scores (Trend, Order Flow, Derivatives, Liquidations, Macro).
- Scoring algorithm outputting composite score $S \in [0, 100]$.
- Dynamic Trade Setup calculator utilizing ATR formula.

### Module C: LLM Explanation Layer
- Async prompt builder injecting JSON metrics into system prompt templates.
- Integration with OpenAI GPT-4o / Anthropic Claude API.
- Fallback caching mechanism utilizing Redis for identical queries within 5-minute windows.

### Module D: Telegram Mini App (TMA) Frontend
- React-based WebApp UI optimized for Telegram internal browser.
- Real-time display of Consensus Cards with visual gauge meter.
- Portfolio management view with basic manual entry forms.

---

## 2. Technical Stack
- **Language:** Python 3.11+
- **Framework:** FastAPI (Async processing)
- **Database:** PostgreSQL (Storage) + Redis (In-memory caching)
- **Deployment:** Docker, Docker Compose, Nginx

---

## 3. Immediate Action Items
1. Initialize FastAPI backend structure (`/app/api`, `/app/core`, `/app/services`).
2. Implement Binance REST/WebSocket connectors for market data ingestion.
3. Build PostgreSQL ORM models matching `005_DatabaseSchema.md`.
