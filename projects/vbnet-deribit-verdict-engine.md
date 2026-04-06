# DeribitVerdictEngine — Project Documentation
**Repo:** https://github.com/Beansz2015/DeribitVerdictEngine  
**Language:** VB.NET / .NET 8 / Windows Forms  
**Last updated:** 2026-04-06 | **App version:** v0.30

> This file lives in `perplexity-best-practices/projects/` and is updated by the AI
> at the end of every session. The authoritative technical handover (file versions,
> architecture diagram, scoring logic) lives at:
> `DeribitVerdictEngine/docs/DeribitIndicatorProject.md`

---

## Purpose

A Windows Forms desktop app that connects live to the Deribit REST API, computes a
battery of technical indicators on BTC-PERPETUAL, scores them through a weighted
multi-tier engine, and emits a directional verdict (STRONG LONG → STRONG SHORT) with
ATR-derived entry / stop / target levels. Designed for use as a discretionary
scalping aid on 1-minute charts.

---

## Completed Features

### Core Engine
- [x] Deribit REST client (`DeribitClient.vb`) — candles (1m×250, 5m×210), funding,
      book summary, order book depth-10, recent trades
- [x] Strongly-typed settings system (`EngineSettings.vb` + `SettingsLoader.vb`);
      all tunable parameters externalised to `settings.json`
- [x] `DynamicNorms` — live ATR/Vol/VWAP thresholds recomputed each run from last 250 candles
- [x] `AnalysisLogger` — CSV logging of every run; Calibration Readiness Report
      (checks ≥300 rows, ≥3 sessions, ≥3 regimes, ≥2 liquidation events)

### Indicators (all in `Indicators.vb` v0.30)
- [x] ROC(9) — lookback from settings
- [x] RSI(9) — period from settings; RSI divergence (price gate + delta gate from settings)
- [x] DMI/ADX(9) on 5m candles
- [x] Volume SMA(9) with DynamicNorms threshold
- [x] VWAP deviation with DynamicNorms threshold
- [x] BBW Squeeze (period 20, StdDev 2.0) — ACTIVE / RELEASING / NONE
- [x] EMA Ribbon (9/21/50) — BULL / BEAR / MIXED; 5m EMA(200) regime anchor
- [x] OFI — top-3 order book levels, weights 3/2/1 — BUY / SELL / NEUTRAL
- [x] Liquidations — large threshold from settings; penalty-only signal
- [x] CVD (Cumulative Volume Delta) — net delta + slope (RISING/FALLING/FLAT) +
      divergence flag (BULLISH_DIV / BEARISH_DIV / NONE); divergence gate from settings
- [x] Donchian(20) — LONG / SHORT / NONE breakout signal
- [x] OBV — trend gate + divergence gate from settings
- [x] OI ring buffer — 15m + 60m delta; NEW LONGS / SHORTS / COVERING / CAPITULATION / NEUTRAL

### Scoring Engine (`ScoringEngine.vb` v0.27)
- [x] Dual long/short score architecture with regime-aware MaxScore
      (TRENDING=17, RANGE_BOUND=16, TRANSITIONAL=13)
- [x] Verdict thresholds (Strong/Med/Weak) as % of MaxScore — from settings
- [x] Cross-category partial upgrade logic (e.g. RSI partial + EMA partial → full)
- [x] Funding rate modifier (high/low positive/negative gates from settings)
- [x] Regime veto for TRENDING (blocks counter-trend signals)
- [x] TRANSITIONAL ADX penalty (tiered, from settings)
- [x] CVD divergence penalty (−1 applied before liquidation penalty)
- [x] Liquidation penalty (−1 standard, −2 large)
- [x] CalcHoldStatus for open position guidance
- [x] All thresholds and gates fully wired to `EngineSettings` / `settings.json`

### UI (`MainForm.vb` v0.30)
- [x] Async analysis loop with configurable interval
- [x] Colour-coded verdict label
- [x] Full tiered output display (CORE → TIER 1 → TIER 2 → TIER 3 → SCORING)
- [x] CVD display line in TIER 2 block
- [x] All indicator call sites pass settings-derived parameters
- [x] `SettingsLoader.Initialise()` called in constructor

---

## Plans & Architectural Decisions

### Auto-Tuning (Planned)
The `AnalysisLogger` CSV accumulates verdict + signal data every run. Once the
Calibration Readiness Report reaches READY (≥300 rows, ≥3 sessions, ≥3 regimes,
≥2 liquidation events), the plan is to build an auto-tuning pass that:
- Reads the CSV
- Correlates each signal's vote with subsequent price direction
- Adjusts scoring weights and gate thresholds in `settings.json` automatically
- This eliminates manual backtesting and makes the engine self-calibrating

### Settings-Driven Architecture (Complete)
All magic numbers removed from code. Every threshold, period, gate, and weight
flows from `settings.json` → `EngineSettings` → calculation methods. This is a
prerequisite for the auto-tuning pass.

### DynamicNorms (Complete)
Live adaptive norms replace static thresholds for Volume and VWAP. This means
the engine self-adjusts to current market volatility without needing manual
threshold changes.

---

## Pending Observations (Conditions Being Watched)

| # | Condition to observe | Status |
|---|---|---|
| 1 | **CVD divergence penalty:** See a real BEARISH or BULLISH divergence in live output and confirm the −1 penalty appears in the scoring breakdown | 🔍 WATCHING |
| 2 | **Transitional ADX penalty:** Run analysis during a TRANSITIONAL regime (ADX < threshold) and confirm the effective score is reduced by the correct tiered amount | 🔍 WATCHING |
| 3 | **Calibration readiness:** Accumulate 300+ log rows across 3+ sessions to trigger the READY FOR RECALIBRATION status in the Calibration Report | 🔍 WATCHING |

---

## Update Protocol (for AI sessions)

At the end of every session that changes code or plans, the AI must:
1. Update `DeribitVerdictEngine/docs/DeribitIndicatorProject.md` (technical handover)
2. Update this file (`perplexity-best-practices/projects/vbnet-deribit-verdict-engine.md`)
3. Move any completed items from Pending Observations to the Completed Features section
4. Add new Planned items to the Plans section when architectural decisions are made

---

## Session Log

| Date | Changes |
|---|---|
| 2026-04-06 | Full engine wired to settings.json; CVD fully integrated (Indicators + ScoringEngine + MainForm); all indicator call sites pass settings params; ScoringEngine.Calculate accepts cfg as 4th arg; handover doc created in DeribitVerdictEngine/docs/ |
