---
name: wagerx-ask-wagie-a2a
description: Ask the Wagie agent a bounded iGaming or regulatory question over A2A JSON-RPC and verify the Ed25519-signed evidence envelope in the answer.
api: Wagie A2A Agent
operations:
  - getWagieA2ADiscovery
  - sendWagieA2AMessage
  - getWagieAgentCard
generated: '2026-09-19'
method: generated
source: openapi/wagerx-io-openapi.yml + a2a/wagerx-io-a2a.yml + https://wagerx.io/.well-known/wagerx-signing.json (verification recipe verbatim from the key document)
---

# Ask Wagie over A2A

1. **Discover.** `GET https://wagerx.io/.well-known/agent-card.json` (`getWagieAgentCard`) for skills and
   sample questions, or `GET /a2a` (`getWagieA2ADiscovery`) for the short usage note. Both are anonymous.
2. **Send.** `POST https://wagerx.io/a2a` (`sendWagieA2AMessage`) with JSON-RPC 2.0. A2A 1.0 clients use
   method `SendMessage`; 0.3.0 clients use `message/send`. Put the question in
   `params.message.parts[].text`. Stick to the supported question patterns (casino safety, comparisons,
   payout speed, jurisdiction status, official MCP lookup, country recommendations) — the gateway says
   "A2A supports bounded question patterns, not every possible paraphrase."
3. **Read both parts.** The result has a `text` part (prose) and a `data` part shaped as
   `SignedEvidenceEnvelope {data, proof}`. Prefer `data.data.claims[]` and `data.data.evidence[]`; honour
   `evidence_state` (`available | stale | missing | unavailable | unaudited | insufficient_evidence`) and
   each evidence item's `freshness`.
4. **Verify the signature.** Take the `data` object, serialise it as JSON with sorted keys, compact
   separators and ASCII escaping, and verify `proof.signature` (base64url Ed25519) against the JWK at
   `proof.public_key_url` (`https://wagerx.io/.well-known/wagerx-signing.json`, kid
   `wagerx-ed25519-2026-v1`). A valid signature proves WagerX origin and integrity — WagerX's own words —
   not that the claim is true.
5. **Surface the AI notice.** `data.data.ai_transparency.notice` is the provider's Article-50-style
   disclosure; pass it to the end user.

Limits: 30 requests/minute per IP; responses are `Cache-Control: public, max-age=300`. Streaming and push
notifications are `false` in the card — one request, one response.
