# Cursor

## Prerequisites

- Cursor with MCP server support.

Cursor's exact MCP configuration UI/schema can change between versions; the
shape below is the standard HTTP-MCP JSON. If your Cursor version's settings
UI or file format looks different, consult [Cursor's own MCP
docs](https://docs.cursor.com/) for the current exact location and schema —
the URL, transport, and header shown here will still be correct even if
where you put them differs.

## Add the server (tokenless)

Add to your Cursor MCP config (typically `~/.cursor/mcp.json` for a global
config, or `.cursor/mcp.json` in a project for a project-scoped one):

```json
{
  "mcpServers": {
    "keelen": {
      "url": "https://keelen.ai/mcp"
    }
  }
}
```

This is enough to call `signup` and `verify_email` — see [`../SETUP.md`](../SETUP.md)
to drive the whole flow from chat.

## Add the server (with a bearer key already in hand)

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

## Where the config lives

`~/.cursor/mcp.json` (global) or `.cursor/mcp.json` at your project root
(project-scoped) — confirm the exact path for your Cursor version in
Cursor's MCP settings panel, which can also generate/edit this file for you.

## Updating the header after `verify_email`

Edit the `headers.Authorization` value in `mcp.json` to `Bearer <api_key>`,
save, and reconnect the server from Cursor's MCP settings panel (or restart
Cursor if no explicit reconnect option is available).

## Troubleshooting

- **Connects but every authenticated tool call fails / 401.** Your key is
  missing, wrong, or was rotated/revoked. Re-run the signup flow for a
  fresh key, or rotate one from the Keelen dashboard, then update
  `mcp.json`.
- **Server doesn't appear in the tool list.** Confirm you edited the config
  file Cursor is actually reading (global vs. project scope can differ) and
  reconnect/restart.
- **Connection errors on add.** Both `/mcp` and `/mcp/` are served
  directly, with no redirect, so the trailing slash is never the cause.
