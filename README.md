# @pipeworx/fca-register

UK Financial Conduct Authority Financial Services Register
(register.fca.org.uk) — the public record of every FCA-regulated firm and
individual: firm search, firm details by FRN, a firm's appointed
representatives, and an individual's own public register record by IRN.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

- `fca_firm_search(q, type?)` — search the register by name;
  `type` is one of `firm` (default), `individual`, `fund`.
- `fca_firm_details(frn)` — a firm's status, business type, and
  address by Firm Reference Number.
- `fca_firm_representatives(frn)` — a firm's appointed
  representatives by FRN. An empty list for a firm with none is a real
  answer, not a failure.
- `fca_individual_lookup(irn)` — a single individual's own public
  register record (name, current approved-function status) by Individual
  Reference Number. Looks up exactly one IRN; does not enumerate or list
  individuals.

## Auth

No key needed from the caller: Pipeworx fronts one. The FCA ties each API key
to a signup email and needs both on every request; a caller who wants to use
their own can pass `_apiKey` as `"<email>:<key>"` (register free at
<https://register.fca.org.uk/Developer/s/registernewuser>).

## Data sources

- <https://register.fca.org.uk/services/V0.1> — base URL. Auth via
  `X-Auth-Email` / `X-Auth-Key` headers, ~50 requests/10s per the FCA's own
  developer documentation.
  - `GET /Search?q=<name>&type=firm|individual|fund`
  - `GET /Firm/{FRN}`
  - `GET /Firm/{FRN}/AR` (appointed representatives)
  - `GET /Individuals/{IRN}`
- <https://register.fca.org.uk/Developer/s/> — developer portal / signup.

**UNVERIFIED beyond the no-key and garbage-key cases — no key exists on the
fleet.** Endpoint paths above are corroborated across multiple independent
third-party client libraries (the `fsrapiclient` / `financial_services_
register_api` Python packages, `CyborgFinance/FCARegisterLaravel`) that all
describe the same shapes, but this pack has never completed a successful
authenticated call. Live-probed 2026-09-23 with a syntactically-valid but
non-functional key: every endpoint answered with a real, on-brand JSON body
(`{"Success":"false", "Value not found"}`, HTTP 404) rather than a DNS
failure or a generic edge error — strong evidence the base URL and path
shapes above are correct, just not proof of the populated/successful
response shape. `raw` is included on every tool's response so a caller with
a real key gets the untouched upstream payload even if a field name
assumption here turns out stale.

Personal-data scope is deliberate: `fca_individual_lookup` exposes only the
register's own single-record public fields for one IRN a caller already
has. This pack ships no bulk/enumeration tool over individuals (no
"list every approved person", no alphabet crawl) — do not add one without
re-checking that boundary.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "fca-register": {
      "url": "https://gateway.pipeworx.io/fca-register/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/fca-register/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/fca_firm_search \
  -H 'Content-Type: application/json' \
  -d '{"q":"Barclays Bank","type":"firm"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/fca_firm_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "fca-register": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-fca-register"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-fca-register
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Fca Register data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
