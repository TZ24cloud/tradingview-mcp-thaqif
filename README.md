# tradingview-mcp-thaqif

**Drive TradingView Desktop from Claude (and other MCP clients) for Pine Script v5/v6, FCPO/futures, and Wyckoff multi-timeframe analysis.**

A local MCP server that bridges TradingView Desktop to an AI assistant over the Chrome DevTools Protocol, shipped with a 64-rule `rules.json` that constrains how the AI writes Pine and reads the market. MIT-licensed, runs entirely on your machine.

> Improved fork of [tradingview-mcp-jackson](https://github.com/LewisWJackson/tradingview-mcp-jackson) by Lewis W. Jackson, itself built on the original [tradingview-mcp](https://github.com/tradesdontlie/tradingview-mcp) by [@tradesdontlie](https://github.com/tradesdontlie). Full credit to both — see [NOTICE](NOTICE).

> [!WARNING]
> **Not affiliated with TradingView Inc. or Anthropic.** This tool connects to the TradingView Desktop app already running on your machine via the Chrome DevTools Protocol. It **requires your own valid TradingView subscription** and does not bypass any paywall. It surfaces analysis only — **it does not place trades**. Read the [Disclaimer](#disclaimer) and [SECURITY.md](SECURITY.md) before use.

> [!NOTE]
> **All processing is local.** No TradingView data leaves your machine; the server has no telemetry.

---

## What it does

- **Read your chart** — symbol, timeframe, indicator values, and custom Pine output (lines, labels, tables, boxes).
- **Develop Pine Script v5/v6** — inject source, smart-compile, and read errors in a loop.
- **Manage alerts, replays, and screenshots.**
- **`morning_brief`** — scan a watchlist, read every indicator, and return structured data so the model produces a Wyckoff multi-timeframe bias, driven by your `rules.json`.

## What's new in this fork

| Feature | What it does |
|---------|-------------|
| `rules.json` v2.0.0 | A 64-rule engine across 9 categories (Pine v5/v6, indicators, multi-timeframe, Wyckoff, FCPO, backtesting, alerts, risk, MCP tools). See [docs/RULESET.md](docs/RULESET.md). |
| `examples/fcpo_wyckoff_mtf.pine` | A worked, rule-compliant Pine v6 indicator (Spring/UTAD, non-repaint HTF, FCPO sessions). |
| npm + `.mcpb` packaging | Install via `npx`, or one-click in Claude Desktop via a Desktop Extension bundle. |
| Inherited from the jackson fork | `morning_brief`, `session_save`/`session_get`, `tv` CLI, and the `tv_launch` fix for TradingView Desktop v2.14+. |

## Requirements

- **Node.js 18+**
- **TradingView Desktop running with the Chrome DevTools Protocol enabled** (see [SETUP_GUIDE.md](SETUP_GUIDE.md))
- **A valid TradingView subscription**

## Install

> Full per-client snippets (Cursor, Cline, VS Code, local clone) are in the install section below.

**Claude Desktop — one click:** install the `.mcpb` from Settings -> Extensions, or build it from [`manifest.json`](manifest.json) (see [DISTRIBUTION.md](DISTRIBUTION.md)).

**Claude Desktop — config file** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "tradingview": {
      "command": "npx",
      "args": ["-y", "tradingview-mcp-thaqif"]
    }
  }
}
```

**Cursor** (`~/.cursor/mcp.json`), **Cline** (`cline_mcp_settings.json`), and **VS Code** (`.vscode/mcp.json`) use the same `command`/`args`. Run from a local clone with `"command": "node", "args": ["/abs/path/tradingview-mcp-thaqif/src/server.js"]`.

Then, with TradingView Desktop open, verify with the `tv_health_check` tool.

## The rules engine

`rules.json` is loaded as AI context so the assistant treats `critical`/`high` rules as hard constraints. The 9 categories: `pine_script`, `indicators`, `multi_timeframe`, `wyckoff`, `fcpo`, `backtesting`, `alerts`, `risk_management`, `mcp_tools`. Each rule has an `id`, `rationale`, `severity`, and a runnable example. Full guide: [docs/RULESET.md](docs/RULESET.md). Worked example: [examples/](examples/).

## Publishing / distribution

This repo is packaged for npm, the [official MCP Registry](https://registry.modelcontextprotocol.io/) (`server.json`), and a Claude `.mcpb` bundle (`manifest.json`). Step-by-step in [DISTRIBUTION.md](DISTRIBUTION.md).

## Disclaimer

For research and educational use. Nothing here is financial advice. Trading FCPO, crude oil, and crypto futures involves substantial risk of loss. This tool is not affiliated with TradingView Inc. or Anthropic, PBC; you are responsible for complying with TradingView's Terms of Use. The server never executes trades — surface signals and place trades yourself.

## Attribution & license

MIT — see [LICENSE](LICENSE) and the full attribution chain in [NOTICE](NOTICE). Original by [@tradesdontlie](https://github.com/tradesdontlie/tradingview-mcp); fork foundation by [@LewisWJackson](https://github.com/LewisWJackson/tradingview-mcp-jackson).
