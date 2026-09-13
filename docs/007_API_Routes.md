# 007 — API Routes Specification (FastAPI)

## Overview
RESTful API specification for CoinPilot AI backend services.

## 1. Authentication
Authorization: Bearer <telegram_init_data_string>

## 2. Market & AI Analysis Endpoints
GET /api/v1/analysis/{symbol}

## 3. User & Energy Management Endpoints
GET /api/v1/user/profile
POST /api/v1/user/energy/claim-daily

## 4. Portfolio Endpoints
GET /api/v1/portfolio
POST /api/v1/portfolio/add
