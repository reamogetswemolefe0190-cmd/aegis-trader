# Aegis Trader

> An experimental algorithmic-trading engineering platform combining TypeScript strategy execution, risk controls, backtesting, broker abstraction and a Python machine-learning service.

[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Python](https://img.shields.io/badge/Python-ML-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/ML-scikit--learn-F7931E)](https://scikit-learn.org/)
[![Flask](https://img.shields.io/badge/API-Flask-black?logo=flask)](https://flask.palletsprojects.com/)
[![Vite](https://img.shields.io/badge/Frontend-Vite-646CFF?logo=vite)](https://vite.dev/)
[![Alpaca](https://img.shields.io/badge/Broker-Alpaca-yellow)](https://alpaca.markets/)

**Developer:** Reamogetswe Molefe  
**Status:** Experimental / Research Project

---

## Overview

**Aegis Trader** is an experimental algorithmic-trading platform built to explore how strategy logic, risk management, broker execution, simulation and machine learning can be separated into a modular trading system.

The project combines:

- a TypeScript trading engine
- technical-indicator strategies
- multi-asset simulation
- local paper trading
- Alpaca broker integration
- deterministic risk controls
- backtesting
- strategy parameter optimisation
- Python machine-learning predictions
- a browser-based trading dashboard

The project is designed primarily as a **software-engineering and quantitative experimentation environment**.

It does not claim to produce profitable trading strategies.

---

# System Architecture

```text
                         AEGIS TRADER
                              │
                              ▼
                    ┌───────────────────┐
                    │   Vite Dashboard  │
                    │    TypeScript     │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   Trading Robot   │
                    │ Signal Execution  │
                    └─────────┬─────────┘
                              │
              ┌───────────────┼────────────────┐
              │               │                │
              ▼               ▼                ▼
        Strategy Engine   Risk Governor     ML Predictor
              │               │                │
              │               │                ▼
              │               │          Python / Flask
              │               │                │
              │               │         Random Forest
              │               │
              └───────────────┼────────────────┐
                              │                │
                              ▼                ▼
                         Broker Layer      Simulator
                              │                │
                     ┌────────┴────────┐       │
                     │                 │       │
                     ▼                 ▼       ▼
                Local Broker      Alpaca API  Market Data
                                                │
                                   ┌────────────┴───────────┐
                                   ▼                        ▼
                              CryptoCompare           Synthetic Data
```

---

# Technology Stack

## Trading Engine

- TypeScript
- JavaScript
- Vite

## Backend / APIs

- Node.js
- Express
- Flask
- REST APIs

## Machine Learning

- Python
- pandas
- NumPy
- scikit-learn
- RandomForestClassifier
- joblib

## Market / Broker Integration

- Alpaca API
- Alpaca Paper Trading
- Alpaca Market Data
- CryptoCompare
- Coinbase fallback for selected crypto history

## Visualisation

- Lightweight Charts
- custom dashboard components

---

# Core Architecture

The trading system is split into independent components.

```text
src/core/
│
├── Broker.ts
├── ReviewEngine.ts
├── RiskGovernor.ts
├── Robot.ts
├── Simulator.ts
├── SonOfAlton.ts
├── Strategies.ts
└── Types.ts
```

Each component is responsible for a different layer of the system.

---

# Broker Abstraction

Aegis Trader uses an `IBroker` interface so the trading logic does not need to know whether orders are being executed against a local simulator or an external broker.

The interface exposes operations including:

```text
initialize broker
get account balance
get positions
place order
liquidate positions
retrieve news
```

Current broker implementations include:

```text
LocalBroker
AlpacaBroker
```

---

## Local Broker

The local broker provides a safe simulation environment.

It tracks:

- cash
- positions
- average entry price
- unrealised P&L
- realised P&L
- trades
- simulated commission/slippage

The simulator currently models a:

```text
0.1%
```

transaction cost on local trades.

---

## Alpaca Integration

The project contains an Alpaca broker implementation supporting:

```text
Paper account
Live account architecture
Market data
Positions
Orders
Account information
News
```

The browser communicates through Node/Vite proxy endpoints such as:

```text
/api/alpaca-paper
/api/alpaca-live
/api/alpaca-data
```

For development and portfolio demonstration, **paper trading should be preferred**.

---

# Strategy Engine

Technical strategies are separated into:

```text
Strategies.ts
```

The engine calculates indicators directly from market candles and returns one of:

```text
BUY
SELL
HOLD
```

Current strategy logic includes:

- Simple Moving Average crossover
- RSI mean reversion
- MACD
- Bollinger Bands
- ML prediction mode

---

# Simple Moving Average

The engine can calculate rolling simple moving averages.

Example concept:

```text
Fast SMA
    │
    │ crosses above
    ▼
Slow SMA

→ BUY signal
```

A downward crossover can generate a sell signal.

---

# RSI

The project implements the Relative Strength Index using Wilder-style smoothing.

The strategy can use configurable:

```text
RSI period
Oversold threshold
Overbought threshold
```

This supports experimental mean-reversion strategies.

---

# MACD

Aegis calculates:

```text
Fast EMA
Slow EMA
MACD Line
Signal Line
Histogram
```

The strategy can then evaluate MACD crossovers as possible trading signals.

---

# Bollinger Bands

The strategy engine also supports:

```text
Moving Average
Standard Deviation
Upper Band
Lower Band
```

This allows experimentation with price deviation and mean-reversion behaviour.

---

# Risk Governor

A dedicated `RiskGovernor` sits between strategy signals and broker execution.

This is one of the most important design decisions in the project.

```text
Trading Signal
      │
      ▼
Proposed Order
      │
      ▼
┌──────────────────────┐
│    Risk Governor     │
├──────────────────────┤
│ Manual Freeze        │
│ Daily Loss Limit     │
│ Drawdown Limit       │
│ Concentration Limit  │
└──────────┬───────────┘
           │
      Approved?
       /      \
     Yes       No
      │         │
      ▼         ▼
   Broker     Reject
```

---

## Manual Freeze

Trading can be disabled globally using an operator-controlled freeze flag.

When active:

```text
All new orders are rejected.
```

---

## Daily Loss Limit

The governor records the account's starting equity and can block new orders after the configured daily-loss threshold is reached.

Example:

```text
Starting equity: R100,000
Maximum daily loss: 5%

Loss reaches R5,000
        ↓
Further orders blocked
```

---

## Maximum Drawdown

The system tracks peak equity.

It calculates:

```text
Drawdown %
    =
(Peak Equity - Current Equity)
──────────────────────────────
          Peak Equity
```

Orders can be rejected if the configured drawdown limit is exceeded.

---

## Position Concentration

For buy orders, Aegis calculates the resulting asset concentration relative to account equity.

This helps prevent a single position from exceeding the configured portfolio limit.

---

# Position-Level Risk Controls

The trading robot also supports per-position controls.

These include:

```text
Stop Loss
Take Profit
Maximum Position Size
Maximum Number of Positions
```

The robot checks position prices as simulated ticks arrive.

When a stop-loss or take-profit threshold is reached, the system can initiate an exit.

---

# Emergency Kill Switch

The system contains an emergency liquidation flow.

```text
Risk Breach
    │
    ▼
Trading Halt
    │
    ▼
Emergency Kill
    │
    ▼
Close Positions
```

This architecture separates risk response from normal signal generation.

---

# Advisor vs Automatic Execution

Aegis supports the concept of two execution modes.

## Automatic

```text
Signal
   ↓
Risk Validation
   ↓
Order
```

## Advisor

```text
Signal
   ↓
Risk Validation
   ↓
Human Confirmation
   ↓
Order
```

This makes it possible to experiment with human-in-the-loop trading instead of always allowing automated execution.

---

# Multi-Asset Simulator

The simulator maintains independent state for multiple assets.

Current configured examples include:

```text
BTC/USD
ETH/USD
SOL/USD
DOGE/USD
LTC/USD

AAPL
TSLA
NVDA
MSFT
GOOGL
```

Each asset can maintain:

- price
- candle history
- volatility assumptions
- drift
- active strategy
- risk configuration

---

# Market Data

For supported cryptocurrency assets, the simulator attempts to retrieve historical data using CryptoCompare.

If external data is unavailable, Aegis can fall back to synthetic price generation.

Synthetic prices are generated using a stochastic process based on:

```text
Drift
Volatility
Gaussian noise
```

This allows the application to remain usable even when no external data source is available.

---

# Backtesting

Aegis includes a historical backtesting engine.

A backtest processes candles chronologically:

```text
Historical Candles
       │
       ▼
Technical Indicators
       │
       ▼
Trading Signal
       │
       ▼
Simulated Execution
       │
       ▼
Equity Update
       │
       ▼
Performance Metrics
```

---

## Backtest Metrics

The project calculates metrics including:

```text
Initial Equity
Ending Equity
Net Profit
Net Return %
Trade Count
Win Rate
Maximum Drawdown
Simplified Sharpe-style metric
```

The backtester also models transaction costs.

---

# Performance Review Engine

`ReviewEngine.ts` analyses completed trades and calculates:

```text
Win Rate
Total Trades
Profit Factor
Average Win
Average Loss
```

It also creates strategy feedback based on the results.

---

# Parameter Optimisation

Aegis includes a grid-search optimiser.

Instead of manually testing one set of parameters, the system can evaluate several combinations.

For example, an SMA strategy can test:

```text
Fast periods:
5
10
15
20

Slow periods:
20
30
45
60
```

Each valid combination is backtested and compared.

The system can then identify the configuration with the strongest result **on the tested historical sample**.

This is an experimental optimisation tool and does not imply future profitability.

---

# Overfitting Experiments

The repository also contains comparative backtesting scripts exploring the difference between:

```text
Continuous in-sample optimisation
```

and:

```text
Validation-gated parameter changes
```

One harness separates data into in-sample and out-of-sample windows before accepting a parameter change.

```text
Historical Window
       │
       ├───────────────┐
       ▼               ▼
   In-Sample      Out-of-Sample
       │               │
       ▼               │
Parameter Search       │
       │               │
       └───────┬───────┘
               ▼
       Validation Gate
               │
          ┌────┴────┐
          ▼         ▼
       Accept     Reject
```

This was added to explore how strategy optimisation can overfit historical data.

---

# Machine-Learning Service

Aegis includes a separate Python ML service:

```text
ml_backend/app.py
```

The service runs using Flask.

It provides endpoints for:

```text
/train
/predict
```

---

# ML Feature Engineering

The Python service calculates features including:

```text
1-period return
5-period return
5-period rolling volatility
14-period RSI
```

These are generated using pandas before model training.

---

# ML Target

The prototype model attempts to classify whether:

```text
Future Close Price > Current Close Price
```

five candles into the future.

The target is therefore binary:

```text
1 → Future price higher
0 → Future price not higher
```

---

# Random Forest Model

The current experimental classifier is:

```python
RandomForestClassifier(
    n_estimators=100,
    max_depth=5,
    random_state=42
)
```

The model is trained using engineered technical features.

---

# Cross-Validation Gate

Before a newly trained model is saved, Aegis performs:

```text
5-fold cross-validation
```

The current prototype requires the mean validation accuracy to meet a minimum threshold before accepting the model.

This is designed to prevent obviously weak models from automatically replacing an existing model.

Cross-validation accuracy alone, however, does not establish trading profitability.

---

# ML Predictions

When a trained model exists, the `/predict` endpoint calculates the latest feature vector and returns:

```text
BUY
SELL
HOLD
```

Current experimental thresholds are:

```text
Probability > 0.58 → BUY
Probability < 0.42 → SELL
Otherwise          → HOLD
```

If no trained model is available, the API defaults to:

```text
HOLD
```

---

# TypeScript + Python Integration

The trading robot communicates with the Python model using HTTP.

```text
TypeScript Robot
      │
      │ Recent candles
      ▼
Flask ML API
      │
      ▼
Feature Engineering
      │
      ▼
Random Forest
      │
      ▼
BUY / SELL / HOLD
      │
      ▼
TypeScript Robot
      │
      ▼
Risk Governor
```

This gives the project a small multi-service architecture rather than placing all model logic inside the browser application.

---

# Model Persistence

Accepted scikit-learn models are stored using:

```text
joblib
```

Model files are saved inside:

```text
ml_backend/models/
```

and excluded from source control.

---

# Historical Data Utilities

The repository contains scripts for retrieving historical Alpaca data.

The fetcher can save OHLCV data into:

```text
data/history/
```

Historical datasets are excluded from Git using `.gitignore`.

---

# Broker Server

The project includes an Express server that can proxy requests to:

```text
Alpaca Paper API
Alpaca Live API
Alpaca Data API
```

It also serves the compiled frontend from:

```text
dist/
```

---

# Security Note

The current broker integration is an experimental architecture.

Broker API credentials are currently passed through the browser-side broker flow and proxied by the Node server.

For any production deployment, broker credentials should instead be stored exclusively on a trusted backend and never exposed to browser JavaScript.

For portfolio demonstrations:

> **Use paper-trading credentials only.**

Do not use real-money broker credentials with the current prototype architecture.

---

# Dashboard

The TypeScript frontend provides a browser-based trading terminal.

The interface is designed to expose:

- account information
- active positions
- market data
- charts
- strategies
- risk configuration
- robot state
- logs
- backtest results
- performance reviews

The dashboard communicates with the underlying trading engine rather than containing the strategy calculations itself.

---

# Repository Structure

```text
trading_bot/
│
├── ml_backend/
│   └── app.py
│
├── scripts/
│   ├── compare_backtests.js
│   ├── compare_backtests.py
│   ├── fetch_history.js
│   └── print_ascii_chart.js
│
├── src/
│   ├── core/
│   │   ├── Broker.ts
│   │   ├── ReviewEngine.ts
│   │   ├── RiskGovernor.ts
│   │   ├── Robot.ts
│   │   ├── Simulator.ts
│   │   ├── SonOfAlton.ts
│   │   ├── Strategies.ts
│   │   └── Types.ts
│   │
│   ├── ui/
│   │   ├── ChartManager.ts
│   │   └── Dashboard.ts
│   │
│   ├── styles/
│   │   └── main.css
│   │
│   └── main.ts
│
├── index.html
├── server.js
├── requirements.txt
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

---

# Running the TypeScript Application

## 1. Clone the Repository

```bash
git clone https://github.com/reamogetswemolefe0190-cmd/trading_bot.git
cd trading_bot
```

---

## 2. Install Node Dependencies

```bash
npm install
```

---

## 3. Start Development Mode

```bash
npm run dev
```

Vite runs by default at:

```text
http://localhost:3000
```

---

# Running the ML Service

The ML backend is separate from the TypeScript frontend.

## 1. Create a Python Environment

### Windows

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
```

---

## 2. Install Python Dependencies

```bash
pip install -r requirements.txt
```

---

## 3. Start Flask

```bash
python ml_backend/app.py
```

The ML service runs by default at:

```text
http://localhost:5000
```

---

# Production Build

Compile the TypeScript/Vite frontend using:

```bash
npm run build
```

Then start the Express server:

```bash
npm start
```

---

# Paper Trading

The broker architecture supports Alpaca Paper Trading.

Use paper credentials for experimentation.

Typical credentials required by the broker are:

```text
APCA API Key ID
APCA API Secret Key
```

Never commit broker credentials to Git.

The repository already ignores:

```text
.env
```

---

# Fetching Historical Data

Historical Alpaca data can be downloaded using:

```bash
node scripts/fetch_history.js AAPL 365
```

Credentials can be supplied through the appropriate environment variables or script parameters.

Downloaded history is stored under:

```text
data/history/
```

and excluded from source control.

---

# Experimental Nature

Aegis Trader is an engineering project, not a validated trading product.

The following should not be inferred from the repository:

- guaranteed profitability
- reliable future-price prediction
- investment advice
- production-grade execution reliability
- institutional risk management
- validated alpha
- real-world strategy profitability

Backtests can overfit.

Machine-learning accuracy does not equal financial returns.

Synthetic data does not reproduce real markets perfectly.

Historical performance does not predict future results.

---

# What the Project Demonstrates

From a software-engineering perspective, Aegis Trader demonstrates work with:

```text
TypeScript
Python
Node.js
Express
Flask
REST APIs
scikit-learn
pandas
NumPy
Machine learning
Feature engineering
Broker abstraction
Risk controls
Backtesting
Technical indicators
Parameter optimisation
Cross-validation
Multi-service architecture
Market-data APIs
Algorithmic state machines
Data visualisation
```

---

# Engineering Principles

## Risk Before Execution

Strategy signals do not automatically imply an order.

Every proposed execution passes through deterministic risk controls.

---

## Separate Simulation From Brokerage

The same trading engine can interact with different broker implementations.

---

## Separate Machine Learning From Execution

The Python model generates a signal.

The TypeScript trading engine still decides whether that signal is allowed to become an order.

---

## Fail Toward HOLD

When the ML service is unavailable or a prediction cannot be produced safely, the robot defaults to:

```text
HOLD
```

rather than blindly creating a trade.

---

## Measure Strategies Instead of Assuming

The repository includes:

- backtesting
- performance metrics
- grid-search experiments
- cross-validation
- out-of-sample experiments

The objective is to evaluate strategy behaviour rather than assume that an indicator is profitable.

---

# Future Improvements

Potential next stages include:

- walk-forward ML validation
- time-series-specific cross-validation
- stronger leakage prevention
- realistic slippage models
- benchmark comparison
- portfolio-level backtesting
- transaction-cost modelling
- persistent trade database
- WebSocket market feeds
- secure server-side broker credentials
- Dockerised services
- automated tests
- CI/CD
- model monitoring
- feature importance reporting
- calibration metrics
- paper-trading performance journal
- observability and structured logs

---

# About the Developer

## Reamogetswe Molefe

I am a Mechanical Engineering student at the **University of Johannesburg** who independently builds software across AI, full-stack development, fintech and quantitative experimentation.

Aegis Trader was built to explore the engineering challenges behind automated decision systems:

- data ingestion
- strategy evaluation
- model inference
- risk validation
- execution
- simulation
- performance review

Other projects include:

- **Kohort** — Python multi-model AI market-research engine
- **CreatorCashFlow** — Node.js / Express SaaS platform
- **Yieldly** — Next.js / TypeScript digital stokvel prototype
- **Zippy** — Flutter fintech application prototype
- **SplitFare** — React Native expense-splitting app with receipt OCR

GitHub:

https://github.com/reamogetswemolefe0190-cmd

---

# Disclaimer

Aegis Trader is an experimental software-engineering project.

It is **not investment advice**, does not guarantee financial returns and should not be relied upon for real-money investment decisions.

Use simulated or paper-trading environments when experimenting with the project.

---

<p align="center">
  <strong>Aegis Trader</strong><br>
  Signals propose. Risk decides.
</p>
