# DeribitVerdictEngine — Project Overview & Roadmap
**Repo:** https://github.com/Beansz2015/DeribitVerdictEngine  
**Language:** VB.NET / .NET 8 / Windows Forms  
**Last updated:** 2026-04-11 | **App version:** Commit 5 (v0.49 base)

> This file is the **strategic overview and roadmap** for the project.
> It covers what the project is and where it is going.
>
> For technical detail, see the engine repo docs:
> - `DeribitVerdictEngine/docs/DeribitIndicatorProject.md` — authoritative handover (file versions, indicator map, scoring logic, version history)
> - `DeribitVerdictEngine/docs/architecture.md` — codebase structure, data flow, design decisions
> - `DeribitVerdictEngine/docs/trader-profile.md` — trader style, preferences, collaboration rules

---

## Purpose

A Windows Forms desktop app that connects live to the Deribit REST API, computes a
battery of technical indicators on BTC-PERPETUAL, scores them through a weighted
multi-tier engine, and emits a directional verdict (STRONG LONG → STRONG SHORT) with
ATR-derived entry / stop / target levels. Designed as a discretionary scalping aid
on 1-minute charts for a momentum-informed hybrid trading style.

All indicator thresholds, scoring weights, and gate parameters are externalised to
`settings.json` — no recompile needed for tuning. The engine is fully operational
at Commit 5.

---

## Strategic Roadmap

### Auto-Tuning (Next major milestone)
The `AnalysisLogger` CSV accumulates verdict + signal data every run. Once the
Calibration Readiness Report reaches READY (≥300 rows, ≥3 sessions, ≥3 regimes,
≥2 liquidation events), the plan is to build an auto-tuning pass that:
- Reads the CSV
- Correlates each signal's vote with subsequent price direction
- Adjusts scoring weights and gate thresholds in `settings.json` automatically
- Eliminates manual backtesting and makes the engine self-calibrating

This requires the settings-driven architecture to be complete (done as of v0.49 / Commit 5).

### Signal Cross-Confirm Upgrades (Future)
- **OI × CVD cross-confirm** — OI (NEW LONGS/SHORTS) and CVD direction are currently
  scored independently. A combined multiplier (NEW LONGS + CVD RISING = full score)
  would reduce double-counting and reward genuine trend confirmation more precisely.
- **Funding momentum** — absolute funding rate currently used as a contrarian modifier.
  Rising vs falling rate direction is a higher-quality signal and has not yet been
  implemented.

### Infrastructure (Future)
- **Websocket upgrade** — engine currently uses REST polling (snapshot-based).
  Moving to Deribit websocket API would provide a real-time order book and trade
  stream, removing the fundamental REST latency constraint. Most impactful
  non-code upgrade available.
- **AWS London (LD4) deployment** — recommended for minimal latency to Deribit API.
  Not yet confirmed as deployment target.

---

## Update Protocol

At the end of every session that changes code or plans:
1. Update `DeribitVerdictEngine/docs/DeribitIndicatorProject.md` (technical handover + version history)
2. Update `DeribitVerdictEngine/docs/architecture.md` if files were added, moved, or renamed
3. Update this file only if the strategic roadmap changes
