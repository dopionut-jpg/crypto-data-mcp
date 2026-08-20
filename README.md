# Crypto Data & Market Analysis Agent

**Crypto and macro market intelligence for autonomous agents — the signal set a trading desk watches, delivered as structured JSON.**

[![MCP Registry](https://img.shields.io/badge/MCP_Registry-listed-2ea44f)](https://registry.modelcontextprotocol.io)
[![TensorBlock MCP Index](https://mcp-index.tensorblock.co/v1/servers/github-dopionut-jpg-crypto-data-mcp-7ab661d9/badge.svg)](https://tensorblock.co/mcp/servers/github-dopionut-jpg-crypto-data-mcp-7ab661d9)
[![Transport](https://img.shields.io/badge/transport-streamable_http-blue)](https://modelcontextprotocol.io)
[![Tools](https://img.shields.io/badge/tools-17-orange)]()
[![Auth](https://img.shields.io/badge/API_key-not_required-brightgreen)]()

```
https://asistent-crypto.vercel.app/mcp
```

No API key. No signup. 17 read-only tools, live at call time.

---

## Most crypto APIs return a price. This returns positioning.

A price tells you where the market is. Positioning tells you what happens next — and it is much harder to get right.

Here is the example that shaped this server. Roughly 190 venues publish a perpetual funding rate per major coin. Average them the obvious way — one venue, one vote — and the result is wrong by an order of magnitude, because a book with a few hundred thousand dollars in it counts exactly as much as one with ten billion. Venues printing **9-10% per 8h**, around 30% *per day*, on books nobody trades are what drag the number.

**[See it computed live →](https://asistent-crypto.vercel.app/proof)** — the same market measured three ways (simple average, open-interest-weighted, and what this server returns), recomputed on every page load, with the distorting venues and their open interest shown. No number on that page is written by hand, because the multiple moves every day and a stale proof is worse than none.

This server takes the **largest-open-interest perpetual per venue** across Binance, Bybit, OKX and Hyperliquid — the three deepest centralised books plus the largest perpetual DEX — and excludes the thin venues entirely. That single decision is the difference between a signal and noise — and it is the kind of decision an agent cannot make for itself from a raw endpoint.

The same discipline applies across every tool here.

---

## Quick start

**Claude Code**

```bash
claude mcp add --transport http crypto-data https://asistent-crypto.vercel.app/mcp
```

**Cursor, Windsurf, VS Code** — add to your MCP config:

```json
{
  "mcpServers": {
    "crypto-data": {
      "url": "https://asistent-crypto.vercel.app/mcp"
    }
  }
}
```

**Claude Desktop** — Settings → Connectors → Add custom connector → paste the URL above.

**Any client without native remote support:**

```json
{
  "mcpServers": {
    "crypto-data": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://asistent-crypto.vercel.app/mcp"]
    }
  }
}
```

Then ask your agent something real:

> *"What's BTC funding and open interest across venues right now, and does the macro backdrop support a long here?"*

---

## The 17 tools

### Positioning & derivatives

| Tool | Returns |
|---|---|
| `get_market_brief` | **The verdict, in one call.** Composes positioning, sentiment, implied volatility, the macro regime and execution cost into a single stance — risk-on, risk-off, fragile or neutral — with a confidence level, every driver that produced it, and the concrete numeric conditions that would invalidate it. Deterministic, not a model opinion: the same numbers always give the same verdict, and any single driver can be argued with. *Fragile* is a state rather than a direction — crowded positioning together with cheap implied volatility, meaning the market leans one way and pays almost nothing for protection |
| `get_execution_cost` | **What does an entry actually cost?** Live order book depth on Bybit and OKX, walked at your size: average fill price, slippage against mid in basis points *and* in dollars, the spread, depth within 1% of mid, and which venue is cheaper for that exact size — separately for buying and selling. A size the book cannot absorb comes back as unfilled rather than priced by extrapolation. The floor is half the spread; anything above it is your size eating through levels |
| `get_derivatives` | **One coin, venue by venue.** Funding and open interest for each of Binance, Bybit, OKX and Hyperliquid separately. Hyperliquid is the only DEX in the set; its native hourly funding is converted to the same percent-per-8h unit as the rest, so the venues are directly comparable, plus average funding, total OI in USD and the funding spread. Reach for this when venue divergence matters — one venue far more positive than the rest is localised positioning, not market consensus |
| `get_derivatives_aggregate` | **A whole watchlist at once.** Headline funding and OI for up to 10 coins in a single call, no venue breakdown. Costs one upstream request regardless of how many coins you ask for, so it beats looping the single-coin tool. Funding comes both per 8h and annualized, so 0.0067% per 8h also reads as 7.3% a year |
| `get_implied_volatility` | **How far is price expected to move?** The 30-day options-implied volatility index (DVOL) for BTC and ETH, annualized in percent, with its percentile over the last month and the derived expected move over 1, 7 and 30 days. Every other derivatives tool here says how the market is *positioned*; this one prices how expensive protection is. A 35% reading says nothing alone — at the 5th percentile of the month it says optionality is cheap and the market is complacent. Distinct from the VIX in `get_macro_rates`, which is US equity volatility |
| `get_market_history` | **Is this reading actually extreme?** Funding, open interest, Fear & Greed and BTC dominance from a series captured every ~10 minutes, each as current, median, min, max and percentile over a window of up to 7 days. A funding number means little until you know where it sits in its own distribution |

### On-chain

| Tool | Returns |
|---|---|
| `get_eth_whale_flows` | Large ETH transfers (>100 ETH) to and from known exchange wallets in the last 2 hours — inflows as distribution pressure, outflows as accumulation |
| `get_eth_address` | ETH balance and recent large transfers for any address |
| `get_btc_network` | Bitcoin network health: hashrate, difficulty, transaction count, miner revenue, largest mempool transactions |
| `get_defi_overview` | Total value locked, top chains by TVL, stablecoin market cap — liquidity and dry powder |

### Market & sentiment

| Tool | Returns |
|---|---|
| `get_crypto_market` | Spot price, market cap, 24h volume and change for any coins |
| `get_fear_greed` | Fear & Greed Index with classification and recent history — a contrarian input, not a standalone signal |
| `get_market_dominance` | BTC.D, ETH.D, USDT.D as a risk-on/risk-off regime filter and rotation signal |
| `get_crypto_news` | Recent headlines for narrative context and catalysts |

### Macro

| Tool | Returns |
|---|---|
| `get_macro_rates` | The macro regime as signals, not raw levels: the 2s10s yield curve spread with its direction and an inverted flag, plus Fed funds, 2Y and 10Y yields, CPI YoY, dollar index, VIX and M2 — each with its previous reading |
| `get_market_quotes` | S&P 500, gold and Nasdaq 100 — traditional-market correlation context |
| `get_economic_calendar` | Scheduled catalysts ahead: FOMC, CPI, PCE, NFP, GDP, Retail Sales |

---

## What makes this different

**Cross-domain by design.** Exchange data alone cannot tell you whether a funding spike is a local squeeze or a macro repricing. Funding, on-chain flows, dominance and the rate environment live behind one endpoint here precisely so an agent can compose them in a single reasoning pass.

**Liquidity-weighted, not naively averaged.** See the 9.2% example above.

**Live at call time.** Every response is built from primary sources when the tool is called. Nothing is simulated, mocked or hardcoded.

**Read-only.** No tool in this server can move funds, place an order, or write anything anywhere. The surface is deliberately narrow.

**Stateless.** No session storage, no account, no key to leak.

---

## Rate limits

120 requests per hour per IP on the free tier — comfortably above normal interactive agent use.

Need more, or want an LLM-synthesized market verdict rather than raw JSON? The same intelligence is available as a paid HTTP API with per-call settlement in USDC on Base (x402), or with prepaid credits:

- `POST /api/x402/query` — $0.05 per call, any single tool
- `POST /api/x402/analyze` — $0.25 per call, multi-tool synthesis into a structured verdict with regime, confidence, key levels and invalidation

Discovery: [`/.well-known/agent-card.json`](https://asistent-crypto.vercel.app/.well-known/agent-card.json) · [OpenAPI](https://asistent-crypto.vercel.app/openapi.json)

---

## Links

- **MCP endpoint:** `https://asistent-crypto.vercel.app/mcp`
- **Official MCP Registry:** `io.github.dopionut-jpg/crypto-data-market-analysis`
- **Agent Card:** https://asistent-crypto.vercel.app/.well-known/agent-card.json

---

## Disclaimer

This server provides market data and analysis for informational purposes only. It is not financial advice, and nothing it returns should be treated as a recommendation to buy or sell any asset. Crypto markets carry substantial risk of loss. Execution decisions are yours alone.

## License

MIT
