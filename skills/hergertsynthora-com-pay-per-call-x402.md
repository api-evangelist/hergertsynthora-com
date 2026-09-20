---
name: hergertsynthora-com-pay-per-call-x402
description: Discover a SYNTHORA product, read its x402 price from the 402 challenge, pay per call in USDC on Base, and verify the Ed25519 receipt on the result.
api: SYNTHORA Machine-Payable API Mesh (openapi/hergertsynthora-com-mesh-aggregate-openapi.yml)
operations: [discoveryManifest, payToPreflight, contractGuard]
generated: '2026-09-19'
method: generated
source: Grounded in the aggregate OpenAPI operationIds, the live 402 responses observed on 2026-09-19, https://api.hergertsynthora.com/pubkey, https://api.hergertsynthora.com/sla/ and https://hergertsynthora.com/terms/.
---

# Pay per call on the SYNTHORA mesh (x402)

Every SYNTHORA product is one `POST` that answers `402 Payment Required` until it is paid. There are no
accounts and no API keys; the 402 body is the contract and the price list.

## Steps

1. **Discover.** `GET https://api.hergertsynthora.com/.well-known/x402.json` (`discoveryManifest`). Each
   `resources[]` entry carries the product URL, `accepts[]` (scheme `exact`, network `eip155:8453`, USDC asset
   `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`, `amount` in 6-decimal atomic units, `payTo`) and, under
   `extensions.bazaar`, the input/output JSON Schema. The gateway form of any product is
   `POST https://api.hergertsynthora.com/v1/{id}`.
2. **Try the free tier first.** Send `X-WALLET: 0x<your Base address>`. The response header `x-free-tier`
   states how many free calls the product allows (observed: 3 on most products, 1 on `wallet-enrich`).
3. **Read the challenge.** Call the product, e.g. `POST /service` with `{"input": "<0x contract address>"}`
   (`contractGuard`, 0.005 USDC). On `402`, read `accepts[0]` from the JSON body (also mirrored base64url in
   the `payment-required` header and as a `WWW-Authenticate: x402 ...` challenge on the gateway).
4. **Optionally preflight the payee.** `POST /preflight` (`payToPreflight`, 0.010 USDC) is the provider's
   verify-before-pay check for any x402 endpoint - use it before a first payment to an unfamiliar `payTo`.
5. **Pay and retry.** Produce an EIP-3009 `transferWithAuthorization` for exactly `amount` USDC to `payTo` and
   retry the same request with it in the `X-PAYMENT` header. Settlement is verified on-chain; delivery only
   happens after settlement succeeds.
6. **Verify the receipt.** A paid `200` is `{ok, niche, result, receipt}`. Recanonicalise `result` with
   `json.dumps(result, sort_keys=True, separators=(",", ":"))` and verify `receipt.sig` with the Ed25519 key at
   `https://api.hergertsynthora.com/pubkey` (kid `synthora-mesh-attest` in the JWKS at
   `https://api.hergertsynthora.com/.well-known/jwks.json`). The settlement tx hash is the invoice.

## Rules

- **Failure is not billed.** If a result cannot be produced you get a `402` and no settlement occurs (SLA
  delivery policy). Check your wallet on Base rather than a support channel.
- **No refunds once delivered** (terms section 2). Treat every paid call as final; there is no reversal
  operation and no idempotency key, so do not retry a paid request blindly - a retry is a second purchase.
- **Third-party targets need proof of control** (terms section 3): assessments against infrastructure you do
  not own are refused without a DNS TXT / meta-tag challenge under `_synthora-auth.<domain>`.
- Outputs are data intelligence, not financial or legal advice (card and terms both say so).
- Deprecations are announced at least 30 days ahead in `llms.txt` and in the 402 body; the envelope
  `{ok, niche, result, receipt}` is stable and grows additively (SLA).
