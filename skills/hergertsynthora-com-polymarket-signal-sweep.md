---
name: hergertsynthora-com-polymarket-signal-sweep
description: Run the six paid Polymarket signal products in sequence - whales, odds movers, resolution watch, dispute risk, insider radar and canonical resolution status - for a prediction-market agent.
api: SYNTHORA Machine-Payable API Mesh (openapi/hergertsynthora-com-mesh-aggregate-openapi.yml)
operations: [polymarket_whales, polymarket_odds_movers, polymarket_resolution_watch, dispute_risk, insider_radar, market_resolution]
generated: '2026-09-19'
method: generated
source: Grounded in the six polymarket-tagged operations of the aggregate OpenAPI (paths /v1/polymarket-whales, /v1/polymarket-odds-movers, /v1/polymarket-resolution-watch, /v1/dispute-risk, /v1/insider-radar, /v1/market-resolution) and their x-payment-info prices; the same six exist as MCP tools on https://mcp.hergertsynthora.com/mcp.
---

# Polymarket signal sweep

All six are `POST` JSON calls on the gateway (`https://api.hergertsynthora.com/v1/<id>`) and all follow the
x402 flow in `hergertsynthora-com-pay-per-call-x402`. Prices are read from each operation's `x-payment-info`.

## Steps

1. **Find smart money.** `polymarket_whales` (0.15 USDC) - latest large trades with wallet, side, size,
   market and tx hash; tune with `min_usd` in the body (default >= 5000 USD per the MCP description).
2. **Find momentum.** `polymarket_odds_movers` (0.10 USDC) - markets ranked by biggest odds move over 1h /
   24h / 1wk with volume and spread.
3. **Find markets about to settle.** `polymarket_resolution_watch` (0.10 USDC) - markets near resolution with
   UMA status (proposed / disputed) and risk flags.
4. **Score settlement risk.** For each candidate, `dispute_risk` (0.05 USDC) returns a 0-100 dispute /
   resolution risk with UMA status, bond and holder concentration.
5. **Check for insider-shaped flow.** `insider_radar` (0.03 USDC) - fresh wallets, size out of proportion to
   volume, extreme-odds entries before moves.
6. **Confirm the canonical state before acting.** `market_resolution` (0.01 USDC) - state, UMA status,
   sources and estimated payout for one market.

## Rules

- Every response is Ed25519-signed; verify `receipt.sig` before trusting a verdict you will trade on.
- These are read-only verdicts: nothing here places an order. Reversibility is `na` on the provider side and
  the USDC you pay per call is non-refundable once delivered.
- The MCP versions of these tools accept a single `input` string; the real parameters (e.g. `min_usd`, a
  market slug) are in the OpenAPI requestBody, so prefer the REST contract when you need typed inputs.
- Not financial advice, by the provider's own statement on every card and in the terms.
