# DeribitVerdictEngine — Project Documentation
**Repo:** https://github.com/Beansz2015/DeribitVerdictEngine  
**Language:** VB.NET / .NET 8 / Windows Forms  
**Last updated:** 2026-04-09 | **App version:** v0.45

> This file lives in `perplexity-best-practices/projects/` and is updated by the AI
> at the end of every session. The authoritative technical handover (file versions,
> architecture diagram, scoring logic) lives at:
> `DeribitVerdictEngine/docs/DeribitIndicatorProject.md`
> A full codebase structure map lives at:
> `DeribitVerdictEngine/docs/architecture.md`

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
- [x] Deribit REST client (`DeribitClient.vb`) — candles (1m×250, 5m×210, 15m×70), funding,
      book summary, order book depth-10, recent trades (100)
- [x] Strongly-typed settings system (`EngineSettings.vb` + `SettingsLoader.vb`);
      all tunable parameters externalised to `settings.json`
- [x] `DynamicNorms` — live ATR/Vol/VWAP thresholds recomputed each run from last 250 candles
- [x] `AnalysisLogger` — CSV logging of every run; Calibration Readiness Report
      (checks ≥300 rows, ≥3 sessions, ≥3 regimes, ≥2 liquidation events)
- [x] `AutoRunTimer` — `IAutoRunTimer` interface + `WinFormsAutoRunTimer` implementation

### Indicators (`Core/Indicators/` partial classes)
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
- [x] Multi-timeframe confluence gate (`CalcMTFGate`) — 15m DMI/ADX + EMA alignment;
      outputs MTFGatePass (bool) + MTFGateReason (string); all params from MTFGateSettings
- [x] VPFR-lite (`CalcVPFRLite`) — volume-profile POC + HVN/LVN proximity signal;
      wired into scoring engine

### Scoring Engine (`Core/ScoringEngine_*.vb` partial classes, base v0.32)
- [x] Dual long/short score architecture with regime-aware MaxScore
      (TRENDING=17, RANGE_BOUND=16, TRANSITIONAL=13)
- [x] Weighted scoring — each signal category contributes a defined max score;
      partial scores awarded when only one side of a cross-category confirm exists;
      full score awarded on confirmed alignment
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
- [x] VPFR-lite scoring — HVN/LVN proximity scored for long/short confirmation
- [x] CalcHoldStatus for open position guidance
- [x] All thresholds and gates fully wired to `EngineSettings` / `settings.json`

### Settings (`Core/Settings/EngineSettings.vb` v0.33 + `settings.json`)
- [x] `VwapSettings` class: `Session1StartHour/Minute`, `Session2StartHour/Minute`,
      `WarmupCandles`, `DevThresholdPct`
- [x] `CvdSettings` class: `SlopeMinUsd`, `SlopePctOfValue`, `DivergencePriceGate`, `TradeLookback`
- [x] `MTFGateSettings` class: `Enabled`, `DmiPeriod`, `RequiredConfirms`, `CandleCount`
- [x] All indicator periods, gates, and scoring weights in settings.json

### UI (`UI/MainForm_*.vb` partial classes, v0.45)
- [x] Async analysis loop with colour-coded RTF verdict output (`RenderOutput` via `AppendRtf`)
- [x] Full tiered colour-coded output (CORE → TIER 1 → TIER 2 → TIER 3 → SCORING)
- [x] ATR entry/stop/target block — ATR value, scale factor, long and short
      stop/entry/target prices, and R:R ratio displayed on every run
- [x] **Last transacted price** line — pulled from `recentTrades(0).Price` and
      displayed above the ATR block (distinct from the ATR entry price which is candle close)
- [x] Run timestamp shown in UTC+8
- [x] VWAP display: value, dev%, session candle count, σ1/σ2 sigma bands,
      session label, [WARMUP] tag if below warmup threshold
- [x] TTM Squeeze direction and signal displayed in BBW/TTM block
- [x] CVD display line in TIER 2 block (net, slope, divergence)
- [x] RSI divergence displayed in signal breakdown
- [x] OFI bid/ask weighted volumes and ratio displayed
- [x] MTF gate block: PASS/BLOCK status, 15m trend/ADX/EMA components, gate reason
- [x] VPFR-lite block in TIER 3 output
- [x] Auto-run mode — configurable interval (NUD), Single/Repeat radio buttons,
      Start/Stop button with ▶/■ icon, countdown label; wired via `IAutoRunTimer`
- [x] NUD digit vertical centering fix (via `EM_SETRECT` / `SendMessage` on inner TextBox)
- [x] All indicator call sites pass settings-derived parameters
- [x] `SettingsLoader.Initialise()` called in constructor

### Codebase Refactor (v0.45)
- [x] **Full partial-class split** — monolithic `MainForm.vb`, `Indicators.vb`, `ScoringEngine.vb`
      replaced with scoped partial classes under `UI/`, `Core/`, `Core/Indicators/`
