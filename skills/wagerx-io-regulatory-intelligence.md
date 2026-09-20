---
name: wagerx-regulatory-intelligence
description: Pull WagerX's gambling regulatory alert feed over REST, and reach for the MCP tool when a jurisdiction-scoped, source-linked answer is needed.
api: WagerX iGaming & Regulatory Intelligence API
operations:
  - getRegulatoryIntel
  - callWagieMCP
generated: '2026-09-19'
method: generated
source: openapi/wagerx-io-openapi.yml + mcp/wagerx-io-mcp.yml + mcp/wagerx-io-tool-crosswalk.yml
---

# Regulatory intelligence

1. **Global feed.** `GET https://wagerx.io/api/regulatory-intel` (`getRegulatoryIntel`) returns
   `{alerts[], summary, threat_label, threat_level, updated}`. `updated` is a Unix epoch. An empty `alerts`
   with "No recent regulatory alerts detected." is a real answer, not an error.
2. **Jurisdiction-scoped.** The REST feed takes no jurisdiction parameter. For "what is the status in
   Germany?" call the MCP tool instead: `POST https://wagerx.io/mcp` (`callWagieMCP`), method `tools/call`,
   `{"name":"regulatory_intelligence","arguments":{"jurisdiction":"Germany","limit":5}}`. The tool resolves
   the jurisdiction through WagerX's approved-regulator registry and drops records whose source URL is not
   an approved authority domain.
3. **Handle the circuit.** If `/mcp` answers HTTP 503 with JSON-RPC error `-32001` ("temporarily disabled by
   the automatic usage safety circuit"), back off and retry later — there is no `Retry-After`. Fall back to
   the REST feed and to the CC BY 4.0 dataset at `https://wagerx.io/regulatory/data.json` (documented in
   llms.txt, not in the OpenAPI).
4. **It is research, not advice.** Every regulatory answer carries `legal_advice: false` semantics in
   `RegulatorySnapshot`; keep that qualifier.

30 requests/minute per IP; no auth.
