# 003 — Consensus/Market Intelligence Engine Specification — v2 (Revised)

## 0. Why This Revision

The original spec exposed a directional call (LONG/SHORT/NEUTRAL) and explicit ATR trade setup as the default, unconditional output. This revision keeps the same math but adds an explicit **mode gate**, consistent with `002_MVP_v2.md` Section 4 and `006_TechnicalTask_v2.md` Module E: the engine always computes a score internally, but only exposes a directional recommendation once that score has passed backtesting validation.

---

## 1. Mathematical Architecture (Unchanged)

The engine evaluates real-time market data across 5 core quantitative vectors to produce a unified score $S \in [0, 100]$.

---

## 2. Factor Weights & Scoring Matrix (Unchanged — Pending Validation)

| Vector | Weight | Metrics Analyzed | Scoring Rule |
|---|---|---|---|
| **Trend & Momentum** | 30% | EMA 20/50/200, RSI (14), MACD | Bullish cross = +100, Bearish cross = 0 |
| **Order Flow & Delta** | 25% | Cumulative Volume Delta (CVD), Order Book Depth | Positive CVD & Bid depth = High score |
| **Derivatives Positioning** | 20% | Open Interest (OI), Funding Rates | OI Surge + Neutral Funding = Bullish |
| **Liquidations Heatmap** | 15% | Nearby Short/Long liquidation pools | Pool magnet proximity drives target |
| **Macro & News Sentiment** | 10% | CryptoPanic API sentiment score | Weighted average of top news |

**Note:** these weights are the initial hypothesis, not a validated model. They must be run through the backtesting module (`006_TechnicalTask_v2.md`, Module E) before Section 4 below is exposed to any user.

---

## 3. Trade Setup Generation (ATR Driven) — Directional Mode Only

To eliminate arbitrary price targets, trade levels are calculated dynamically using the Average True Range (ATR):

- **Entry Range:** Current Market Price $\pm (0.2 \times \text{ATR}_{14})$
- **Stop Loss (SL):** Entry $- (1.5 \times \text{ATR}_{14})$ for LONG / Entry $+ (1.5 \times \text{ATR}_{14})$ for SHORT
- **Take Profit 1 (TP1):** Risk/Reward 1:1.5
- **Take Profit 2 (TP2):** Risk/Reward 1:3.0

This entire section is computed at all times for internal backtesting purposes but is only returned by the API when `directional_enabled = true` (see Section 5).

---

## 4. Output Modes

### Context Mode (default, ships in MVP v2)

- Returns the composite score $S$ and per-factor breakdown internally, but the API surfaces it to the frontend as a **qualitative summary** (e.g., "elevated" / "neutral" / "weak" order flow), not a numeric score framed as a signal.
- No LONG/SHORT/NEUTRAL label. No entry/SL/TP fields in the API response.
- LLM explanation layer (`004_SystemPrompts_v2.md`) is constrained to describe *what is happening*, not *what to do*.

### Directional Mode (gated, v2.0+)

- Original behavior: numeric score $S$, direction label (Section 5 below), and ATR trade setup.
- Gated behind a passing report from Module E (backtest period, win rate, expected return, max drawdown — see `006_TechnicalTask_v2.md`).
- When shipped, the historical track record (win rate, expected return, sample size) must be displayed alongside the score in the UI — not just the score in isolation — so users see the same accuracy context the team used to decide whether to ship it.
- If live performance materially diverges from backtested performance after shipping, this mode reverts to Context Mode automatically (manual override logged) pending re-validation.

---

## 5. Directional Thresholds (Directional Mode Only — Not Active in MVP v2)

- **Score 65 – 100:** Strong LONG Signal
- **Score 36 – 64:** NEUTRAL / Hold Range
- **Score 0 – 35:** Strong SHORT Signal

These thresholds are themselves part of what must be validated — there's no evidence yet that 65/36 are the correct cut points rather than, say, 70/30. The backtest in Module E should test threshold sensitivity, not just the weights.
