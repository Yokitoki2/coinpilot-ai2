# 007 — API Routes Specification (FastAPI)

## Overview
RESTful API specification for CoinPilot AI backend services, handling Telegram WebApp authentication, market analysis queries, user energy balance management, and portfolio tracking.

---

## 1. Authentication
All endpoints require Telegram WebApp initData validation via HMAC-SHA256 signature passed in the headers.

```http
Authorization: Bearer <telegram_init_data_string>
2. Market & AI Analysis Endpoints
GET /api/v1/analysis/{symbol}
Fetch real-time Consensus Score and AI explanation for a given crypto asset.

Parameters:

symbol (path, string): Trading pair symbol (e.g., BTCUSDT, ETHUSDT).

timeframe (query, string, default: 5m): Chart timeframe (1m, 5m, 15m, 1h).

Energy Cost: 10 Units.

Response (200 OK):

JSON
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
  "explanation": "Strong bullish continuation signal driven by $45M in short liquidations above $92,000 paired with positive CVD order flow delta.",
  "cached": true,
  "timestamp": "2026-09-13T09:30:00Z"
}
3. User & Energy Management Endpoints
GET /api/v1/user/profile
Retrieve current user stats, premium status, and remaining energy.

Response (200 OK):

JSON
{
  "telegram_id": 123456789,
  "username": "trader_joe",
  "energy_balance": 90,
  "is_premium": false,
  "referral_code": "https://t.me/CoinPilotBot?start=123456789",
  "total_referrals": 3
}
POST /api/v1/user/energy/claim-daily
Claim daily free energy refill (100 units max).

Response (200 OK):

JSON
{
  "status": "success",
  "new_balance": 100,
  "claimed_amount": 10
}
4. Portfolio Endpoints
GET /api/v1/portfolio
Get all user holdings and total portfolio valuation.

POST /api/v1/portfolio/add
Add a new manual holding record.

Request Body:

JSON
{
  "coin_symbol": "SOLUSDT",
  "amount": 15.5,
  "buy_price": 135.20
}

4. Сохрани файл через **Commit changes...**

---

### Шаг 2. Финальный документ `008_Deployment.md`

После сохранения 7-го документа создаем заключительный файл:

1. В папке `docs` нажми **Add file** $\rightarrow$ **Create new file**.
2. Введи название: `008_Deployment.md`.
3. Скопируй и вставь текст ниже:

```markdown
# 008 — Deployment & Infrastructure Specification

## Overview
Production-ready deployment framework for CoinPilot AI utilizing Docker Compose, FastAPI, PostgreSQL, Redis, and Nginx acting as a reverse proxy with SSL termination.

---

## 1. Environment Architecture

```text
[ Telegram Mini App ] 
         │ (HTTPS / TLS 1.3)
         ▼
    [ Nginx ] ── (Static Content & SSL)
         │
         ▼
   [ FastAPI ] ── (Async Application Server)
     ├── [ Redis Cache ] (5-min market response cache)
     └── [ PostgreSQL ]  (User balances & trade logs)
2. Docker Compose Configuration (docker-compose.yml)
YAML
version: '3.8'

services:
  web:
    build: .
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
    volumes:
      - .:/app
    environment:
      - DATABASE_URL=postgresql://coinpilot:secret@db:5432/coinpilot_db
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    ports:
      - "8000:8000"

  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: coinpilot
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: coinpilot_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    restart: always
    ports:
      - "6379:6379"

volumes:
  postgres_data:
3. Deployment Command Sequence
Clone repository to VPS:
git clone [https://github.com/Yokitoki2/coinpilot-ai2.git](https://github.com/Yokitoki2/coinpilot-ai2.git)

Create environment variables configuration:
cp .env.example .env

Launch container infrastructure:
docker-compose up -d --build
