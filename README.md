# AI Forex Bot (EURUSD, XAUUSD) — Exness MT5

This repository provides two fully working MetaTrader 5 trading solutions for Exness:

1) High-Frequency Trading EA (HFT, tick-based, no external latency)
   - AiHFTEA.mq5
   - Runs entirely inside MT5 for minimal latency.
   - Microstructure momentum, tight SL/TP, strict throttles (cooldown, orders per minute), daily risk guard, and max-hold.

2) API-driven AI EA (bar-based with Python FastAPI server)
   - AiTraderEA.mq5 + server FastAPI for signal generation
   - Technical rules baseline (EMA50/EMA200 trend, RSI momentum, ATR-based sizing).
   - Easier to extend with ML or research workflows.

IMPORTANT: Always start on a DEMO account. HFT on retail infrastructure is highly sensitive to latency, spreads, and slippage. Use a VPS near your broker’s servers.

## Quick Start — HFT EA (Recommended for high frequency)

1) Open MT5 (Exness) and the exact broker symbols (e.g., EURUSD, XAUUSD or XAUUSDm).
2) Copy the EA file:
   - Place mql5/Experts/AiHFTEA.mq5 into your MT5 Data Folder:
     - MT5 -> File -> Open Data Folder -> MQL5/Experts/
3) Compile in MetaEditor (F7).
4) Attach AiHFTEA to EURUSD and XAUUSD charts (M1 recommended).
5) Suggested inputs:
   - EURUSD:
     - SlPoints: 10–20, TpPoints: 10–20
     - MaxSpreadPoints: 20–30
     - TickWindow: 120–200
     - BuyImbalance: 0.58–0.62, SellImbalance: 0.42–0.38
     - MinStdPoints: 1–2, MaxStdPoints: 20–40
   - XAUUSD:
     - SlPoints: 150–300, TpPoints: 150–300
     - MaxSpreadPoints: 800–1200
     - TickWindow: 120–200
     - MinStdPoints: 15–40, MaxStdPoints: 200–600
6) Start with FixedLot = 0.01 (or risk-based sizing very small) and tune parameters only after demo testing.

Key risk controls in HFT EA:
- CooldownMsBetweenTrades, MaxOrdersPerMinute, MaxDailyTrades
- DailyLossLimitPercent (equity-based lockout)
- MaxHoldSeconds (force exit if trade lingers)
- Spread guard (MaxSpreadPoints)

See docs/HFT-README.md for more detail.

## Quick Start — API-driven AI EA

This option separates signal logic into a Python FastAPI server.

1) Install server (Windows, Python 3.10+):
   - cd server
   - python -m venv .venv
   - . .venv/Scripts/activate (PowerShell: .\.venv\Scripts\Activate.ps1)
   - pip install -r requirements.txt
   - uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
2) In MT5: Tools -> Options -> Expert Advisors -> Allow WebRequest for:
   - http://127.0.0.1:8000
3) Copy mql5/Experts/AiTraderEA.mq5 to MQL5/Experts and mql5/Include/SimpleJson.mqh to MQL5/Include.
4) Compile AiTraderEA.mq5 and attach it to EURUSD and XAUUSD charts.
5) Configure inputs:
   - ApiBaseUrl: http://127.0.0.1:8000
   - RiskPerTradePercent: 1.0 (demo)
   - AtrPeriod: 14, AtrMultForSL: 2.0
   - RewardRiskRatio: 1.5
   - MaxSpreadPoints: ~35 (EURUSD), ~500–1200 (XAUUSD depending on broker)
   - BarsToSend: 300

Server endpoints:
- GET /health -> { status: "ok" }
- POST /signal -> { action, sl_points, tp_points, confidence, reason }

You can replace the rules in server/app/signal.py with a trained ML model.

## Repository Structure

- mql5/
  - Experts/
    - AiHFTEA.mq5           (High-frequency EA)
    - AiTraderEA.mq5        (API-driven EA)
  - Include/
    - SimpleJson.mqh        (minimal JSON helper for API EA)
- server/
  - app/
    - main.py               (FastAPI entry)
    - signal.py             (signal logic)
    - indicators.py         (EMA/RSI/ATR)
    - schemas.py            (request/response models)
    - config.py             (parameters)
  - requirements.txt
- docs/
  - HFT-README.md

## Notes and Recommendations

- Use a low-latency VPS for HFT (target sub-10ms ping). Prefer raw accounts and measure slippage.
- Exness symbol names may include suffixes (e.g., XAUUSDm). Attach EAs to the exact symbol you intend to trade.
- Always demo first. Validate behavior across sessions (London/NY) and news events.

## Disclaimer

This software is for educational purposes only. Trading involves substantial risk. You are solely responsible for any use of this software and all trading decisions you make.