---
name: hergertsynthora-com-wallet-and-tx-screening
description: Screen a counterparty wallet for risk and sanctions exposure, then decode the intent of a transaction before signing it.
api: SYNTHORA Machine-Payable API Mesh (openapi/hergertsynthora-com-mesh-aggregate-openapi.yml) + Tx Intent Classifier (openapi/services/hergertsynthora-com-txintent-openapi.json)
operations: [walletEnrich, txintent]
generated: '2026-09-19'
method: generated
source: Grounded in operation walletEnrich (POST /v1/wallet-enrich, 0.15 USDC, aggregate OpenAPI; live 402 observed 2026-09-19 with x-free-tier 1) and operation txintent (POST https://txintent.hergertsynthora.com/service, 0.010 USDC, requestBody {input: tx hash, chain: base|ethereum}).
---

# Screen a wallet, then decode the transaction

## Steps

1. **Enrich the counterparty.** `POST https://api.hergertsynthora.com/v1/wallet-enrich` with
   `{"address": "<0x address or ENS name>"}` (`walletEnrich`, 0.15 USDC; one free call per wallet with
   `X-WALLET`). The result carries a 0-100 risk score with level, sanctions screening against OFAC SDN wallet
   lists, on-chain activity on Ethereum + Base, and labels (ENS reverse record, GoPlus address-security flags).
2. **Decide on the counterparty.** Treat `sanctions.status != CLEAR` or `risk.level == high` as a stop; the
   provider's terms require that payments themselves are screened by the facilitator, but your own
   counterparty decision is yours.
3. **Decode the transaction before signing.** `POST https://txintent.hergertsynthora.com/service` with
   `{"input": "<tx hash 0x...>", "chain": "base"}` (`txintent`, 0.010 USDC). It classifies the real intent
   (approve / swap / transfer / mint / deploy) and flags unlimited approvals and opaque selectors; the MCP
   tool of the same name accepts `{"to": ..., "data": ...}` for an unsent transaction.
4. **Verify both receipts.** Check `receipt.sig` against the mesh JWKS before acting on either verdict.

## Rules

- Read-only verdicts; nothing here moves funds. The per-call USDC is the only irreversible action.
- No idempotency key: a retried call is a new purchase once the free tier is used.
- Data intelligence, not legal or compliance advice - the provider states it "does not operate bank KYC".
