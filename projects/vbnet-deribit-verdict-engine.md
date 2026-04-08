# DeribitVerdictEngine — Project Documentation
**Repo:** https://github.com/Beansz2015/DeribitVerdictEngine  
**Language:** VB.NET / .NET 8 / Windows Forms  
**Last updated:** 2026-04-08 | **App version:** v0.36

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
- [x] Deribit REST client (`DeribitClient.vb`) — candles (1m×250, 5m×210, 15m×100), funding,
      book summary, order book depth-10, recent trades
- [x] Strongly-typed settings system (`EngineSettings.vb` + `SettingsLoader.vb`);
      all tunable parameters externalised to `settings.json`
- [x] `DynamicNorms` — live ATR/Vol/VWAP thresholds recomputed each run from last 250 candles
- [x] `AnalysisLogger` — CSV logging of every run; Calibration Readiness Report
      (checks ≥300 rows, ≥3 sessions, ≥3 regimes, ≥2 liquidation events)

### Indicators (`Indicators.vb` v0.36)
- [x] ROC(9) — lookback from settings
- [x] RSI(9) — period from settings
- [x] RSI divergence — price gate + RSI delta gate from settings; displayed in signal breakdown
- [x] DMI/ADX on 5m candles — period from settings
- [x] Volume SMA with DynamicNorms threshold (H/M tiers)
- [x] VWAP deviation with DynamicNorms threshold; dual-session support
      (daily 00:00 UTC / US session 13:30 UTC); session boundary times fully
      configurable in settings.json; warmup candle threshold from settings;
      σ1/σ2 sigma bands computed and displayed
- [x] BBW Squeeze (period 20, StdDev 2.0) — ACTIVE / RELEASING / NONE
- [x] TTM Squeeze momentum histogram — direction (RISING/FALLING) and signal
      (BULL_BUILDING / BULL_FADING / BEAR_BUILDING / BEAR_FADING) displayed in UI
- [x] EMA Ribbon (9/21/50 on 1m) — BULL / BEAR / MIXED; 5m EMA(200) regime anchor
- [x] OFI — bid/ask top-3 order book level imbalance, weights 3/2/1;
      displays weighted bid vol, ask vol, ratio, and BUY/SELL/NEUTRAL signal
- [x] Liquidations — large threshold from settings; penalty-only signal
- [x] CVD (Cumulative Volume Delta) — net delta + slope (RISING/FALLING/FLAT) +
      divergence flag (BULLISH / BEARISH / NONE); all gates from settings
- [x] Donchian(20) — LONG / SHORT / NONE breakout signal
- [x] OBV — trend gate + divergence gate from settings
- [x] OI ring buffer — 15m + 60m delta; NEW LONGS / SHORTS / COVERING / CAPITULATION / NEUTRAL
- [x] Multi-timeframe confluence gate (CalcMTFGate) — 15m DMI/ADX + EMA alignment;
      outputs MTFGatePass (bool) + MTFGateReason (string); all params from MTFGateSettings

### Scoring Engine (`ScoringEngine.vb` v0.32)
- [x] Dual long/short score architecture with regime-aware MaxScore
      (TRENDING=17, RANGE_BOUND=16, TRANSITIONAL=13)
- [x] Weighted scoring — each signal category contributes a defined max score;
      partial scores awarded when only one side of a cross-category confirm exists;
      full score awarded on confirmed alignment (e.g. RSI partial + EMA partial → full)
- [x] Verdict thresholds (Strong/Med/Weak) as % of MaxScore — from settings
- [x] Funding rate modifier (high/low positive/negative gates from settings)
- [x] Regime veto for TRENDING (blocks counter-trend signals)
- [x] TRANSITIONAL ADX penalty (tiered, from settings)
- [x] CVD divergence penalty (−1 applied before liquidation penalty)
- [x] Liquidation penalty (−1 standard, −2 large)
- [x] MTF gate veto — if proposed LONG/SHORT fails 15m confluence gate, verdict
      forced to NO TRADE; gate reason appended to signal breakdown
