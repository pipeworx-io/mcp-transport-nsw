# @pipeworx/transport-nsw

Journey planning, live departures, service alerts and commuter car-park
occupancy for Sydney and the whole of New South Wales — trains, metro, buses,
ferries and light rail.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

**Requires an API key** — free and self-service at
<https://opendata.transport.nsw.gov.au/>.

- Platform key: `PLATFORM_TFNSW_KEY`
- Caller-supplied: `_apiKey` argument on any tool

## Tools

| Tool | What it answers |
|---|---|
| `tfnsw_departures` | Live departures from a stop, station or wharf |
| `tfnsw_trip_plan` | A full multi-leg journey from A to B |
| `tfnsw_alerts` | Current incidents and trackwork |
| `tfnsw_carparks` | Live park-and-ride occupancy |

## Traps

**The auth header is not a bearer token.** It is literally the word `apikey`
followed by the key: `Authorization: apikey abc123`. Sending `Bearer abc123`
returns 401 with a message about "the calling application", which reads as a
bad key rather than a bad scheme — and sends you off rotating a key that was
fine.

**The trip planner is EFA, not REST.** `/v1/tp/*` is a hosted Mentz EFA
instance, so the parameters are EFA's (`type_dm`, `name_dm`, `itdDate`) and its
"no results" answer is **HTTP 200 with a `systemMessages` array** explaining
why — never an error status. `efaComplaint()` surfaces that message whenever
the result list is empty, because a silent empty journey list is otherwise
indistinguishable from "no trains run between these two places".

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "transport-nsw": {
      "url": "https://gateway.pipeworx.io/transport-nsw/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/transport-nsw/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/tfnsw_departures \
  -H 'Content-Type: application/json' \
  -d '{"stop_id":"200060"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/tfnsw_departures`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-transport-nsw` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/transport-nsw/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "transport-nsw": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-transport-nsw"]
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
ask_pipeworx({ question: "your question about Transport Nsw data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
