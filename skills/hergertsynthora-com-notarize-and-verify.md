---
name: hergertsynthora-com-notarize-and-verify
description: Notarize a SHA-256 (proof-of-existence, proof-of-execution or proof-of-agreement) with the SYNTHORA Agent Notary and later verify the receipt, over REST, A2A or MCP.
api: SYNTHORA Agent Notary (openapi/hergertsynthora-com-agent-notary-openapi.yml)
operations: [trust_notary]
generated: '2026-09-19'
method: generated
source: Grounded in the single operation trust_notary (POST https://notary.hergertsynthora.com/service), the notary llms.txt, the notary agent card skill inputSchema (op enum notarize / execution / agreement / verify) and the live 402 observed on 2026-09-19 (amount 2000 atomic = 0.002 USDC, x-free-tier 3).
---

# Notarize and verify with the Agent Notary

One endpoint, one operation (`trust_notary`), four modes selected by `op`. The content is never stored:
you send a hash (or content the notary hashes) and receive an Ed25519-signed receipt with an id `nt-...`.

## Steps

1. **Hash locally.** Compute `sha256` of the document, execution trace or agreement text. Sending `hash`
   instead of `content` keeps the payload off the provider's systems entirely.
2. **Choose the proof.**
   - `{"op": "notarize", "hash": "<sha256 hex>", "meta": {...}}` - proof-of-existence.
   - `{"op": "execution", "input": ..., "output": ...}` - proof-of-execution of an agent step.
   - `{"op": "agreement", "party_a": "...", "party_b": "...", "terms": {...}}` - proof-of-agreement between two agents.
3. **Call.** `POST https://notary.hergertsynthora.com/service`. First three calls per wallet are free with
   `X-WALLET: 0x<Base address>`; after that the `402` asks for 0.002 USDC (`payTo`
   `0x6D9F318463168fF637b50E5b9a21392e97c7c167` on the live challenge). Same call is available as A2A
   JSON-RPC at `/a2a` and as the MCP tool `notario` at `/mcp`.
4. **Keep the receipt.** The `200` result carries `receipt_id`, `sha256`, `kind`, `notarized_at` and a
   `statement`, signed with the mesh key.
5. **Verify later.** `{"op": "verify", "receipt_id": "nt-..."}` returns the verification; independently,
   check the signature against `https://api.hergertsynthora.com/.well-known/jwks.json`.

## Rules

- A notarization is permanent by design: there is no revoke or void, and the payment is non-refundable once
  the receipt is delivered. Do not notarize secrets - hash them.
- No idempotency key exists; a retried `notarize` mints a second receipt (and, past the free tier, a second
  charge). Store the `receipt_id` from the first success before retrying.
- The notary's `llms.txt` and its agent card disagree on the `payTo` address; trust the `accepts[].payTo` in
  the live 402 (which agrees with the card), never a cached value.
