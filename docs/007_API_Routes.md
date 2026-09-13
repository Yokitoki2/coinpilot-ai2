# 007 — API Routes Specification (FastAPI)
## Overview
RESTful API specification for CoinPilot AI backend services, handling Telegram WebApp authentication, market analysis queries, user energy balance management, and portfolio tracking.

## 1. Authentication
All endpoints require Telegram WebApp initData validation via HMAC-SHA256 signature passed in the headers.

```http
Authorization: Bearer <telegram_init_data_string>
```
## 2. Market & AI Analysis Endpoints
### GET /api/v1/analysis/{symbol}
Fetch real-time Consensus Score and AI explanation for a given crypto asset.

Parameters:

symbol (path, string): Trading pair symbol (e.g. BTCUSDT)

timeframe (query, string, default: 5m): Chart timeframe (1m, 5m, 15m, 1h)

Energy Cost: 10 Units

Response (200 OK):

```json
{
  "symbol": "BTCUSDT",
  "consensus_score": 78,
  "direction": "LONG",
  "atr_setup": {
    "entry": 92100.00,
    "stop_loss": 91250.00,
    "take_profit_1": 93375.00,
    "take_profit_2": 95600.00
  },
  "explanation": "Strong bullish continuation signal driven by $45M in short liquidations paired with positive CVD order flow delta.",
  "cached": true,
  "timestamp": "2026-09-13T09:30:00Z"
}
```
## 3. User & Energy Management Endpoints
### GET /api/v1/user/profile
Retrieve current user stats, premium status, and remaining energy balance.

Response (200 OK):

```json
{
  "telegram_id": 123456789,
  "username": "trader_joe",
  "energy_balance": 90,
  "is_premium": false,
  "referral_code": "https://t.me/CoinPilotBot?start=123456789",
  "total_referrals": 3
}
```
POST /api/v1/user/energy/claim-daily
Claim daily free energy refill (100 units max).

Response (200 OK):

```json
{
  "status": "success",
  "new_balance": 100,
  "claimed_amount": 10
}
```
## 4. Portfolio Endpoints
### GET /api/v1/portfolio
Get all user holdings and total portfolio valuation.

### POST /api/v1/portfolio/add
Add a new manual holding record.

Request Body:

```json
{
  "coin_symbol": "SOLUSDT",
  "amount": 15.5,
  "buy_price": 135.20
}
```
