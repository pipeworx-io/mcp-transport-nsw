# @pipeworx/transport-nsw

Journey planning, live departures, service alerts and commuter car-park
occupancy for Sydney and the whole of New South Wales — trains, metro, buses,
ferries and light rail.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1684+ live data sources.

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

Both URLs reach the same gateway and the same 1684+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/tfnsw_departures \
  -H 'Content-Type: application/json' \
  -d '{"stop_id":"200060"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/tfnsw_departures`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

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

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-transport-nsw
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

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
