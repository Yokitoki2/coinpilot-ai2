# 005 — Database Schema Specification (PostgreSQL)

## Overview
Relational schema for CoinPilot AI, engineered for scalable user tracking, precise financial balances (`DECIMAL`), execution history, and Redis/PostgreSQL hybrid caching.

---

## 1. DDL SQL Schema

```sql
-- Create Users Table
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

-- Create Portfolios Table (Manual holdings)
CREATE TABLE portfolios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    telegram_id BIGINT NOT NULL REFERENCES users(telegram_id) ON DELETE CASCADE,
    coin_symbol VARCHAR(20) NOT NULL,
    amount NUMERIC(38, 18) NOT NULL CHECK (amount > 0),
    buy_price NUMERIC(38, 18) NOT NULL CHECK (buy_price > 0),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create AI Analysis History Table (Saved cards per user)
CREATE TABLE analysis_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    telegram_id BIGINT NOT NULL REFERENCES users(telegram_id) ON DELETE CASCADE,
    symbol VARCHAR(20) NOT NULL,
    consensus_score INT NOT NULL,
    direction VARCHAR(10) NOT NULL,
    analysis_json JSONB NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create Energy Transactions Log Table
CREATE TABLE energy_transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    telegram_id BIGINT NOT NULL REFERENCES users(telegram_id) ON DELETE CASCADE,
    amount INT NOT NULL, -- Positive for rewards, Negative for usage
    reason VARCHAR(50) NOT NULL, -- 'daily_reset', 'referral_bonus', 'ai_generation', 'chat_query'
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create AI Response Cache Table (Persistent fall-back for Redis)
CREATE TABLE ai_cache (
    cache_key VARCHAR(50) PRIMARY KEY, -- e.g., 'BTCUSDT_5m'
    analysis_json JSONB NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes for Query Optimization
CREATE INDEX idx_portfolios_telegram_id ON portfolios(telegram_id);
CREATE INDEX idx_analysis_history_telegram_id ON analysis_history(telegram_id);
CREATE INDEX idx_energy_transactions_telegram_id ON energy_transactions(telegram_id);
```
## 2. Table Relationships Summary

- users.telegram_id -> portfolios.telegram_id (1 : Many)
- users.telegram_id -> analysis_history.telegram_id (1 : Many)
- users.telegram_id -> energy_transactions.telegram_id (1 : Many)
- users.telegram_id -> users.referrer_id (Self-referencing Foreign Key for invites)
