# Distribution & launch checklist

How to take `tradingview-mcp-thaqif` from a repo to a widely-installed MCP. Work top to bottom; each step builds on the last. Items marked **(you)** need your accounts/credentials.

---

## 0. Pre-flight

- [ ] Confirm license compatibility — upstream is **MIT**, so redistribution of a modified version is permitted **with attribution**. Keep `LICENSE`, `NOTICE`, and the TradingView non-affiliation notice.
- [ ] Decide repo strategy (see `README.md` in this kit). Recommended: make `tradingview-mcp-thaqif` the full fork.

## 1. Assemble the fork repo

- [ ] Copy the server source into the repo: `src/`, `scripts/`, `tests/`, `rules.example.json` (from `~/tradingview-mcp-jackson`). **Do not** copy `node_modules/`, `.git/`, `.env`, or `safety-check-log.json`.
- [ ] Add this kit's files to the repo root: `LICENSE` (replace), `NOTICE`, `SECURITY.md`, `package.json` (merge), `manifest.json`, `server.json`.
- [ ] Add a shebang as the **first line** of `src/server.js` so the npm `bin` works: `#!/usr/bin/env node`
- [ ] Paste the blocks from `install-snippets.md` into your `README.md`. Add a 20-30s demo GIF near the top.
- [ ] Commit and push.

## 2. Test locally

```bash
cd tradingview-mcp-thaqif
npm install
node src/server.js        # should start and wait on stdio (Ctrl-C to stop)
npm test                  # run the suite
npx .                     # simulate the published bin
```

- [ ] With TradingView Desktop running (CDP enabled), point Claude Desktop at the local path and confirm `tv_health_check` succeeds.

## 3. Publish to npm  **(you)**

```bash
npm login
# bump version if needed:
npm version 2.0.0 --no-git-tag-version
npm publish --access public        # use a scoped name (@tz24cloud/...) if "tradingview-mcp-thaqif" is taken
```

- [ ] Verify a clean install works: `npx -y tradingview-mcp-thaqif` from another directory.
- [ ] Confirm the npm page shows README, repo link, and keywords.

## 4. Official MCP Registry  **(you)**

The registry stores **metadata only**, so npm must be published first. It verifies the `io.github.<username>` namespace via GitHub OAuth.

```bash
# get the publisher CLI (see modelcontextprotocol/registry)
# then, from the repo root containing server.json:
mcp-publisher login github
mcp-publisher publish
```

- [ ] Confirm your server appears at registry.modelcontextprotocol.io.
- [ ] `name` in `server.json` must match your GitHub namespace: `io.github.TZ24cloud/tradingview-mcp-thaqif`.

## 5. Claude one-click (.mcpb)  **(you)**

Build a Desktop Extension bundle from `manifest.json` (see github.com/modelcontextprotocol/mcpb):

```bash
npx @anthropic-ai/mcpb init      # if you want to regenerate the manifest interactively
npx @anthropic-ai/mcpb validate  # validate manifest.json
npx @anthropic-ai/mcpb pack      # produces tradingview-mcp-thaqif.mcpb
```

- [ ] Double-check `manifest.json` env var names match what `src/connection.js` actually reads (e.g., the CDP port). Adjust `user_config` accordingly.
- [ ] Test: in Claude Desktop, Settings -> Extensions -> Advanced -> Install Extension -> pick the `.mcpb`. Confirm it runs.
- [ ] Attach the `.mcpb` to your GitHub Releases.
- [ ] Submit to Claude's curated Extensions directory (Anthropic-reviewed) so it shows under Browse Extensions.

## 6. Third-party directories  **(you)**

Most accept community submissions; a good README + working install command is the bar.

- [ ] **PulseMCP** — Submit button in the nav (hand-reviewed, tracks weekly visitors = social proof).
- [ ] **Smithery** — app-store UI; add install command; optional hosted remote.
- [ ] **Glama** — auto-indexes public GitHub MCP repos; claim/manage your listing.
- [ ] **mcp.so** — Submit button or their GitHub issues.
- [ ] **awesome-mcp-servers** — open a PR adding your entry under the right category.
- [ ] (Optional) `mcp-submit` CLI pushes to 10+ directories in one command.

## 7. Docs & demo

- [ ] 30-second screen recording: morning brief on FCPO -> Claude returns a Wyckoff bias. Convert to GIF for the README; keep the full clip for YouTube/X.
- [ ] README sections: What it does, Requirements (TradingView sub + CDP), Install (per client), Safety, Attribution.

## 8. Launch  **(you)**

Use the drafts in `LAUNCH_POSTS.md`. Sequence: Show HN + dev.to on day one, Reddit (r/mcp, r/algotrading) same day, X thread, then the YouTube demo. Respond to every early comment.

## 9. Maintenance (keeps you ranked)

- [ ] Semver + a maintained `CHANGELOG.md`; tag GitHub Releases.
- [ ] Add CI (GitHub Actions: `npm test` on push) and a green build badge.
- [ ] Issue/PR templates and a short `CONTRIBUTING.md`.
- [ ] Re-publish to npm + bump the registry/`.mcpb` version on each release.

---

## Reference links

- Official MCP Registry: https://registry.modelcontextprotocol.io/ — publish quickstart: https://modelcontextprotocol.io/registry/quickstart
- MCPB (Desktop Extensions): https://github.com/modelcontextprotocol/mcpb — Claude docs: https://claude.com/docs/connectors/building/mcpb
- Directories: PulseMCP (pulsemcp.com), Smithery (smithery.ai), Glama (glama.ai), mcp.so
- npm publishing for MCP: https://www.aihero.dev/publish-your-mcp-server-to-npm
