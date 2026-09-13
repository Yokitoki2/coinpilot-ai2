# 007 — API Routes Specification (FastAPI)

## Overview
RESTful API specification for CoinPilot AI backend services, handling Telegram WebApp authentication, market analysis queries, user energy balance management, and portfolio tracking.

---

## 1. Authentication
All endpoints require Telegram WebApp initData validation via HMAC-SHA256 signature passed in the headers.

```http
Authorization: Bearer <telegram_init_data_string>
