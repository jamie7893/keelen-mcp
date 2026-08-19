# Codex

## Prerequisites

- Codex CLI with MCP server support, configured via `~/.codex/config.toml`.

## Add the server (tokenless)

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.keelen]
url = "https://keelen.ai/mcp"
```

This is enough to call `signup` and `verify_email` — see [`../SETUP.md`](../SETUP.md)
to drive the whole flow from chat.

## Add the server (with a bearer key already in hand)

Export the raw key as an environment variable and point Codex at it. Codex
builds the `Authorization: Bearer <api_key>` header for you, so `KEELEN_TOKEN`
holds the **raw** token — no `Bearer ` prefix, no surrounding quotes.

```sh
export KEELEN_TOKEN=<api_key>
```

```toml
[mcp_servers.keelen]
url = "https://keelen.ai/mcp"
bearer_token_env_var = "KEELEN_TOKEN"
```

This is the same wiring the Keelen dashboard's **MCP access** card generates,
so the dashboard and these docs stay consistent.

## Where the config lives

`~/.codex/config.toml` in your home directory. Edit it directly with a text
editor.

## Updating the token after `verify_email`

Re-export `KEELEN_TOKEN` in your shell with the new raw key (again: no
`Bearer ` prefix, no quotes), then restart/reconnect Codex so it picks up the
change. The `config.toml` above doesn't need to change.

## Troubleshooting

- **Connects but every authenticated tool call fails / 401.** This is a token
  problem, not a header-format bug — the server accepts the bare token and
  strips an optional `Bearer ` prefix, so the format is not the cause. Your key
  is most likely missing, wrong, rotated/revoked, or was pasted with a stray
  `Bearer ` prefix or surrounding quotes. Re-run the signup flow for a fresh
  key, or rotate one from the Keelen dashboard, then re-export `KEELEN_TOKEN`
  with the raw value.
- **Server doesn't show up after editing config.** Codex reads
  `config.toml` at startup — restart the Codex session after editing it.
- **Connection errors.** Both `/mcp` and `/mcp/` are served directly, with
  no redirect, so a custom HTTP client outside Codex's own MCP client does
  not need redirect-following on POST.
