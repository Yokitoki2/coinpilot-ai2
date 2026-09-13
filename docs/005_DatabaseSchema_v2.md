# 005 — Database Schema Specification (PostgreSQL) — v2 (Revised)

## 0. Why This Revision

The v1 schema only supported the directional Market Card feature. It had no tables for: portfolio side (spot/futures, needed for net exposure calculation in Feature A), smart alerts (Feature C), backtest results (Module E), or referral anti-abuse tracking (flagged as a gap in the original project review). This revision adds all four.

---

## 1. DDL SQL Schema

```sql
-- Create Users Table (unchanged from v1, comments added)
CREATE TABLE users (
    telegram_id BIGINT PRIMARY KEY,
    username VARCHAR(64),
    first_name VARCHAR(64),
    energy_balance INT DEFAULT 100 CHECK (energy_balance >= 0),
    is_premium BOOLEAN DEFAULT FALSE,
    referrer_id BIGINT REFERENCES users(telegram_id) ON DELETE SET NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Portfolios Table — NEW: side column, required for net directional exposure (Feature A)
CREATE TABLE portfolios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    telegram_id BIGINT NOT NULL REFERENCES users(telegram_id) ON DELETE CASCADE,
    coin_symbol VARCHAR(20) NOT NULL,
    side VARCHAR(10) NOT NULL DEFAULT 'spot' CHECK (side IN ('spot', 'futures_long', 'futures_short')),
    amount NUMERIC(38, 18) NOT NULL CHECK (amount > 0),
    buy_price NUMERIC(38, 18) NOT NULL CHECK (buy_price > 0),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Analysis History Table (unchanged) — now also stores mode ('context'|'directional')
CREATE TABLE analysis_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    telegram_id BIGINT NOT NULL REFERENCES users(telegram_id) ON DELETE CASCADE,
    symbol VARCHAR(20) NOT NULL,
    mode VARCHAR(12) NOT NULL DEFAULT 'context' CHECK (mode IN ('context', 'directional')),
    consensus_score INT,
    direction VARCHAR(10),
    analysis_json JSONB NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Energy Transactions Log Table (unchanged)
CREATE TABLE energy_transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    telegram_id BIGINT NOT NULL REFERENCES users(telegram_id) ON DELETE CASCADE,
    amount INT NOT NULL,
    reason VARCHAR(50) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- AI Response Cache Table (unchanged)
CREATE TABLE ai_cache (
    cache_key VARCHAR(50) PRIMARY KEY,
    analysis_json JSONB NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- NEW: Smart Alerts Table (Feature C)
CREATE TABLE alerts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    telegram_id BIGINT NOT NULL REFERENCES users(telegram_id) ON DELETE CASCADE,
    symbol VARCHAR(20) NOT NULL,
    condition_type VARCHAR(30) NOT NULL, -- 'rsi_divergence', 'funding_spike', 'oi_surge', 'liquidation_proximity'
    threshold_value NUMERIC(20, 8) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    last_triggered_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- NEW: Backtest Results Table (Module E — required before directional_enabled can be true)
CREATE TABLE backtest_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    run_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    strategy_version VARCHAR(20) NOT NULL, -- e.g. 'weights-v1'
    symbol VARCHAR(20) NOT NULL,
    period_start DATE NOT NULL,
    period_end DATE NOT NULL,
    total_signals INT NOT NULL,
    win_rate NUMERIC(5, 2) NOT NULL,
    avg_win_pct NUMERIC(6, 2) NOT NULL,
    avg_loss_pct NUMERIC(6, 2) NOT NULL,
    expected_return_pct NUMERIC(6, 2) NOT NULL,
    max_drawdown_pct NUMERIC(6, 2) NOT NULL,
    market_change_pct NUMERIC(6, 2) NOT NULL, -- buy-and-hold benchmark, same period
    passed_validation BOOLEAN NOT NULL DEFAULT FALSE,
    report_json JSONB NOT NULL
);

-- NEW: Referral Qualification Tracking (anti-abuse for Feature D)
-- Bonus energy is credited only once a referred user crosses this qualifying-activity threshold.
CREATE TABLE referral_qualifications (
    referred_telegram_id BIGINT PRIMARY KEY REFERENCES users(telegram_id) ON DELETE CASCADE,
    referrer_telegram_id BIGINT NOT NULL REFERENCES users(telegram_id) ON DELETE CASCADE,
    qualifying_actions_count INT NOT NULL DEFAULT 0,
    qualified_at TIMESTAMP WITH TIME ZONE, -- NULL until threshold met and bonus credited
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes for Query Optimization
CREATE INDEX idx_portfolios_telegram_id ON portfolios(telegram_id);
CREATE INDEX idx_analysis_history_telegram_id ON analysis_history(telegram_id);
CREATE INDEX idx_energy_transactions_telegram_id ON energy_transactions(telegram_id);
CREATE INDEX idx_alerts_telegram_id_active ON alerts(telegram_id) WHERE is_active = TRUE;
CREATE INDEX idx_backtest_results_strategy_symbol ON backtest_results(strategy_version, symbol);
```

---

## 2. Table Relationships Summary

- `users.telegram_id` → `portfolios.telegram_id` (1 : Many)
- `users.telegram_id` → `analysis_history.telegram_id` (1 : Many)
- `users.telegram_id` → `energy_transactions.telegram_id` (1 : Many)
- `users.telegram_id` → `alerts.telegram_id` (1 : Many) — **new**
- `users.telegram_id` → `users.referrer_id` (self-referencing FK for invites)
- `users.telegram_id` → `referral_qualifications.referred_telegram_id` / `.referrer_telegram_id` (**new**, decouples "invited" from "bonus-eligible")
- `backtest_results` has no FK to `users` — it's an operational/internal table, not user-facing data, but its `passed_validation` flag is what the API layer checks before setting `directional_enabled = true` (see `007_API_Routes_v2.md`).

## 3. Data Retention Note

Raw historical OHLCV/derivatives data referenced in `006_TechnicalTask_v2.md` Module A is intentionally **not** stored in PostgreSQL — it belongs in flat-file/object storage to keep bulk backtesting I/O off the operational database. `backtest_results` stores only the summary output of a run, not the underlying tick data.