- [x] `ScoringEngine` split into `ScoringEngine_Types`, `_Helpers`, `_Calculate`
- [x] `IndicatorEngine` split into `_Types`, `_Core`, `_VWAP`, `_OrderFlow`, `_Structure`
- [x] `MainForm` split into `MainForm_Layout`, `_AutoRun`, `_Analysis`, `_Render`
- [x] All partial files compile clean; BC40003 shadow warning suppressed via rename
- [x] `docs/architecture.md` created — full directory tree, data flow diagram,
      partial class responsibility table, key design decisions

---

## Plans & Architectural Decisions

### Auto-Tuning (Planned)
The `AnalysisLogger` CSV accumulates verdict + signal data every run. Once the
Calibration Readiness Report reaches READY (≥300 rows, ≥3 sessions, ≥3 regimes,
≥2 liquidation events), the plan is to build an auto-tuning pass that:
- Reads the CSV
- Correlates each signal's vote with subsequent price direction
- Adjusts scoring weights and gate thresholds in `settings.json` automatically
- Eliminates manual backtesting and makes the engine self-calibrating

### Settings-Driven Architecture (Complete)
All magic numbers removed from code. Every threshold, period, gate, and weight
flows from `settings.json` → `EngineSettings` → calculation methods. This is a
prerequisite for the auto-tuning pass.

### DynamicNorms (Complete)
Live adaptive norms replace static thresholds for Volume and VWAP. The engine
self-adjusts to current market volatility without needing manual threshold changes.

### Multi-Timeframe Confluence Gate (Complete)
15m DMI/ADX + EMA alignment gate prevents low-timeframe signals from producing a
verdict when higher-timeframe structure conflicts. Gate is a hard veto (forces
NO TRADE), not a score modifier. Future calibration: consider tightening pass
conditions when 15m ADX is below trend-strength threshold.

### Partial-Class Architecture (Complete)
Monolithic files split into single-responsibility partials. Each file under ~200 lines.
See `docs/architecture.md` for full mapping.

---

## Pending Observations / Calibration Watchlist

| # | Condition to observe | Status |
|---|---|---|
| 1 | **CVD divergence penalty:** See a live BEARISH or BULLISH divergence and confirm the −1 penalty fires correctly in the scoring breakdown | 🔍 WATCHING |
| 2 | **Transitional ADX penalty:** Run during a TRANSITIONAL regime and confirm tiered penalty is applied correctly | 🔍 WATCHING |
| 3 | **Calibration readiness:** Accumulate 300+ log rows across 3+ sessions to trigger READY FOR RECALIBRATION | 🔍 WATCHING |
| 4 | **MTF weak-15m pass:** Review whether PASS should require stricter confirmation when 15m ADX is below trend-strength threshold | 🔍 WATCHING |
| 5 | **VPFR-lite signal:** Observe live NEAR_HVN_SUPPORT / NEAR_HVN_RESIST / IN_LVN_BULL / IN_LVN_BEAR signals and confirm scoring and display correct | 🔍 WATCHING |

---

## Update Protocol (for AI sessions)

At the end of every session that changes code or plans, the AI must:
1. Update `DeribitVerdictEngine/docs/DeribitIndicatorProject.md` (technical handover)
2. Update `DeribitVerdictEngine/docs/architecture.md` if files were added/moved/renamed
3. Update this file (`perplexity-best-practices/projects/vbnet-deribit-verdict-engine.md`)
4. Move any completed items from Pending Observations to the Completed Features section
5. Add new Planned items to the Plans section when architectural decisions are made

---

## Session Log

| Date | Changes |
|---|---|
| 2026-04-06 | Full engine wired to settings.json; CVD fully integrated; all indicator call sites pass settings params; ScoringEngine.Calculate accepts cfg as 4th arg; handover doc created |
| 2026-04-06 | v0.31: CalcVWAP ByRef sessionCandleCount; CalcVWAPBands σ1/σ2 |
| 2026-04-07 | v0.32: VWAP session boundary times moved to settings.json; VwapSettings expanded |
| 2026-04-08 | v0.33–v0.36: MTF confluence gate end-to-end; OBV scoring fix; build fixes; docs updated |
| 2026-04-08 | v0.37: VolumeUSD field rename. v0.38: Auto-run feature (interval NUD, Single/Repeat, Start/Stop, countdown). v0.38a: SettingsLoader.Save fix. v0.39: 6 UI bug fixes |
| 2026-04-09 | v0.40: InitAutoRunControls forced False on load; RenderOutput rewritten to RTF colour-coded output. v0.41: NUD digit vertical centering via EM_SETRECT. v0.42: Last transacted price line (recentTrades(0).Price); timestamp to UTC+8. v0.43–v0.44: CalcVPFRLite implemented, wired, and call-site fix. v0.45: OnFormHandleCreated rename (BC40003 fix) |
| 2026-04-09 | **v0.45 refactor:** Full partial-class split — ScoringEngine → Core/ScoringEngine_Types/_Helpers/_Calculate; IndicatorEngine → Core/Indicators/IndicatorEngine_Types/_Core/_VWAP/_OrderFlow/_Structure; MainForm → UI/MainForm_Layout/_AutoRun/_Analysis/_Render. All monolithic root files deleted. New docs/architecture.md created. Both project docs updated. |
