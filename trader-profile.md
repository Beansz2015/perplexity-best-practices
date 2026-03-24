# Trader Profile

This document captures the trader's style, preferences, and strategic context
for the Deribit Verdict Engine project. Attach this file at the start of any
new conversation (coding or strategy) to bootstrap full context instantly.

Last updated: 2026-03-14

---

## 1. Background

    Exchange experience:    Former employee of a digital assets exchange (role: operations/trading side)
    Trading experience:     Many years actively trading crypto; highly experienced
    Current setup:          Deribit perpetuals (BTC-PERPETUAL); also trades spot
    Primary instrument:     BTC-PERPETUAL on Deribit
    Session style:          Part-time / discretionary; trades when conditions are met,
                            not on a fixed schedule. Does not trade every session.
    Timezone:               GMT+8 (Penang, Malaysia)
    Other context:          Software engineering background (ex-dev); understands code
                            and can review VB.NET implementations critically.
                            Also runs a separate business (hostel). Trading is a
                            significant but not sole focus.

---

## 2. Trading Style

    Primary style:          Momentum-Informed Scalper (Hybrid Style C)
                            Uses multi-timeframe bias (5m/15m/1h) to determine
                            direction, then executes entries and exits on 1m chart.
                            NOT a pure scalper (fixed % targets) and NOT a pure
                            momentum trader (ride indefinitely). Trades between
                            structural swing levels.

    Preferred timeframe:    1m execution chart; 5m/15m for regime and bias

    Entry logic:            Price breaks above/below previous swing high/low,
                            confirmed by impulse (ROC) and volume spike.
                            Requires structural breakout -- does not chase candles.

    Profit targets:         Previous swing high (for longs) / previous swing low
                            (for shorts). Structural targets, NOT fixed % or ATR.
                            This means R:R is dynamic depending on swing size.

    Stop-loss placement:    Below previous swing low (longs) / above previous
                            swing high (shorts). Structural stops, NOT ATR-based.
                            Stop distance defines risk per trade, not a fixed %.

    Hold duration:          2-15 minutes typical. Will hold through 2-3 red candles
                            IF trend is confirmed intact (RSI > 60 check).
                            Does NOT hold overnight -- always flat at end of session.

    Risk tolerance:         Medium. Comfortable with short retracements during
                            holds but has clear exit rules.

    Preferred market state: Both trending AND range-bound markets are acceptable,
                            as long as there is a high-probability swing opportunity
                            (clear high and low to trade between).
                            Pure chop with no swing structure = no trade.

    Trade frequency:        Selective. Only enters when checklist conditions are met.
                            Prefers fewer high-quality trades over frequent low-quality.

---

## 3. Indicator Preferences

    ROC(9):          PREFERRED | Fast impulse confirmation for breakout entries.
    RSI(9):          PREFERRED | Hold/exit decisions during trades. Divergence detection.
    DMI/ADX(9):      PREFERRED | Core regime filter on 5m chart.
    ATR(7):          PREFERRED | Position sizing ONLY. NOT for stop placement.
    Volume SMA(9):   PREFERRED | Volume spike detection. Breakout requires volume > 3x SMA(9).
    VWAP:            PREFERRED | Institutional fair-value reference. Session-reset at 00:00 UTC.
    Bollinger/BBW:   PREFERRED | BBW for squeeze detection only.
    EMA Ribbon:      PREFERRED | 9/21/50 on 1m for dynamic trend structure.
    Funding Rate:    PREFERRED | Contrarian crowd-positioning signal (Step 3 modifier only).
    Open Interest:   PREFERRED | OI change direction + price direction = trend quality.
    Order Flow/OFI:  PREFERRED | Real-time buy/sell pressure from L2 order book.
    Liquidations:    PREFERRED | Cascade detection. Penalty-only signal.
    OBV:             NEUTRAL   | Tier 3 -- nice to have.
    Donchian(20):    NEUTRAL   | Tier 3 -- nice to have.
    VPVR:            PREFERRED | Visual use on TradingView only. NOT in engine.

---

## 4. Explicitly Rejected Indicators/Approaches

    Stochastic (8,3,3)    -- Harmful for breakout trading. Jan 2026
    MACD (6,13,5)         -- Redundant with ROC. Jan 2026
    CMF (20)              -- Too slow, redundant. Jan 2026
    Fixed % profit targets -- Replaced by structural swing targets. Jan 2026
    ATR-based stops        -- Replaced by structural swing stops. Jan 2026
    Pure scalping (Style A) -- Fixed 0.1-0.5% targets. Jan 2026
    Pure momentum (Style B) -- Riding trend indefinitely. Jan 2026
    BBW NONE = +1 both sides -- Non-directional padding. Removed v0.18
    Funding OK in Step 2    -- Double-counting. Removed v0.17
    No Adverse Liq as positive -- Non-directional padding. Removed v0.17
    Flat TRANSITIONAL (-2)  -- Replaced with ADX-proximity scale. Mar 2026

---

## 5. Risk Management Rules

    Position sizing:    Dynamic via ATR: Base x (20d AvgATR / CurrATR)
    Stop-loss:          STRUCTURAL — below/above previous swing low/high
    Take-profit:        STRUCTURAL — previous swing high (longs) / low (shorts)
    Hold through drawdown: Yes IF RSI(9) > 60 AND trend structure intact
    Overnight holding:  NEVER. Always flat at end of session.
    ATR thresholds:     Low < 80 | Normal 80-150 | High > 150 (calibrated for BTC ~$80k-$100k, Q1 2026)

---

## 6. Verdict Engine Design Preferences

    Minimum confidence to trade:  MEDIUM or HIGH (score >= 9)
    Regime preference:            TRENDING and RANGE_BOUND acceptable. TRANSITIONAL = caution.
    Score thresholds:             6/9/12 — feels correct as of Mar 2026
    False positive tolerance:     Low. Prefer NO TRADE over weak directional verdict.
    Display preference:           Clean, scannable. Headline verdict prominent. Score breakdown shown.

---

## 7. Key Design Decisions (Scorecard)

    v0.13  Partial signal upgrade system added
    v0.15  TRANSITIONAL regime penalty: ADX-proximity scale (-1 or -2) + TierFloor() guard
    v0.16  TierFloor() guard formalised (max 1 tier drop per penalty)
    v0.17  Non-directional padding cleanup; score denominator /13
    v0.18  BBW redesign: ACTIVE=-1 both, RELEASING=+1 ROC-aligned, NONE=no change

---

## 8. What This Trader Values in AI Collaboration

    Communication:      Technical and concise. No hand-holding. Correct terminology.
    Decision process:   Spec-first. Novel questions to strategy conversation (Perplexity).
                        Decisions documented in .md and committed before coding.
    GitHub workflow:    Proposal .md → strategy review → response .md committed → implementation.
                        All docs in /docs folder of DeribitVerdictEngine repo.
    Review preference:  Show what changed and why. Changelog for every version.
    Proactive flagging: Flag design issues before implementing. Don't silently conflict with spec.
    Push back when:     A change reintroduces deliberately removed patterns. Cite the version.
    Avoid:              Re-opening settled decisions without new data. Increasing indicator correlation.
    Conversation split: Strategy/spec decisions → Perplexity. Implementation/debugging → Claude.
