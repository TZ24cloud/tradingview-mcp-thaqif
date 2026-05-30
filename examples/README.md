# Examples

Worked Pine Script examples that obey the rules in [`../rules.json`](../rules.json).

## `fcpo_wyckoff_mtf.pine`

A starter **Pine v6** indicator for FCPO (and any futures/crypto symbol). It demonstrates the ruleset end to end:

- **Range definition** — rolling support/resistance from the prior-bar window (`ps-006`, no current-bar leak).
- **Wyckoff Spring & UTAD** detection (`wyk-002`, `wyk-003`) with **climactic-volume** confirmation (Effort vs Result, `wyk-005` / `ind-002`).
- **Non-repainting HTF bias** via `request.security(sym, tf, expr[1], lookahead=barmerge.lookahead_on)` (`ps-009` / `mtf-002`); signals are only taken in the higher-timeframe direction (`mtf-001`).
- **FCPO session gating** to the Bursa day sessions with the lunch break excluded (`fcpo-001`).
- **ATR-based protective stops** anchored to structural invalidation (`ind-003` / `risk-004`).
- **Confirmed-bar JSON alerts** for webhook/MCP automation, no secrets in the payload (`alr-001`, `alr-002`, `alr-005`).
- **Titled plots** and a **status table** readable by the MCP (`ps-012`, `mcp-004`).

### How to load and validate

1. Open Pine Editor in TradingView and paste the script, **or** use the TradingView MCP:
   - `pine_new` -> `pine_set_source` -> `pine_smart_compile` -> `pine_get_errors` (the `mcp-005` loop).
2. Add it to an `MYX:FCPO1!` chart (M15 recommended) with an H4/Daily HTF bias setting.
3. Tune `rangeLen`, `volMult`, the HTF timeframe, and ATR multiple to your instrument and style (see "Customizing" in the main README).

> Educational only. Not financial advice. Surface signals and place trades yourself (`mcp-010`).
