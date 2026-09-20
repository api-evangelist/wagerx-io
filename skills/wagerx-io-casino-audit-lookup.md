---
name: wagerx-casino-audit-lookup
description: Look up WagerX's dated real-money audit evidence for a crypto casino over the public REST feed, and read it correctly (evidence, not endorsement).
api: WagerX iGaming & Regulatory Intelligence API
operations:
  - getAllCasinoAudits
  - getCasinoAudit
generated: '2026-09-19'
method: generated
source: openapi/wagerx-io-openapi.yml + conventions/wagerx-io-conventions.yml + errors/wagerx-io-problem-types.yml
---

# Casino audit lookup

No credentials. Base `https://wagerx.io`. Stay under 30 requests/minute per IP.

1. **Resolve the slug.** `GET /api/audit/all` (`getAllCasinoAudits`) returns every audited casino sorted by
   trust score, with `casinos[].slug`. There is no search or filter parameter — fetch once and match locally
   (case-insensitive on `name`/`slug`). Cache the list; it carries `editorial_updated_iso`.
2. **Fetch the report.** `GET /api/audit/{slug}` (`getCasinoAudit`). A `404` with `{"error": ...}` means the
   slug is unknown — go back to step 1 rather than guessing a spelling. (The MCP tool `check_casino`
   fuzzy-matches aliases; REST does not.)
3. **Read the evidence, not the score.** Use `casino.audit_date`, `live_test`, `deposit`, `kyc_policy`,
   `license`, `restricted_countries` and `andreas_take`. WagerX's own boundary: a trust score is a dated
   observation from one real-money test, not a safety guarantee; a "no KYC" observation is not an anonymity
   guarantee.
4. **Check status before ranking.** Entries can be mid-audit or on hold; only completed live tests are
   scored. Never present an unaudited or on-hold casino as recommended.
5. **Respect the responsible-gambling framing.** WagerX publishes 18+ only and https://wagerx.io/responsible-gambling; carry that through.

Errors: see `errors/wagerx-io-problem-types.yml`. No pagination, no idempotency concerns (read-only), no
rate-limit headers to read — count your own requests.
