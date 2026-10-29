# @pipeworx/aurelmark-ted-changes

Has an EU tender's submission deadline moved? Per-lot previous versus revised deadline for tenders watched by Aurelmark (Ishan Patel), each change citing both official TED notices. The change feed that `ted-eu` (search and detail) does not have.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1858+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `aurelmark_ted_changes(notice_id, cursor?, limit?)` — for a watched tender: current deadline per lot, and every recorded change with lot, previous deadline, revised deadline, both TED notice references and links, and freshness. Always carries a `status` (below).
- `aurelmark_ted_coverage()` — which tenders are watched, since when, last successful check, retention window and eligibility rule.

## Reading the answer: five outcomes, never an ambiguous empty list

| `status` | Meaning |
|---|---|
| `not_covered` | The tender is not watched. Says nothing about its deadline. |
| `covered_no_change_recorded` | Watched and fresh; no change since `observation_start`. The first observation is a baseline, not history. |
| `change_recorded` | One or more changes, each tied to a lot and to both notices. |
| `change_recorded_lot_unresolved` | A change exists but no lot could be attached; it is flagged, never assigned to the whole notice. |
| `quota_exhausted` / `source_stale` | No answer was obtained. NOT "no change". |

Coverage is small (a handful of tenders) and enrolment is by request to the publisher, so a `not_covered` result is the normal outcome for most tenders, not a fault. Pass the tender's FIRST notice number in the procedure (e.g. `662865-2026`); a later notice id is not a watched root. Only standard eForms contract notices with complete lot deadlines are eligible.

## Coverage window

Monitoring starts at each tender's `observation_start`; deadlines are re-checked about hourly; recorded events are retained 30 days. A deadline change published before `observation_start` is not visible.

## What is official and what is Aurelmark's

- Official (TED, the EU's Tenders Electronic Daily): the deadlines themselves and the notices cited. Authoritative.
- Aurelmark's own observation: that a change happened, which lot it was paired to, `observed_at`, `checked_at`, `observation_start`, freshness/completeness.
- The publisher's terms say not to rely on it as your only source of a submission deadline. Confirm on the cited notice.

## Auth

Keyless. The publisher allows 100 successful responses per caller per day, reset at midnight UTC; errors never count. Pipeworx calls from shared infrastructure, so that allowance is shared across callers and can run out, reported as `quota_exhausted`. A prepaid key from the publisher raises it; this pack does not take one.

## Data sources

- Aurelmark (Ishan Patel), TED submission-deadline change observations, derived from official TED eForms contract notices (Publications Office of the EU). Called live per request; nothing is stored or copied here. Publisher terms (accuracy, acceptable use): https://aurelmark.live/terms. TED notice content is public EU data; Aurelmark is not affiliated with the EU.

Notes: the publisher states its API under `/v1` is additive-only. 404 means not watched, 402 free allowance used, 503 the publisher could not confirm TED within three hours (it refuses to serve stale or partial data).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "aurelmark-ted-changes": {
      "url": "https://gateway.pipeworx.io/aurelmark-ted-changes/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/aurelmark-ted-changes/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/aurelmark_ted_changes \
  -H 'Content-Type: application/json' \
  -d '{"notice_id":"662865-2026"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/aurelmark_ted_changes`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-aurelmark-ted-changes` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/aurelmark-ted-changes/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "aurelmark-ted-changes": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-aurelmark-ted-changes"]
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
ask_pipeworx({ question: "your question about Aurelmark Ted Changes data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
