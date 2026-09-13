# 001 — Product Overview & Project Vision — v2 (Revised)

## 0. Why This Revision

Aligns the product narrative with `002_MVP_v2.md` and `006_TechnicalTask_v2.md`: the product's primary value is portfolio risk clarity, not a directional trade signal. This avoids overstating an unvalidated edge and reduces regulatory exposure (see Section 4).

## 1. Executive Summary

CoinPilot AI is a crypto portfolio intelligence assistant deployed as a Telegram Mini App (TMA). It resolves market noise and cognitive overload in two ways: (1) it tells users what their own portfolio's risk actually looks like — concentration, correlation, directional exposure — and (2) it summarizes multi-factor market context (trend, order flow, derivatives, liquidations, sentiment) in plain language, without presenting an unproven directional call as if it were fact.

A directional Consensus Score with explicit trade setups (LONG/SHORT + entry/SL/TP) exists in the codebase as **Module B** but is not exposed to users until it passes the validation gate defined in Section 4 below and in `006_TechnicalTask_v2.md`, Module E.

---

## 2. Core Value Proposition

- **Portfolio Risk Intelligence (primary):** Concentration risk, correlation warnings, directional exposure, and stress-test scenarios computed from the user's actual holdings — value that doesn't depend on market direction being predicted correctly.
- **Market Context Engine (secondary, non-directional in v1):** Converts trend, order flow, derivatives positioning, liquidations, and sentiment into a composite score and a factual, descriptive explanation — not a buy/sell instruction.
- **Institutional AI Explainer:** LLMs (GPT-4o / Claude) are used strictly as an output formatter/explainer over pre-computed numbers, never as the source of the numbers themselves — this eliminates a specific failure mode (AI hallucinating financial figures) but does not by itself make the underlying numbers predictive.
- **Telegram Native UX:** Web-app interface inside Telegram, freemium energy-based access, viral referral loop with anti-abuse thresholds.

---

## 3. High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Telegram Mini App (React Frontend)                          │
│  Portfolio view · Market Context cards · Alerts · Referrals   │
└───────────────────────────┬───────────────────────────────────┘
                             │ REST (FastAPI, initData auth)
┌───────────────────────────▼───────────────────────────────────┐
│  Module D — API Layer (FastAPI)                               │
├─────────────────────────────────────────────────────────────┤
│  Module A          │ Module B          │ Module C             │
│  Data Ingestion &   │ Consensus/        │ LLM Explanation      │
│  Normalization      │ Intelligence      │ Layer                │
│  (Binance, Coinglass)│ Engine (scoring, │ (GPT-4o/Claude,       │
│  + historical store │  gated directional│  prompt templates,   │
│                      │  output)          │  Redis cache)        │
├─────────────────────────────────────────────────────────────┤
│  Module E — Backtesting & Validation (offline, gates Module B) │
├─────────────────────────────────────────────────────────────┤
│  PostgreSQL (users, portfolios, alerts, ledger) · Redis        │
└─────────────────────────────────────────────────────────────┘
```

Module B's directional output (LONG/SHORT + ATR trade setup) is computed but flagged `directional_enabled: false` at the API layer until Module E produces a passing validation report — see `003_ConsensusEngine.md` Section 5 and `007_API_Routes.md` Section 2.

---

## 4. Positioning & Risk Disclosure

CoinPilot AI is a market information and portfolio analytics tool, not a source of investment advice. This distinction is not just marketing language — it is a functional constraint that determines which fields the API is allowed to return and what the LLM layer is permitted to say (see `004_SystemPrompts_v2.md`, Section 1). Specific jurisdictional compliance requirements are out of scope for this document and should be reviewed with qualified legal counsel before public launch.

---

## 5. Differentiation

The individual data inputs (funding rates, OI, liquidation heatmaps, CVD) are available from several existing tools (Coinglass, TradingView, various Telegram signal bots). CoinPilot AI's differentiation in v1 is: (a) Telegram-native distribution and UX, (b) portfolio-centric framing rather than signal-centric framing, and (c) a documented commitment to not shipping unvalidated directional claims — a stance most competing "signal bot" products in this category do not take. This differentiation must be validated with real retention data (see `002_MVP_v2.md` Section 5) — it is a hypothesis, not a guarantee.
