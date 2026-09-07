# Cursor

## Prerequisites

- Cursor with MCP server support.

Cursor's exact MCP configuration UI/schema can change between versions; the
shape below is the standard HTTP-MCP JSON. If your Cursor version's settings
UI or file format looks different, consult [Cursor's own MCP
docs](https://cursor.com/docs/mcp) for the current exact location and schema:
the URL, transport, and header shown here will still be correct even if
where you put them differs.

## Add the server (tokenless)

Add to the global Cursor MCP config, `~/.cursor/mcp.json` in your home
directory. Use this location so the bearer key added later stays outside
your repository:

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

Save this entry in the global `~/.cursor/mcp.json` only. Never save the key
in a repository, including a gitignored project config.

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

Use `~/.cursor/mcp.json` in your home directory for the authenticated Keelen
connection. Cursor also supports a project file at `.cursor/mcp.json`, but
do not put a Keelen bearer key there. If you already added a tokenless
project entry, remove that entry when moving the connection to the global
config. Preserve any unrelated server entries in both files.

## Updating the header after `verify_email`

Edit the `headers.Authorization` value in the global `~/.cursor/mcp.json`
to `Bearer <api_key>`, save, and reconnect the server from Cursor's MCP
settings panel (or restart Cursor if no explicit reconnect option is
available).

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
