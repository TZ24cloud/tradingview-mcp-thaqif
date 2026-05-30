# Changelog

All notable changes to `rules.json` are documented here. This project adapts and expands the original `rules.json` from [`LewisWJackson/tradingview-mcp-jackson`](https://github.com/LewisWJackson/tradingview-mcp-jackson).

The format follows [Keep a Changelog](https://keepachangelog.com/), and the ruleset uses [Semantic Versioning](https://semver.org/).

---

## [2.0.1] - 2026-05-30

### Added
- Published to the official MCP Registry as io.github.TZ24cloud/tradingview-mcp-thaqif.
- Added mcpName in package.json and a registry server.json for package-ownership verification.

### Changed
- Trimmed the registry description to the 100-character limit.

## [2.0.0] - 2026-05-30

Complete restructure from a single-strategy config into a reusable Pine Script / FCPO development ruleset for AI-assisted trading systems. This is a breaking change to the file shape (new top-level keys), hence the major version bump.

### Added

- **`meta` block** — name, version, author, source attribution, target Pine version (v6, v5-compatible), a description of the rule object format, and how-to-use guidance.
- **`trading_context` block** — primary instruments (FCPO, WTI/Brent, BTCUSDT), methodology stack (Wyckoff, multi-timeframe, Effort-vs-Result, Fibonacci confluence), analysis timeframes, FCPO external drivers (Dalian palm olein, CBOT soybean oil, Brent/WTI, USD/MYR, MPOB data), and FCPO session map in MYT/GMT+8.
- **9 rule categories with 64 actionable rules**, each with `id`, `rule`, `rationale`, `severity`, and `example`/`anti_example` where useful:
  - `pine_script` (12) — version declaration, single declaration statement, mandatory namespacing (`ta.*`/`math.*`/`request.*`), typed inputs, `var` state, na/zero guards, drawing/bars-back limits, no-repaint `request.security` defaults, v6 `if`/`switch` expressions, titled plots.
  - `indicators` (7) — RSI as filter not trigger, volume confirmation, ATR-based stops, session-anchored VWAP, no correlated-oscillator stacking, confirmed-pivot divergence, machine-readable plot output.
  - `multi_timeframe` (5) — top-down Daily->H4->H1->M15 alignment, the non-repaint HTF pattern (`expr[1]` + `lookahead_on`), timeframe validation, Fibonacci+Wyckoff confluence, `request.security_lower_tf` for intrabar detail.
  - `wyckoff` (7) — phase labeling, Spring and UTAD detection, climax (SC/BC) + AR classification, the price-volume truth table, composite-man framing, structural stop placement.
  - `fcpo` (6) — session gating, real contract maths (25 tons, RM25/point), inter-market context pulls, MPOB/export-survey volatility windows, USD/MYR macro filter, lunch-break gap handling.
  - `backtesting` (7) — `process_orders_on_close`, realistic commission/slippage, `calc_on_every_tick` discipline, minimum sample size, out-of-sample/walk-forward validation, correct contract multiplier, session-constrained fills.
  - `alerts` (5) — confirmed-bar firing, JSON payloads for webhook/MCP automation, `alertcondition` vs `alert()` separation, de-duplication, no secrets in messages.
  - `risk_management` (5) — fractional risk sizing, pre-defined stop/target with minimum R:R, daily loss and trade caps, structural (not arbitrary) stops, notional caps for time-based scalps.
  - `mcp_tools` (10) — `chart_get_state` first, full indicator names, `summary=true` for OHLCV, `study_filter` for Pine drawing reads, the `pine_set_source -> pine_smart_compile -> pine_get_errors` edit loop, avoiding huge `pine_get_source` dumps, targeted screenshots, replay/batch validation, alert lifecycle hygiene, and a hard no on AI-driven live execution.
- **`strategy_profiles` block** — named, reusable strategy configs.
- **README.md** — overview, MCP integration steps, category reference table with code examples, usage context, and a customization guide.
- **CHANGELOG.md** — this file.
- **Server bundled (fork assembly)** — the MIT TradingView MCP server (Chrome DevTools Protocol bridge to TradingView Desktop) is now included (`src/`, `scripts/`, `tests/`), making this repo an installable MCP, not just a ruleset.
- **Packaging & distribution** — npm metadata for `npx` install, a Claude Desktop `.mcpb` manifest (`manifest.json`), an official MCP Registry entry (`server.json`), plus `NOTICE`, `SECURITY.md`, and a `DISTRIBUTION.md` launch checklist.
- **docs/RULESET.md** — the detailed ruleset guide, relocated from the original README (which is now the server's front page).

### Changed

- **Purpose:** from "config for one demo bot" to "development ruleset / AI project context."
- **Instrument focus:** from BTCUSDT-only to FCPO and crude oil first, with crypto retained as a demo profile.
- **Rule expression:** vague single-line strings replaced with structured, testable rules that carry rationale, severity, and runnable Pine examples.
- **No-repaint posture:** repainting risk is now addressed explicitly across `pine_script`, `multi_timeframe`, `backtesting`, and `alerts` (the v1 scalper relied on same-tick evaluation with no repaint guidance).

### Preserved (from v1)

- The original **10-second momentum scalper** is kept verbatim in intent under `strategy_profiles.ten_second_momentum_scalper`: BTCUSDT watchlist, 1-minute signal source, RSI 30-70 filter, price-momentum entries, 10-second time-based exit, and the 1%/$3/6-trades-per-run risk caps. It is flagged demo-only and superseded by the Wyckoff/MTF methodology for real analysis.
- The spirit of the v1 `risk_rules` (fractional sizing, max trades per run) is generalized into the `risk_management` category.

### Migration notes

- Consumers that read v1 top-level keys (`watchlist`, `default_timeframe`, `entry_rules`, `exit_rules`, `risk_rules`, `notes`) should now read them from `strategy_profiles.ten_second_momentum_scalper`.
- New consumers should read bias/standards from `categories.*` and instrument settings from `trading_context`.

---

## [1.0.0] - prior (upstream)

Original `rules.json` from `LewisWJackson/tradingview-mcp-jackson`: a flat configuration for the "10-Second Momentum Scalper" demo (watchlist, default timeframe, strategy, indicators, bias criteria, entry/exit/risk rules, notes).
