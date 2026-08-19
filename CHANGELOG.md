# Changelog

Notable changes to the Keelen MCP server's public contract. Newest first.

Dates are when the change reached `https://keelen.ai/mcp`. Deprecations name
the date the old shape is removed; nothing is removed without appearing here
first.

## 2026-08-07 — rotated workspace tokens now carry the `klk_` prefix

**What changed.** Rotating your workspace token in the dashboard
(**Settings → MCP token → Rotate**) now returns a key that starts with `klk_`,
matching every other Keelen key. It previously returned a key with no prefix.

**Why it matters.** The prefix is what makes a key redactable. Keelen's own
logs and error reports strip any `klk_` value before writing it, so an
unprefixed key was the one shape that could survive into them. Rotating now
also upgrades you onto the redacted shape.

**Do you need to do anything?** No. Keys minted before this change keep working
until you rotate them, and nothing about how you send the key changed. If you
want the redaction benefit on an older key, rotate it once.

## 2026-08-06 — API keys are redacted from Keelen's logs and error reports

Nothing changed in the API. Keelen keys carry a `klk_` prefix, and every sink
Keelen controls now strips any `klk_` value before writing it — so a key can no
longer reach our application logs or our error-reporting service by riding
along in an exception or a stack frame. Your own client's transcript is still
outside our reach; see
[`SECURITY.md`](SECURITY.md#reveal-once-keys-rotation-and-revocation).

## 2026-08-06 — hourly caps on the two billable tools

**What changed.** `create_project` (10/hour) and `run_security_review`
(4/hour) are now capped per workspace — see
[`SECURITY.md`](SECURITY.md#rate-limits). Past a cap the tool returns
`ok: false` with `next_action: "wait"` and no `poll_after_seconds`, and starts
nothing at all.

**Why.** Both spawn real machine work and neither had a throughput cap.
`create_project`'s only ceiling was the total per-tier project count, and
`run_security_review`'s one-in-flight guard still allowed unbounded
back-to-back audits — so an agent retry loop could burn compute as fast as it
could call. Every other tool is a cheap read or a row update and is unaffected.

**Client action:** none for normal use — the caps are far above interactive
rates. If your client retries on a falsy `ok`, make it stop on
`next_action: "wait"` without `poll_after_seconds` and report to the user
instead. That is the same shape `run_security_review` already returned.

## 2026-08-06 — the key `verify_email` returns is a transcript surface

Documentation only; nothing changed in the server. [`SECURITY.md`](SECURITY.md#reveal-once-keys-rotation-and-revocation)
now says plainly that an API key minted over MCP passes through the model's
context window and lands in your client's transcript and any hosted provider's
logs, and what to do about it — including minting in the dashboard instead if
you would rather the key never reach a model.

## 2026-08-05 — `import_project` accepts a project kind

**What changed.** `import_project` gained three optional params —
`project_kind`, `stack`, and `preview_command` — and `get_onboarding_status`
now points the repo step at `list_github_repos` instead of `create_project`.

**Why.** Import detected a repository's kind from two root files
(`package.json`, `pyproject.toml`). Everything else resolved to `unknown`,
which blocks a project's dev lane. Several stacks recovered on their own once
the first setup run inspected the checkout, but a **monorepo** (detection is
root-only by design) and any stack without a standard root manifest (Java,
Ruby, PHP, .NET, Elixir) did not — and there was no way to correct the kind
over MCP, so those imports needed the web dashboard to finish. Passing
`project_kind` now settles it at import time, authoritatively.

Separately, `get_onboarding_status`'s repo step named `create_project` in its
machine-readable `next_tool` while mentioning import only in prose. A client
following `next_tool` literally — which the server instructions ask for —
therefore always scaffolded a **new** repo, even for a user who already had
one. That step now names `list_github_repos`, whose result tells you which
branch applies; `next_step` spells out both.

**Client action:** none required. The new params are optional and omitting
them preserves the previous behaviour exactly. If your client hardcodes an
expectation that the repo step's `next_tool` is `create_project`, read
`next_tool` dynamically instead.

## 2026-07-29 — protocol revision `2026-07-28` is served

The server now speaks MCP protocol revision **`2026-07-28`** alongside the
earlier handshake revisions, from the same endpoint, selected per request from
your `MCP-Protocol-Version` header.

**No client action is required** — existing configurations keep working
unchanged and every tool behaves identically on both. Clients on the new
revision additionally get `server/discover` and a cacheable `tools/list`. See
[`clients/generic.md`](clients/generic.md#protocol-support).

## 2026-07-29 — `next_action` is a string on every tool

**What changed.** `next_action` is now guaranteed to be a **string** on every
tool, drawn from a documented closed set — see
[the `next_action` registry](TOOLS.md#the-next_action-registry). Two responses
previously fell outside that shape:

- **`get_onboarding_status`** returned `next_action` as an **object**
  (`{kind, tool, instructions}`) — the only tool of 29 that did, so a client
  reading `next_action` uniformly broke on exactly the tool that drives
  onboarding. It now returns `next_action: "call_tool"` (or `"done"` when
  onboarding is finished), the tool to call in `next_tool`, and the prose in
  `next_step`.
- **`import_project`** (loop-off arm) and **`create_project`**
  (first-Request-failed arm) put a *tool name* where an action belongs
  (`next_action: "tool"` with a `tool` key; `next_action: "submit_request"`).
  Both now return `next_action: "call_tool"` with `next_tool`.

**What you need to do.** If you read `next_action` as a string and follow
`next_tool` / `next_step`, nothing. If you special-cased the
`get_onboarding_status` object, migrate to `next_action` / `next_tool` /
`next_step`.

**Compatibility (removed 2026-10-29).** For the migration window, the old
shapes still ride along:

| deprecated field | on | replaced by |
| --- | --- | --- |
| `next_action_detail` (the old object) | `get_onboarding_status` | `next_action` + `next_tool` + `next_step` |
| `tool` | `import_project` | `next_tool` |

Both are removed on **2026-10-29**.
