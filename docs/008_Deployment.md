# 008 — Deployment Specification

## Overview
Production deployment framework for CoinPilot AI backend services using Docker Compose and Nginx.

## 1. Core Infrastructure Services
- FastAPI (App Server)
- PostgreSQL (Primary Database)
- Redis (In-Memory Response Cache)
- Nginx (Reverse Proxy & SSL)

## 2. Deployment Command Sequence
git clone https://github.com/Yokitoki2/coinpilot-ai2.git
cd coinpilot-ai2
docker-compose up -d --build
