# Generic / other MCP clients

Keelen's MCP server speaks standard **Streamable HTTP** MCP transport with
**Bearer token** authentication. If your client isn't covered by one of the
other guides in this directory, it almost certainly supports this shape —
consult your client's own documentation for the exact config file location
and key names, and adapt the values below.

## Connection details

- **URL:** `https://keelen.ai/mcp`
- **Transport:** Streamable HTTP
- **Server name:** `keelen` (use this as the config key/identifier — not
  required by the protocol, but keeps things consistent if you're following
  along with other docs in this repo)
- **Auth:** `Authorization: Bearer <api_key>` header once you have a key
  (unnecessary for the tokenless `signup` / `verify_email` tools)

## Protocol support

The server serves MCP protocol revision **`2026-07-28`** and the earlier
handshake revisions (`2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`)
from the same `https://keelen.ai/mcp` endpoint. The revision is selected per
request from your client's `MCP-Protocol-Version` header; a request without
that header is served the older handshake.

**No client action is required** — whichever revision your client speaks,
existing configurations keep working unchanged, and every tool behaves
identically on both.

Clients on `2026-07-28` additionally get `server/discover`, and a `tools/list`
response that is cacheable for an hour (privately scoped — the listing rides
an authenticated request, so it is never shared between workspaces).

## Standard HTTP-MCP JSON shape

Most JSON-config-based clients use some variant of this shape:

```json
{
  "mcpServers": {
    "keelen": {
      "url": "https://keelen.ai/mcp",
      "headers": {
        "Authorization": "Bearer <api_key>"
      }
    }
  }
}
```

Omit the `headers` block entirely for a tokenless connection (enough to
call `signup` and `verify_email` — see [`../SETUP.md`](../SETUP.md)).

If your client uses a different key name for the URL (`serverUrl`,
`endpoint`, etc.) or a different mechanism for custom headers, consult your
client's docs for the exact field name — the values themselves (the URL and
the `Authorization: Bearer <api_key>` header) don't change.

## Updating the header after `verify_email`

Update the `Authorization` header value to `Bearer <api_key>` wherever your
client stores it, then reconnect the server (most clients need an explicit
disconnect/reconnect, or a restart, to pick up header changes).

## Troubleshooting

- **Connects but every authenticated tool call fails / 401.** Your key is
  missing, wrong, or was rotated/revoked. Re-run the signup flow for a
  fresh key (`signup` → `verify_email`), or rotate one from the Keelen
  dashboard (**Settings → API keys**), then update the header.
- **Connection issues on a bare `/mcp` request.** Both `/mcp` and `/mcp/`
  are served directly — the server issues no redirect, so a raw HTTP
  library does not need redirect-following on POST to reach either.
- **No tools appear after connecting.** Some clients require an explicit
  "list tools" / capability refresh after connecting — check your client's
  docs for how it discovers tools.
