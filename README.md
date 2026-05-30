# tradingview-mcp-thaqif

**Improved TradingView MCP rules for Pine Script development, FCPO/futures trading indicators, and AI-assisted trading systems.**

A structured `rules.json` ruleset that you load as project context for an AI coding assistant (Claude + the TradingView MCP). It encodes how I want Pine Script v5/v6 written, how multi-timeframe and Wyckoff analysis should be applied to FCPO (Bursa Malaysia Crude Palm Oil Futures) and crude oil, and how the AI should drive the TradingView MCP tools safely.

This is an improved, reorganized successor to the single-strategy `rules.json` in [`LewisWJackson/tradingview-mcp-jackson`](https://github.com/LewisWJackson/tradingview-mcp-jackson). See [CHANGELOG.md](CHANGELOG.md) for what changed.

---

## Why this exists

The original `rules.json` was a flat config for one demo strategy (a 10-second BTCUSDT momentum scalper). It described *what one bot does*, not *how indicators should be built*.

This version turns the file into a reusable **development ruleset**: 64 actionable rules across 9 categories that an AI assistant treats as hard constraints when generating or reviewing Pine Script and when operating the TradingView MCP. The original scalper is preserved verbatim under `strategy_profiles` so nothing is lost.

---

## Quick start: integrating with TradingView MCP

1. **Clone the repo** next to your TradingView MCP project:

   ```bash
   git clone https://github.com/TZ24cloud/tradingview-mcp-thaqif.git
   ```

2. **Point your assistant at the rules.** With Claude + the TradingView MCP, reference the file as project context (for example, drop it in the project root, or copy the relevant block into your `CLAUDE.md` / system rules). A simple instruction works well:

   > "Follow `rules.json`. Treat every rule with severity `critical` or `high` as a hard constraint. When you write or review Pine Script, cite the rule IDs you applied."

3. **Let the rules gate the MCP workflow.** The `mcp_tools` category mirrors the real TradingView MCP tools, so the assistant knows to call `chart_get_state` first, compile with `pine_smart_compile` then `pine_get_errors`, use full indicator names, and never auto-execute live orders.

4. **Customize** the `trading_context` and any thresholds to your own account, then commit. See [Customizing](#customizing-for-your-own-style).

---

## File structure

```
tradingview-mcp-thaqif/
  rules.json                    # the ruleset (load this as AI context)
  README.md                     # this file
  CHANGELOG.md                  # improvements over the v1 original
  LICENSE                       # MIT
  .gitignore
  examples/
    fcpo_wyckoff_mtf.pine       # starter Pine v6 indicator that obeys these rules
    README.md                   # what the example demonstrates
```

### Rule format

JSON has no native comments, so each rule is an object carrying its own explanation:

```json
{
  "id": "mtf-002",
  "rule": "Retrieve non-repainting HTF values with request.security(sym, htf, expr[1], lookahead=barmerge.lookahead_on).",
  "rationale": "Using expr[1] with lookahead_on returns the last CLOSED HTF value on every bar - no future leak, no repaint.",
  "severity": "critical",
  "example": "f_htf(_src,_tf) => request.security(syminfo.tickerid,_tf,_src[1],lookahead=barmerge.lookahead_on)"
}
```

`severity` is one of `critical`, `high`, `medium`, `low`. Treat `critical`/`high` as non-negotiable.

---

## Rule categories explained

| Category | Rules | What it governs |
|---|---|---|
| `pine_script` | 12 | Core v5/v6 syntax, namespacing, state, no-repaint declarations, file layout |
| `indicators` | 7 | Indicator selection, ATR stops, session VWAP, divergence on confirmed pivots |
| `multi_timeframe` | 5 | Top-down Daily->H4->H1->M15 alignment and non-repainting HTF retrieval |
| `wyckoff` | 7 | Phase labeling and event detection (Spring, UTAD, SC/BC, price-volume table) |
| `fcpo` | 6 | FCPO session gating, contract maths (RM25/point), inter-market context |
| `backtesting` | 7 | Honest backtests: fill timing, costs, sample size, out-of-sample validation |
| `alerts` | 5 | Confirmed-bar alerts, JSON payloads for webhooks, de-duplication |
| `risk_management` | 5 | Fractional risk sizing, structural stops, daily loss limits |
| `mcp_tools` | 10 | Safe, correct use of the TradingView MCP tools |

### `pine_script` — example

Rule `ps-003` forbids un-namespaced built-ins:

```pine
// anti-pattern (v3/v4, removed in v5/v6 -> will not compile)
r = rsi(close, 14)

// correct (v5/v6)
r = ta.rsi(close, 14)
```

### `multi_timeframe` — example

The non-repaint HTF pattern from `mtf-002`, used to read a confirmed Daily/H4 trend on an M15 chart:

```pine
f_htf(_src, _tf) => request.security(syminfo.tickerid, _tf, _src[1], lookahead=barmerge.lookahead_on)
htfTrend = f_htf(ta.ema(close, 50), "240")   // confirmed H4 EMA, no repaint
```

### `wyckoff` — example

Spring detection from `wyk-002` (a best-quality long trigger):

```pine
spring = low < support and close > support and ta.highest(volume, 20) == volume
```

### `fcpo` — example

Session gating from `fcpo-001` and contract maths from `fcpo-002`:

```pine
inSess = not na(time(timeframe.period, "1030-1230,1430-1800", "Asia/Kuala_Lumpur"))
riskRM = math.abs(entry - stop) * 25   // RM per 1 lot (25 tons * RM1/ton)
```

### `backtesting` — example

Honest fill timing and costs from `bt-001`/`bt-002`:

```pine
strategy("FCPO Wyckoff", overlay=true, process_orders_on_close=true,
         calc_on_every_tick=false, commission_type=strategy.commission.cash_per_order,
         commission_value=8.0, slippage=2)
```

### `alerts` — example

Confirmed-bar, automation-ready alert from `alr-001`/`alr-002`:

```pine
if longSignal and barstate.isconfirmed
    msg = '{"sym":"' + syminfo.ticker + '","tf":"' + timeframe.period + '","side":"long","px":' + str.tostring(close) + '}'
    alert(msg, alert.freq_once_per_bar_close)
```

### `mcp_tools` — example

The safe edit loop from `mcp-005`: inject source, compile, then check errors.

```
pine_set_source  ->  pine_smart_compile  ->  pine_get_errors  (->  pine_get_console)
```

And `mcp-010`: surface signals, but never auto-execute live orders or move funds.

---

## Examples

[`examples/fcpo_wyckoff_mtf.pine`](examples/fcpo_wyckoff_mtf.pine) is a starter Pine v6 indicator that puts the ruleset into practice on FCPO. It defines a trading range, detects Wyckoff **Spring** and **UTAD** events with climactic-volume confirmation, filters by a non-repainting higher-timeframe EMA bias, gates signals to Bursa FCPO session hours, derives ATR-based protective stops, emits confirmed-bar JSON alerts for webhook/MCP automation, and prints a status table the MCP can read. Every block in the script header cites the rule IDs it satisfies.

Validate it against a live chart with the TradingView MCP using the `mcp-005` loop: `pine_set_source` -> `pine_smart_compile` -> `pine_get_errors`. See [`examples/README.md`](examples/README.md) for details.

---

## My usage context

These rules are tuned to how I actually trade and build:

- **Pine Script v5/v6.** Targeting v6 (request.* in local scopes, value-returning `if`/`switch`, cleaner `na`/type handling) while keeping v5-compatible patterns flagged.
- **FCPO futures (Bursa Malaysia Crude Palm Oil)** and crude oil (WTI/Brent), plus BTCUSDT from the legacy demo. FCPO specifics modeled: two Asian day sessions (GMT+8) with a 12:30-14:30 lunch gap, 25 metric tons/contract, RM25/point, and inter-market drivers (Dalian palm olein, CBOT soybean oil, Brent/WTI, USD/MYR, MPOB inventory/export data).
- **Multi-timeframe top-down:** Daily bias -> H4 structure -> H1 zone -> M15 trigger, with non-repainting HTF retrieval throughout.
- **Wyckoff methodology:** accumulation/markup/distribution/markdown phases; events PS, SC, AR, ST, Spring, SOS, LPS, UTAD, SOW, LPSY; the Effort-vs-Result price-volume table; composite-man framing. Best longs at Spring/LPS, best shorts at UTAD/LPSY, stops at structural invalidation.
- **Backtesting systems:** realistic fills and costs, 100+ trade samples, walk-forward / out-of-sample validation, correct FCPO contract multiplier in P&L.

---

## Customizing for your own style

`rules.json` is meant to be forked and edited. Suggested changes:

1. **Swap the instruments.** Edit `trading_context.primary_instruments`, `analysis_timeframes`, and `default_signal_timeframe` to your markets. The `fcpo` category can be removed or replaced with your own instrument block.
2. **Retune thresholds.** RSI band (`ind-001`), minimum R:R (`risk-002`, default 1.5R), risk fraction (`risk-001`, default 1%), daily loss limit and max trades (`risk-003`), and minimum backtest sample (`bt-004`) are all explicit and easy to change.
3. **Add or drop categories.** Keep the object shape `{ "description": "...", "rules": [ ... ] }` and give each rule a unique `id`, a `rationale`, and a `severity`. Add an `example`/`anti_example` where a code snippet clarifies intent.
4. **Promote/demote severity.** If you want the assistant to be stricter, raise rules to `high`/`critical`; relax to `medium`/`low` for preferences rather than hard rules.
5. **Keep your strategies under `strategy_profiles`.** Store reusable named configs there (the legacy scalper is one example) so the ruleset and your concrete setups live together.

---

## Attribution and disclaimer

Based on the `rules.json` concept from [`LewisWJackson/tradingview-mcp-jackson`](https://github.com/LewisWJackson/tradingview-mcp-jackson). The legacy 10-second scalper config originates there and is retained for reference.

This repository is for research and educational use. Nothing here is financial advice. Trading FCPO, crude oil, and crypto futures involves substantial risk of loss. The `mcp_tools` rules deliberately forbid AI-driven live execution — surface signals and place trades yourself.

---

## License

Released under the [MIT License](LICENSE). Copyright (c) 2026 Thaqif (TZ24cloud).
