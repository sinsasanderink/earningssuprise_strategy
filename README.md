# High-Confidence Earnings Surprise Trading Strategy

## Overview
This system performs a market-wide scan of upcoming earnings calls, scores every candidate using a transparent 100-point multi-factor confidence model, and surfaces the highest-probability earnings beat opportunities. It currently operates purely on static signals—no reinforcement learning is deployed yet; RL is reserved as a planned future enhancement to adaptively mitigate pricing-in and institutional asymmetry.

## Current Functionality

### 1. Comprehensive Earnings Discovery
- **Universe:** Scans all publicly reporting companies with earnings in the next 1–2 weeks (not limited to major indices).  
- **Data Source:** Pulls real-time earnings dates, consensus EPS/revenue estimates, and related market data from professional APIs (e.g., Finnhub) to build a full opportunity set.  
- **No Artificial Limits:** Entire market is eligible; no cap like “only S&P 500” or similar.

### 2. Multi-Factor Confidence Scoring (100-Point System)
Each stock is evaluated across three broad signal categories, combined into a normalized confidence score (0–100) intended to reflect probability of a meaningful positive earnings surprise and associated return.

#### Earnings-Specific Metrics (50 points)
- **Earnings Surprise History (20 pts):** Historical frequency of beats/misses over the last 4 quarters.  
- **Analyst Revisions (15 pts):** Recent upward or downward changes in earnings estimates, capturing momentum in expectations.  
- **Sector Performance (10 pts):** How the stock’s sector is performing on earnings relative to the broader market (sectoral tailwinds/headwinds).  
- **Short Squeeze Potential (5 pts):** Elevated short interest that could act as a catalyst if a surprise materializes.

#### Technical Metrics (30 points)
- **Price Momentum (15 pts):** Short- and medium-term trend confirmation (e.g., 5-day and 20-day returns).  
- **RSI Analysis (10 pts):** Favorable relative strength index indicating momentum without immediate overbought risk.  
- **Volume Confirmation (5 pts):** Above-average or supportive volume lending conviction to recent moves.

#### Analyst & Fundamental Signals (20 points)
- **Analyst Sentiment (12 pts):** Aggregated ratings, price target upside, and consensus direction from sell-side coverage.  
- **Fundamental Health (5 pts):** Valuation screens (e.g., reasonable P/E) and profitability metrics (e.g., margins).  
- **Insider / Institutional Proxy (3 pts):** High institutional ownership or equivalent as a loose confidence proxy.

### 3. Thresholding & Triage
- **High-confidence cutoff:** Stocks scoring ≥60 are flagged for further consideration.  
- **Premium plays:** Stocks scoring ≥70 receive prioritized attention.  
- **Transparency:** Each recommendation includes a breakdown of sub-scores so users can audit why a stock scored as it did.

### 4. Market Intelligence Layers
- Revision momentum to detect shifting analyst consensus.  
- Sector rotation context to avoid stale themes.  
- Short interest tracking for squeeze dynamics.  
- Historical beat patterns used as priors to temper overreaction to noisy new data.

## Backtest Findings & Real-World Dynamics
Backtesting shows that a large portion of the anticipated earnings move is **priced in before the official release**—stocks often drift ahead of earnings, and only materially unexpected results (far above or below consensus) produce outsized post-event reactions. Furthermore, institutional players and hedge funds frequently consume and front-run predictable edges because of:
- **Early/privileged access to information.**  
- **Scale and latency advantages** that allow them to place large directional bets before public signals fully propagate.

This creates a structural headwind for static signal-based retail or systematic strategies, reducing edge unless the model adapts to these dynamics.

## Output
- Ranked list of earnings candidates with confidence scores.  
- Detailed factor breakdown per stock (why it scored what it did).  
- EPS and revenue estimates, official earnings date.  
- Risk/reward commentary for potential position sizing.  
- CSV export for tracking, auditing, and feeding downstream systems.

## Configuration
- Thresholds (e.g., high-confidence cutoff) are adjustable.  
- Weighting of subcomponents can be tuned to fit different regimes or risk tolerances.  
- Time windows for momentum, lookback for revision history, and sector definitions are parameterized for experimentation.

## Future Work: Reinforcement Learning Enhancements

### Motivation
Static scoring identifies potential opportunities but cannot:
- **Neutralize pricing-in effects** where expected moves are already embedded in pre-earnings prices.  
- **Adapt to changing institutional behavior** or front-running pressure.  
- **Calibrate timing and sizing dynamically** based on how earlier similar signals actually performed.

### Proposed RL Layer (Post-hoc / Layered on Existing Pipeline)

#### Core Goals
- Learn **when to act or abstain** given inferred crowding/pricing-in.  
- **Time entries and exits** optimally around earnings events.  
- **Scale position size** based on dynamic confidence and regime.  
- Optionally **switch to hedged variants** when pure directional exposure is too risky.

#### Environment Design
- **State:** Multi-factor confidence vector + pre-earnings drift proxies + inferred pricing-in signal (e.g., abnormal volume, implied move vs historical) + sector regime + recent prediction residuals + time-to-event.  
- **Actions:** Long / Short / Flat, with extensions for position sizing and overlays (e.g., hedge, delay).  
- **Reward:** Realized post-earnings return adjusted for risk and transaction cost, with penalties for overconfidence and excessive waiting; rewards for avoiding low-edge scenarios.  
- **Dynamics:** Simulate institutional absorption via anticipatory price shifts and slippage, so the agent internalizes when edge is already consumed.

#### Learning Strategy
- **Offline pretraining:** Use logged historical earnings episodes and batch RL techniques (e.g., Conservative Q-Learning, BCQ) to bootstrap a safe policy.  
- **Online fine-tuning:** Gradually adapt to new regimes with conservative updates, leveraging fresh outcomes for continual improvement.  
- **Uncertainty-aware decisioning:** Combine static confidence score as a prior with learned policy output to stabilize behavior.  
- **Meta-adaptation:** Detect regime changes (e.g., sudden shift in how pricing-in manifests) and adjust learning rates or gating dynamically.

#### Institutional Asymmetry Mitigation
- Explicitly model **latency / information leakage** so agent learns to defer or hedge when traditional edges are eroded.  
- Incorporate **alternative public early signals** (e.g., high-frequency sentiment, order flow proxies) to partially regain timing advantages.  
- Ensemble gating to prevent over-reliance on any single brittle signal.

#### Feedback Loop
Track prediction errors, feed them back to recalibrate the static scorer, and update the RL agent’s state representation—creating a self-correcting, co-evolving pipeline.

## Limitations
- Static scoring cannot fully overcome information asymmetries or eliminate pricing-in; it surfaces probability-weighted opportunities but still suffers from crowding and institutional speed.  
- Backtest realism hinges on accurate modeling of slippage and pre-earnings drift.  
- RL extension introduces complexity and risk of overfitting; requires careful regime validation, conservative deployment, and robust uncertainty handling.

## Roadmap / Future Extensions
- Add options-implied surface features (volatility skew, term structure) to both static and RL inputs.  
- Hierarchical agents separating timing vs sizing decisions.  
- Jointly learn execution impact to internalize cost-aware decision-making.  
- Causal disentanglement of signal vs noise in earnings surprise prediction.  
- Multi-agent simulation of competing institutional strategies to stress-test robustness.