- [x] OBV scoring fix — aligned trend with non-adverse divergence receives full score;
      adverse divergence remains partial-upgrade only
- [x] CalcHoldStatus for open position guidance
- [x] All thresholds and gates fully wired to `EngineSettings` / `settings.json`

### Settings (`EngineSettings.vb` v0.33 + `settings.json`)
- [x] `VwapSettings` class: `Session1StartHour/Minute`, `Session2StartHour/Minute`,
      `WarmupCandles`, `DevThresholdPct`
- [x] `CvdSettings` class: `SlopeMinUsd`, `SlopePctOfValue`, `DivergencePriceGate`, `TradeLookback`
- [x] `MTFGateSettings` class: `Enabled`, `DmiPeriod`, `RequiredConfirms`, `CandleCount`
- [x] All indicator periods, gates, and scoring weights in settings.json

### UI (`MainForm.vb` v0.36)
- [x] Async analysis loop with colour-coded verdict label
- [x] Full tiered output display (CORE → TIER 1 → TIER 2 → TIER 3 → SCORING)
- [x] ATR entry/stop/target block — ATR value, scale factor, long and short
      stop/entry/target prices, and R:R ratio displayed on every run
- [x] VWAP display: value, dev%, session candle count, σ1/σ2 sigma bands,
      session label, [WARMUP] tag if below warmup threshold
- [x] TTM Squeeze direction and signal displayed in BBW/TTM block
- [x] CVD display line in TIER 2 block (net, slope, divergence)
- [x] RSI divergence displayed in signal breakdown
- [x] OFI bid/ask weighted volumes and ratio displayed
- [x] MTF gate block: PASS/BLOCK status, 15m trend/ADX/EMA components, gate reason
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

### Multi-Timeframe Confluence Gate (Complete)
15m DMI/ADX + EMA alignment gate prevents low-timeframe signals from producing a
verdict when higher-timeframe structure conflicts. Gate is a hard veto (forces
NO TRADE), not a score modifier. Future calibration: consider tightening pass
conditions when 15m ADX is below trend-strength threshold (currently passes on
absence of conflict rather than presence of confirmation).

---

## Pending Observations / Calibration Watchlist

| # | Condition to observe | Status |
|---|---|---|
| 1 | **CVD divergence penalty:** See a live BEARISH or BULLISH divergence and confirm the −1 penalty fires correctly in the scoring breakdown | 🔍 WATCHING |
| 2 | **Transitional ADX penalty:** Run during a TRANSITIONAL regime and confirm tiered penalty is applied correctly | 🔍 WATCHING |
| 3 | **Calibration readiness:** Accumulate 300+ log rows across 3+ sessions to trigger READY FOR RECALIBRATION | 🔍 WATCHING |
| 4 | **MTF weak-15m pass:** Review whether PASS should require stricter confirmation when 15m ADX is below trend-strength threshold (currently passes on absence of bearish evidence) | 🔍 WATCHING |

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
| 2026-04-06 | v0.31: CalcVWAP captures sessionCandleCount via ByRef; CalcVWAPBands added (σ1/σ2); VWAP display shows bands, session candle count, warmup tag |
| 2026-04-07 | v0.32: VWAP session boundary times and warmup threshold moved from hardcoded values to settings.json; VwapSettings class expanded; CalcVWAP/CalcVWAPBands parameterised; MainForm reads session times from cfg.Indicators.VWAP |
| 2026-04-08 | v0.33–v0.36: MTF confluence gate added end-to-end (EngineSettings MTFGateSettings, CalcMTFGate in Indicators, MainForm 15m candle fetch + gate call + UI block, ScoringEngine MTF veto); OBV scoring fix (aligned+non-adverse = full score); build fixes for MTFGateSettings property names; docs updated to reflect all features including ATR display, TTM Squeeze direction, RSI divergence display, OFI bid/ask imbalance detail, and weighted scoring clarification |
