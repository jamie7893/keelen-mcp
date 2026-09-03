# Security model

## Bearer authentication

Every tool except `signup` and `verify_email` requires an
`Authorization: Bearer <api_key>` header on the MCP connection. The key
scopes every call to exactly one Keelen workspace (tenant) — there is no
cross-workspace access, and no way to enumerate or affect any workspace
other than the one the key belongs to.

## Reveal-once keys, rotation, and revocation

- `verify_email` mints a fresh API key and returns it in the response
  **exactly once**. Keelen never stores or displays the raw key again — only
  a hash. If you lose it, mint a new one (dashboard, or rerun the signup
  flow) and update your MCP client config.
- **A key minted over MCP passes through your agent's context window.**
  `verify_email` hands the raw key back as a tool result, so it enters the
  model's context and is written to whatever transcript, session file, or
  request log your MCP client keeps — and, if that client sends context to a
  hosted model, to that provider's logs under their retention policy. This is
  inherent to minting a credential through a language-model tool call, not a
  quirk of Keelen's implementation. Treat any transcript of the signup flow as
  holding a live credential: keep it out of shared or synced locations, and
  revoke the key (below) if it went somewhere you would not put a password.
  To avoid the exposure entirely, mint the key in the dashboard instead and
  paste it straight into your client config, so it never reaches a model.
- **On our side of the wire, the key is redacted.** Keelen API keys carry a
  `klk_` prefix, and every sink Keelen controls — application logs and error
  reports — strips any `klk_` value before writing it. That covers our
  infrastructure. It cannot cover your client's transcript, which is why the
  bullet above matters.
- Keys are individually revocable. Go to the Keelen dashboard →
  **Settings → API keys** and delete any key you no longer trust. Revocation
  is immediate — the next call using that key is rejected.
- Minting a new key never invalidates existing keys; each key is independent,
  so you can run multiple agents/clients against the same workspace with
  separate, separately-revocable keys.
- There is no key expiry by default — treat a Keelen API key like a
  password, and rotate it if you suspect it leaked (e.g. it ended up in a
  chat log, a committed file, or a screen share).

## What the MCP server can and can't reach

- **Tenant-scoped only.** Every tool call is resolved against the bearer
  key's workspace. There is no tool that reads or writes another workspace's
  data.
- **Your engine credentials never transit MCP.** Claude, Codex, GLM, and Kimi
  subscription credentials are connected exclusively through the Keelen
  dashboard (a browser session), never accepted as tool arguments or
  returned in tool responses. No MCP tool call can read, set, or forward
  those credentials.
- **GitHub access is explicit and browser-approved.** `connect_github`
  returns an install link; nothing is connected until the user approves the
  GitHub App install in their own browser. The resulting install is scoped
  to whatever repositories the user grants during that flow.
- **Builds run in an isolated per-tenant sandbox.** Once a project is
  provisioned, the autonomous loop's dev/QA work happens inside a sandbox
  scoped to that workspace — not on your local machine, and not shared with
  any other tenant.
- **Your content is treated as data, not instructions.** Code, issues, PR
  comments, and the requests the build agents read are wrapped in an explicit
  trust boundary — supplied as reference material describing *what* to build,
  never as instructions that could redirect the loop, change its permissions,
  or exfiltrate data. This is Keelen's defense against prompt injection from
  untrusted content in your own repository.

## Rate limits

The two unauthenticated tools carry their own limits (independent of any
workspace, since there's no bearer key yet to scope them to):

| Action | Limit |
| --- | --- |
| `signup` | 5 requests / hour / email, 20 requests / hour / IP |
| `verify_email` | 30 requests / hour / IP |
| 6-digit code attempts | 5 attempts per issued code (code is invalidated after) |

The authenticated tools that start billable machine work are capped per
workspace, to bound runaway agent loops and abuse:

| Tool | Limit |
| --- | --- |
| `create_project` | 10 calls / hour / workspace |
| `run_security_review` | 4 calls / hour / workspace |
| `run_legal_exposure_review` | 4 calls / hour / workspace |
| `run_control_gap_review` | 4 calls / hour / workspace |

Every cap sits well above normal interactive use. Past one, the tool returns
`ok: false` with `next_action: "wait"` and no `poll_after_seconds`, and starts
nothing — report it to the user rather than retrying. Every other
authenticated tool is a read or a row update, and inherits the workspace's
normal API rate limits.

## Reporting a security issue

Email **support@keelen.ai** with details. Please don't file public issues
for anything you believe is a security vulnerability.
