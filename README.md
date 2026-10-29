# West Virginia statutes by citation

Keyless. `61-2-1` is first degree murder.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

See `docs/state-law-probe.md` — every state pack proves a MISS is
distinguishable from a HIT before shipping, because many state sites answer
HTTP 200 for statutes that do not exist.

## Stored copy and `data_as_of` (fleet #3546)

The Legislature's code site answers US data centres but fails from China and Singapore. A cache cannot help a first touch, so the West Virginia Code is stored, one JSON
object per chapter, in the shared statute store (R2, `statutes/wv/`) that the Illinois,
Indiana, North Carolina, Wisconsin and Wyoming packs also use. The gateway hands the
pack its R2 binding and the pack reads only the object a call names (a very large
chapter is a small head plus parts, and a call reads the head and one part). A section
found there is answered with no upstream request; the response says
`served_from: "stored-copy"` and carries `data_as_of` (when that chapter was last read
from the source). A section the copy cannot answer (chapter missing, object unreadable,
or listed with no stored text) falls back to the live read. Absence is never remembered:
a chapter published later is served as soon as it exists.

Nothing is loaded at module scope: the data never enters the gateway bundle.

## Refresh

```
node mcps/west-virginia-code/scripts/bake-chapters.mjs            # all chapters -> mcps/west-virginia-code/.bake/
node mcps/west-virginia-code/scripts/bake-chapters.mjs --only X   # one chapter
node mcps/west-virginia-code/scripts/bake-chapters.mjs --resume   # keep source pages already fetched
node mcps/west-virginia-code/scripts/upload-chapters.mjs          # publish to R2 (statutes/wv/), changed files only
```

The source is the Legislature's West Virginia Code; the site returns every section of one article in a single call, so the refresh makes one request per article (about 2,400), not one per section (about 31,000). Each article is checked against the site's own list of its sections, and an article that returns fewer fails the run. Sequential, with a delay between requests, an honest user agent, bounded
retries, and a stop on the first 429/403/451. Exit 0 only if every chapter was fetched
and parsed and the totals clear the floors (default 120 chapters, 25,000 sections with text); anything less exits 1 with the
counts. A chapter whose text is unchanged keeps its previous `data_as_of`. The baked
files are a working copy, not committed (`.bake/` is gitignored); `upload-chapters.mjs`
writes only the files whose content hash differs from the one recorded in
`statutes/wv/_manifest.json`, so a refresh that finds nothing new writes nothing. It
uses `wrangler r2 object put --remote` under the operator login; no API token.

## Data source

West Virginia Legislature, West Virginia Code. West Virginia statutes are public record.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "west-virginia-code": {
      "url": "https://gateway.pipeworx.io/west-virginia-code/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/west-virginia-code/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/wv_code_section \
  -H 'Content-Type: application/json' \
  -d '{"section":"61-2-1"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/wv_code_section`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-west-virginia-code` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/west-virginia-code/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "west-virginia-code": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-west-virginia-code"]
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
ask_pipeworx({ question: "your question about West Virginia Code data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
