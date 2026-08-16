# Crypto Data & Market Analysis Agent

**Crypto and macro market intelligence for autonomous agents — the signal set a trading desk watches, delivered as structured JSON.**

[![MCP Registry](https://img.shields.io/badge/MCP_Registry-listed-2ea44f)](https://registry.modelcontextprotocol.io)
[![TensorBlock MCP Index](https://mcp-index.tensorblock.co/v1/servers/github-dopionut-jpg-crypto-data-mcp-7ab661d9/badge.svg)](https://tensorblock.co/mcp/servers/github-dopionut-jpg-crypto-data-mcp-7ab661d9)
[![Transport](https://img.shields.io/badge/transport-streamable_http-blue)](https://modelcontextprotocol.io)
[![Tools](https://img.shields.io/badge/tools-13-orange)]()
[![Auth](https://img.shields.io/badge/API_key-not_required-brightgreen)]()

```
https://asistent-crypto.vercel.app/mcp
```

No API key. No signup. 13 read-only tools, live at call time.

---

## Most crypto APIs return a price. This returns positioning.

A price tells you where the market is. Positioning tells you what happens next — and it is much harder to get right.

Here is a real example from building this server. Aggregating perpetual funding for BTC naively — averaging across every venue that publishes a rate — produced **9.2%**. That number implied an extremely overheated market on the verge of a long squeeze.

The real number was **0.43%**.

The gap came from thin venues with almost no open interest publishing wild outlier rates, each weighted exactly the same as a deep, liquid book. The average was arithmetically correct and completely useless.

This server takes the **largest-open-interest perpetual per venue** across Binance, Bybit and OKX, and excludes the thin venues entirely. That single decision is the difference between a signal and noise — and it is the kind of decision an agent cannot make for itself from a raw endpoint.

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

## The 13 tools

### Positioning & derivatives

| Tool | Returns |
|---|---|
| `get_derivatives` | Funding rate and open interest for one coin across Binance, Bybit, OKX — per-venue values, average funding, total OI in USD, funding spread |
| `get_derivatives_aggregate` | The same signal with liquidity filtering applied — largest-OI perpetual per venue, thin venues excluded |

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
| `get_macro_rates` | Fed funds rate, 2Y and 10Y Treasury yields, CPI inflation YoY, dollar index |
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
