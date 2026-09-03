# Claude Code

## Prerequisites

- Claude Code CLI installed and on your `PATH` (`claude --version` to check).

## Add the server (tokenless)

```
claude mcp add --transport http keelen https://keelen.ai/mcp
```

This is enough to call `signup` and `verify_email` — see [`../SETUP.md`](../SETUP.md)
to drive the whole flow from chat.

## Add the server (with a bearer key already in hand)

If you already have an API key (minted via `verify_email` or from the
dashboard), add the header directly:

```
claude mcp add --transport http keelen https://keelen.ai/mcp \
  --header "Authorization: Bearer <api_key>"
```

## Where the config lives

`claude mcp add` writes to Claude Code's own MCP server config (managed by
the `claude mcp` subcommands — use `claude mcp list` / `claude mcp get keelen`
to inspect it rather than editing files by hand, since the exact file and
scope — user vs. project — depend on your Claude Code version and how you
ran the add command).

## Updating the header after `verify_email`

Remove and re-add with the header, then reconnect:

```
claude mcp remove keelen
claude mcp add --transport http keelen https://keelen.ai/mcp \
  --header "Authorization: Bearer <api_key>"
```

## Troubleshooting

- **Connects but every authenticated tool call fails / 401.** Your key is
  missing, wrong, or was rotated/revoked. Re-run `verify_email` (via
  `signup` again) or rotate a key from the dashboard, then update the
  header as above.
- **Add command seems to hang or fails to connect.** The URL is not the
  cause: both `/mcp` and `/mcp/` are served directly, with no redirect.
  Re-check the host and the `Authorization` header value.
- **Tool list looks stale after reconnecting with a new header.** Restart
  the Claude Code session so it re-negotiates the connection.
