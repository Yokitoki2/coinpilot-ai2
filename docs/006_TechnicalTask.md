# 006 — Technical Task & Development Roadmap — v2 (Revised)

## 0. Why This Revision

Adds a dedicated **Backtesting & Validation Module** (Module E), required before any directional Consensus Score or trade-setup feature can ship (see `002_MVP_v2.md`, Section 4 — Validation Gate). Structure is modeled on freqtrade's backtesting engine, a mature open-source reference implementation for this exact problem.

---

## 1. Core Modules Specification

### Module A: Data Ingestion & Normalization

- Integration with Binance API (Kline, OrderBook, Ticker Data).
- Integration with Coinglass API (Open Interest, Funding Rate, Liquidation Heatmap).
- Normalization pipeline converting raw websocket/REST data into structured float vectors.
- **New requirement:** persist raw historical OHLCV + derivatives data to disk/object storage (not just live cache), separate from the production hot path. This is a prerequisite for Module E — you cannot backtest what you never stored.

### Module B: Consensus Intelligence Engine (CIE)

- Calculation of sub-scores (Trend, Order Flow, Derivatives, Liquidations, Macro).
- Scoring algorithm outputting composite score $S \in [0, 100]$.
- Dynamic Trade Setup calculator utilizing ATR formula.
- **Gate:** this module's output must not be exposed to end users with a directional call (LONG/SHORT) until Module E has produced a passing validation report.

### Module C: LLM Explanation Layer

- Async prompt builder injecting JSON metrics into system prompt templates.
- Integration with OpenAI GPT-4o / Anthropic Claude API.
- Fallback caching mechanism utilizing Redis for identical queries within 5-minute windows.

### Module D: Telegram Mini App (TMA) Frontend

- React-based WebApp UI optimized for Telegram internal browser.
- Real-time display of Consensus Cards with visual gauge meter.
- Portfolio management view with basic manual entry forms.

### Module E: Backtesting & Validation (New)

**Purpose:** Validate that Module B's factor weights and directional thresholds have genuine predictive value before they reach users, and keep re-validating over time as market regimes shift.

**Architecture (modeled on freqtrade):**

- Runs offline, against stored historical data from Module A — never against the live production path.
- Each run is a standalone, timestamped, reproducible artifact (config + strategy version + date range → result file), so past runs can be re-opened and compared, not just viewed once and discarded.
- Simulates trades exactly as the live Consensus Engine would have called them, including exchange fee assumptions — no silent "frictionless" assumptions that flatter the numbers.

**Required output — Summary Metrics Table:**

| Metric | Purpose |
|---|---|
| Backtest period (from/to) + total signals generated | Sample size transparency |
| Win rate (% direction calls correct) | Core accuracy metric |
| Avg. profit on winners / avg. loss on losers | Needed to compute expected return — win rate alone is misleading |
| **Expected Return** = WinRate × AvgWin + (1-WinRate) × AvgLoss | The single number that answers "does this actually pay for its own mistakes" |
| Max Drawdown (value + start/end dates) | Without this, a profit % is meaningless |
| Market change (buy-and-hold benchmark, same period) | Answers "did this even beat doing nothing" |
| Best day / worst day | Tail risk exposure |
| Breakdown by asset (BTC/ETH/SOL) and by market regime (trending vs. ranging, if feasible) | Weights may work in one regime and fail in another |

**Mandatory caveats to surface alongside any published metric (not just in internal docs — in the user-facing disclosure if the score ever ships):**

- Backtests assume historical conditions recur similarly; live results will differ, usually for the worse, since real trades cluster and correlate in time rather than behaving independently.
- If multiple weight configurations were tried before landing on the shipped one, that search itself inflates the apparent quality of the best result — a good-looking backtest is necessary but not sufficient evidence of a real edge.

**Re-validation cadence:** re-run on a rolling basis (e.g., monthly) as new data accumulates, and treat a material drop in live performance vs. backtested performance as a signal to pull the directional feature back to "context only" mode, not to keep shipping on stale validation.

---

## 2. Technical Stack

- **Language:** Python 3.11+
- **Framework:** FastAPI (Async processing)
- **Database:** PostgreSQL (Storage) + Redis (In-memory caching)
- **Historical data storage (New):** Flat-file or object storage (e.g., Parquet/S3-compatible) for OHLCV + derivatives history, separate from the operational Postgres instance, to keep backtesting I/O off the production database.
- **Deployment:** Docker, Docker Compose, Nginx

---

## 3. Immediate Action Items

1. Initialize FastAPI backend structure (`/app/api`, `/app/core`, `/app/services`).
2. Implement Binance REST/WebSocket connectors for market data ingestion, with historical persistence from day one (not bolted on later).
3. Build PostgreSQL ORM models matching `005_DatabaseSchema.md`.
4. **New:** Stand up Module E as a standalone offline job before Module B's directional output is exposed anywhere in the frontend — sequencing matters, this is a gate, not a parallel-track nice-to-have.
