# CoinPilot AI — Technical Documentation Suite

Welcome to the technical documentation suite for **CoinPilot AI** — a crypto portfolio intelligence Telegram Mini App, combining portfolio risk analysis with market context summaries.

## Revision Note (v2)

This documentation set was revised to shift the product's primary value from an unvalidated directional trading signal to portfolio risk intelligence, with directional scoring retained but gated behind a documented backtesting process before it reaches users. See `002_MVP_v2.md` Section 0 for the full rationale. Files marked `_v2` supersede their original counterparts; where a file has no `_v2` version, the original still applies as-is.

## Documentation Index

1. [Product Overview](docs/001_ProductOverview_v2.md) — Vision, value proposition, architecture, and positioning (v2: portfolio-risk-first).
2. [MVP Scope Specification](docs/002_MVP_v2.md) — Core functional scope, validation gate, and success metrics.
3. [Consensus/Market Intelligence Engine](docs/003_ConsensusEngine_v2.md) — Scoring algorithm, Context vs. Directional output modes.
4. [System Prompts Specification](docs/004_SystemPrompts_v2.md) — LLM prompts for Market Context, Portfolio Risk, and Smart Alerts.
5. [Database Schema Specification](docs/005_DatabaseSchema_v2.md) — PostgreSQL DDL, including alerts, backtest results, and referral anti-abuse tracking.
6. [Technical Task & Roadmap](docs/006_TechnicalTask_v2.md) — Module breakdown including the new Backtesting & Validation module.
7. [API Routes Specification](docs/007_API_Routes_v2.md) — REST routes, including Portfolio Risk and Smart Alerts endpoints, with directional-mode gating.
8. [Deployment Specification](docs/008_Deployment_v2.md) — Docker Compose, environment variables, historical data storage, and the offline backtest job.

## Reading Order for New Contributors

Start with `001` for the product vision, then `002` for scope and the validation-gate rationale — that context makes the gating logic in `003`, `004`, and `007` make sense rather than looking like arbitrary restrictions.
