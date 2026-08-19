# Windsurf

## Prerequisites

- Windsurf with MCP server support.

Windsurf's exact MCP configuration UI/schema can change between versions;
the shape below is the standard HTTP-MCP JSON. If your Windsurf version's
settings UI or file format looks different, consult [Windsurf's own MCP
docs](https://docs.windsurf.com/) for the current exact location and
schema — the URL, transport, and header shown here will still be correct
even if where you put them differs.

## Add the server (tokenless)

Add to your Windsurf MCP config (typically `~/.codeium/windsurf/mcp_config.json`,
or reachable via Windsurf Settings → MCP Servers → "View raw config"):

```json
{
  "mcpServers": {
    "keelen": {
      "serverUrl": "https://keelen.ai/mcp"
    }
  }
}
```

This is enough to call `signup` and `verify_email` — see [`../SETUP.md`](../SETUP.md)
to drive the whole flow from chat. (Some Windsurf versions use `url` instead
of `serverUrl` for HTTP servers — if `serverUrl` isn't recognized, try `url`,
matching the shape in [`generic.md`](generic.md).)

## Add the server (with a bearer key already in hand)

```json
{
  "mcpServers": {
    "keelen": {
      "serverUrl": "https://keelen.ai/mcp",
      "headers": {
        "Authorization": "Bearer <api_key>"
      }
    }
  }
}
```

## Where the config lives

`~/.codeium/windsurf/mcp_config.json` on most installs — confirm via
Windsurf Settings → **MCP Servers**, which can also generate/edit this file
for you.

## Updating the header after `verify_email`

Edit the `headers.Authorization` value in the MCP config file to
`Bearer <api_key>`, save, and reconnect/refresh the server from Windsurf's
MCP Servers settings panel.

## Troubleshooting

- **Connects but every authenticated tool call fails / 401.** Your key is
  missing, wrong, or was rotated/revoked. Re-run the signup flow for a
  fresh key, or rotate one from the Keelen dashboard, then update the
  config file.
- **Server doesn't appear in the tool list.** Use the settings panel's
  refresh/reconnect action after editing the file directly.
- **Connection errors on add.** Both `/mcp` and `/mcp/` are served
  directly, with no redirect, so the trailing slash is never the cause.
