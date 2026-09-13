# 007 — API Routes Specification (FastAPI) — v2 (Revised)

## 0. Why This Revision

v1 only specified the directional Market Card endpoint and basic portfolio CRUD. It had no routes for Portfolio Risk Intelligence (the new primary feature, `002_MVP_v2.md` Feature A) or Smart Alerts (Feature C), and the Market Card response always returned a direction + trade setup with no gating. This revision adds the missing routes and applies the mode gate from `003_ConsensusEngine_v2.md`.

---

## 1. Authentication (Unchanged)

All endpoints require Telegram WebApp initData validation via HMAC-SHA256 signature.

```
Authorization: Bearer <telegram_init_data_string>
```

---

## 2. Market & AI Analysis Endpoints

### GET /api/v1/analysis/{symbol}

Fetch market context (and, once validated, directional signal) for a given asset.

Parameters: `symbol` (path), `timeframe` (query, default `5m`)
Energy Cost: 10 Units

Response (200 OK) — **Context Mode (default, current behavior)**:

```json
{
  "symbol": "BTCUSDT",
  "mode": "context",
  "directional_enabled": false,
  "factors": {
    "trend": "bullish_continuation",
    "order_flow": "positive_cvd",
    "derivatives": "oi_rising_funding_elevated",
    "liquidations": "short_cluster_below_price",
    "sentiment": "neutral"
  },
  "explanation": "Funding rates are elevated and open interest is rising, with a cluster of short liquidations sitting below current price.",
  "cached": true,
  "timestamp": "2026-09-13T09:30:00Z"
}
```

Response (200 OK) — **Directional Mode, only returned once `backtest_results.passed_validation = true` for this symbol/strategy version**:

```json
{
  "symbol": "BTCUSDT",
  "mode": "directional",
  "directional_enabled": true,
  "consensus_score": 78,
  "direction": "LONG",
  "atr_setup": {
    "entry": 92100.00,
    "stop_loss": 91250.00,
    "take_profit_1": 93375.00,
    "take_profit_2": 95600.00
  },
  "track_record": {
    "win_rate": 61.4,
    "sample_size": 214,
    "backtest_period": "2025-01-01/2025-12-31"
  },
  "explanation": "Strong bullish continuation signal driven by $45M in short liquidations paired with positive CVD order flow delta. Backtested win rate 61.4% over 214 signals.",
  "disclaimer": "This is not financial advice. Past backtest performance does not guarantee future results.",
  "cached": true,
  "timestamp": "2026-09-13T09:30:00Z"
}
```

`directional_enabled` is set server-side from `backtest_results`, not client-controlled — a client cannot request directional mode for a symbol that hasn't passed validation.

---

## 3. Portfolio Risk Intelligence Endpoints (New — Feature A)

### GET /api/v1/portfolio/risk-analysis

Compute and return AI-explained risk metrics for the user's full portfolio.
Energy Cost: 10 Units

Response (200 OK):

```json
{
  "total_value_usdt": 18420.50,
  "concentration": {
    "top_holding_symbol": "BTCUSDT",
    "top_holding_pct": 54.2,
    "top_3_pct": 81.7
  },
  "net_exposure": {
    "net_long_pct": 72.0,
    "net_short_pct": 0.0,
    "spot_pct": 28.0
  },
  "correlation_warning": "68% of your holdings (BTC, ETH) tend to move together.",
  "stress_test": {
    "scenario": "BTC -10%",
    "portfolio_impact_pct": -6.8
  },
  "explanation": "Your portfolio is heavily concentrated in BTC (54%) with an additional correlated ETH position, giving you limited diversification. A 10% BTC decline would reduce total portfolio value by approximately 6.8%.",
  "disclaimer": "This is a risk description, not investment advice.",
  "generated_at": "2026-09-13T09:30:00Z"
}
```

---

## 4. Smart Alerts Endpoints (New — Feature C)

### POST /api/v1/alerts

Create a new alert condition.

Request Body:

```json
{
  "symbol": "BTCUSDT",
  "condition_type": "funding_spike",
  "threshold_value": 0.05
}
```

### GET /api/v1/alerts

List the user's active and inactive alerts.

### DELETE /api/v1/alerts/{alert_id}

Deactivate/remove an alert.

---

## 5. User & Energy Management Endpoints (Unchanged, referral fields clarified)

### GET /api/v1/user/profile

```json
{
  "telegram_id": 123456789,
  "username": "trader_joe",
  "energy_balance": 90,
  "is_premium": false,
  "referral_code": "https://t.me/CoinPilotBot?start=123456789",
  "total_referrals": 3,
  "qualified_referrals": 1
}
```

`total_referrals` counts everyone who joined via the link; `qualified_referrals` counts only those who crossed the anti-abuse activity threshold in `referral_qualifications` and therefore earned the referrer a bonus — see `005_DatabaseSchema_v2.md`.

### POST /api/v1/user/energy/claim-daily

Unchanged from v1.

---

## 6. Portfolio Endpoints (Unchanged, `side` field now required)

### GET /api/v1/portfolio

Get all user holdings and total valuation.

### POST /api/v1/portfolio/add

Request Body:

```json
{
  "coin_symbol": "SOLUSDT",
  "side": "spot",
  "amount": 15.5,
  "buy_price": 135.20
}
```

`side` must be one of `spot`, `futures_long`, `futures_short` — required for the net exposure calculation in Section 3.
