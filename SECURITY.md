# Security Policy

## What this server does (and does not) do

`tradingview-mcp-thaqif` is a **local** MCP server. It connects to a copy of
TradingView Desktop already running on your own machine over the Chrome
DevTools Protocol (CDP) and exposes read/control tools to an MCP client
(Claude Desktop, Cursor, Cline, etc.).

- **Local only.** No TradingView data leaves your machine. The server makes no
  outbound network calls of its own beyond the local CDP endpoint.
- **No telemetry.** No analytics, no phone-home.
- **No paywall bypass.** It controls the app you are already logged into and
  requires your own valid TradingView subscription.
- **No autonomous trading.** The server surfaces signals and chart data only.
  Order placement, fund movement, and trade execution are out of scope by
  design (see rule `mcp-010` in `rules.json`). Do not wire this to a live
  broker for automatic execution.

## Threat model / things to know

- The CDP debugging port is powerful. Only enable it on a trusted machine, and
  do not expose the CDP port to your network or the internet.
- An MCP client driving this server can read whatever is on your chart and
  change your chart/indicators/scripts. Review tool calls before approving.
- Treat alert webhook URLs and API keys as secrets — never hard-code them in
  Pine alert messages or commit them (`.gitignore` excludes `.env`/secrets).

## Supported versions

| Version | Supported |
|---------|-----------|
| 2.x     | Yes       |
| < 2.0   | No        |

## Reporting a vulnerability

Please report security issues privately via GitHub Security Advisories on the
repository ("Report a vulnerability") rather than opening a public issue.
Include reproduction steps and affected version. Target response: 5 business
days. Please allow a reasonable window for a fix before public disclosure.
