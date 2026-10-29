# @pipeworx/romania-tenders

Romanian public procurement: recent contract notices and direct acquisitions from SEAP / SICAP, the national electronic procurement system.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1858+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `ro_tender_notices(query?, days?, date_from?, date_to?, limit?, page?)` -- contract notices (CN, SCN, RFQ, PC), newest first, title keyword and publish-date window.
- `ro_direct_acquisitions(query?, hours?, date_from?, date_to?, limit?)` -- small-value direct acquisitions (about 1,600 a day), newest first, name keyword.

## Auth

Keyless.

## Data sources

- Agentia pentru Agenda Digitala a Romaniei (AADR), SEAP / SICAP public notice lists: contract notices and direct acquisitions.

## Notes for the next person

An earlier attempt got the same 2019 rows for any query. Cause (captured with a headless browser, 2026-10-29): the lists can sort OLDEST-first and the portal always sends a publication-date window; without date fields the server returns the head of the archive. The date window is therefore mandatory in this pack.

- Notices: sorted newest-first inside the window; reported total is capped at 3000.
- Direct acquisitions: newest-first while the window holds fewer than 2000; a window at that cap comes back OLDEST-first with only its oldest 2000, so the pack narrows it from the old edge until it fits and reports `effective_window` and a `note`. The source labels these times `+02:00` although they are UTC; `published` is returned as a UTC instant and the source's text is kept as `published_as_listed`.
- Requests need Referer, Origin and Culture: ro-RO headers; no token or cookie. An honest pack User-Agent is accepted from the gateway's edge.
- Title keyword filters match Romanian text; diacritics optional.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "romania-tenders": {
      "url": "https://gateway.pipeworx.io/romania-tenders/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/romania-tenders/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1858+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ro_tender_notices \
  -H 'Content-Type: application/json' \
  -d '{"query":"echipamente","days":14,"limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ro_tender_notices`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-romania-tenders` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/romania-tenders/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "romania-tenders": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-romania-tenders"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Romania Tenders data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
