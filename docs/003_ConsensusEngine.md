# 003 — Consensus Engine Specification

## 1. Mathematical Architecture
The Consensus Engine evaluates real-time market data across 5 core quantitative vectors to produce a unified score $S \in [0, 100]$ and direction recommendation.

---

## 2. Factor Weights & Scoring Matrix

| Vector | Weight | Metrics Analyzed | Scoring Rule |
| :--- | :--- | :--- | :--- |
| **Trend & Momentum** | 30% | EMA 20/50/200, RSI (14), MACD | Bullish cross = +100, Bearish cross = 0 |
| **Order Flow & Delta** | 25% | Cumulative Volume Delta (CVD), Order Book Depth | Positive CVD & Bid depth = High score |
| **Derivatives Positioning**| 20% | Open Interest (OI), Funding Rates | OI Surge + Neutral Funding = Bullish |
| **Liquidations Heatmap** | 15% | Nearby Short/Long liquidation pools | Pool magnet proximity drives target |
| **Macro & News Sentiment**| 10% | CryptoPanic API sentiment score | Weighted average of top news |

---

## 3. Trade Setup Generation (ATR Driven)
To eliminate arbitrary price targets, trade levels are calculated dynamically using the Average True Range (ATR):

- **Entry Range:** Current Market Price $\pm (0.2 \times \text{ATR}_{14})$
- **Stop Loss (SL):** $\text{Entry} - (1.5 \times \text{ATR}_{14})$ for LONG / $\text{Entry} + (1.5 \times \text{ATR}_{14})$ for SHORT
- **Take Profit 1 (TP1):** Risk/Reward 1:1.5
- **Take Profit 2 (TP2):** Risk/Reward 1:3.0

---

## 4. Directional Thresholds
- **Score 65 – 100:** Strong LONG Signal
- **Score 36 – 64:** NEUTRAL / Hold Range
- **Score 0 – 35:** Strong SHORT Signal
