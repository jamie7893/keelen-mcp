# Claude Desktop

## Prerequisites

- Claude Desktop, a recent version with custom connector / remote MCP
  server support.

## Add the server (tokenless)

1. Open Claude Desktop → **Settings → Connectors** (some versions label
   this **Settings → Integrations**).
2. Choose **Add custom connector** (or **Add MCP server**).
3. Set the URL to:
   ```
   https://keelen.ai/mcp
   ```
4. Leave any auth/header fields blank for now and save. This is enough to
   call `signup` and `verify_email` — see [`../SETUP.md`](../SETUP.md) to
   drive the whole flow from chat.

## Add the server (with a bearer key already in hand)

In the same connector settings, add a custom header:

- **Name:** `Authorization`
- **Value:** `Bearer <api_key>`

## Where the config lives

Claude Desktop stores connector configuration in its own app settings
(managed through the Settings UI). Exact file locations vary by OS and
version and are not guaranteed stable — use the in-app Settings UI rather
than editing config files directly.

## Updating the header after `verify_email`

Open the `keelen` connector's settings again and replace the `Authorization`
header value with `Bearer <api_key>`, then reconnect the connector (toggle
it off and on, or restart Claude Desktop if the UI doesn't offer a
reconnect action).

## Troubleshooting

- **Connects but every authenticated tool call fails / 401.** Your key is
  missing, wrong, or was rotated/revoked. Re-run the signup flow to get a
  fresh key, or rotate one from the Keelen dashboard, then update the
  header.
- **Connector shows an error immediately after adding the URL.** Both
  `/mcp` and `/mcp/` are served directly, with no redirect, so the trailing
  slash is never the cause. Double-check the URL has no typos and try
  removing/re-adding the connector.
- **Header changes don't seem to take effect.** Fully disconnect/remove and
  re-add the connector rather than just editing the header in place, if the
  UI doesn't offer an explicit reconnect.
