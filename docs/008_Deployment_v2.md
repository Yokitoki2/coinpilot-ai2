# 008 — Deployment Specification — v2 (Revised)

## 0. Why This Revision

The v1 spec listed core services and a single deploy command but had no environment variable reference, no mention of the historical-data volume required by Module A/E, and no separation between the always-on web service and the offline backtesting job. This revision fills those gaps.

---

## 1. Core Infrastructure Services (Unchanged)

- FastAPI (App Server)
- PostgreSQL (Primary Database)
- Redis (In-Memory Response Cache)
- Nginx (Reverse Proxy & SSL)

## 2. New: Historical Data Volume

A separate persistent volume/object-storage mount is required for OHLCV + derivatives history used by Module E backtesting (`006_TechnicalTask_v2.md`). This must be distinct from the Postgres data volume so bulk backtest reads/writes never contend with production database I/O.

```yaml
# docker-compose.yml excerpt
volumes:
  postgres_data:
  historical_data:   # new — mounted read/write by ingestion service, read-only by backtest job
```

## 3. Environment Variables (New — Previously Undocumented)

```env
# Database
DATABASE_URL=postgresql://user:pass@postgres:5432/coinpilot

# Cache
REDIS_URL=redis://redis:6379/0

# Exchange & Data APIs
BINANCE_API_KEY=
BINANCE_API_SECRET=
COINGLASS_API_KEY=

# LLM Providers
OPENAI_API_KEY=
ANTHROPIC_API_KEY=

# Telegram
TELEGRAM_BOT_TOKEN=
TELEGRAM_WEBAPP_URL=

# Feature Gating
DIRECTIONAL_MODE_ENABLED=false   # global kill-switch, overrides per-symbol backtest flags if set false

# Historical Data Storage
HISTORICAL_DATA_PATH=/data/historical
```

`DIRECTIONAL_MODE_ENABLED` is a deliberate global override: even if `backtest_results.passed_validation` is true for a symbol, this flag lets ops disable directional output instantly across the whole app without a code deploy — useful if live performance diverges from backtest (see `003_ConsensusEngine_v2.md` Section 4).

Secrets should be injected via the deployment environment (e.g., Docker secrets or a secrets manager), not committed to `.env` files in version control.

## 4. Deployment Command Sequence (Unchanged)

```bash
git clone https://github.com/Yokitoki2/coinpilot-ai2.git
cd coinpilot-ai2
docker-compose up -d --build
```

## 5. New: Backtesting Job (Module E) — Runs Separately From the Web Service

The backtest is an offline batch job, not a request handled by the always-on API container — running it inline would block production traffic and mixes an I/O-heavy bulk workload with the low-latency request path.

```bash
# Run manually or on a scheduled basis (e.g., monthly cron)
docker-compose run --rm backtest-runner \
  --strategy-version weights-v1 \
  --symbols BTCUSDT,ETHUSDT,SOLUSDT \
  --period 2025-01-01:2025-12-31
```

Results are written to `backtest_results` (see `005_DatabaseSchema_v2.md`) and are what the API layer reads to decide `directional_enabled`.

## 6. Monitoring (New — Minimal Baseline)

Not fully specified here, but required before public launch:

- Health check endpoint (`/health`) for Nginx/orchestrator liveness probes.
- Alerting on exchange API rate-limit responses (Binance/Coinglass 429s) — Module A should degrade gracefully to cached data rather than fail requests outright.
- Basic LLM cost tracking per day (token usage × API pricing), since this is a variable cost that scales with usage and was flagged as unaccounted-for in the original project review.
