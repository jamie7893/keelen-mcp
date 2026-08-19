# Keelen MCP tools

Full reference for all 33 tools exposed by `https://keelen.ai/mcp`. Most
tool responses include `next_action` (and usually `next_step`) — follow it;
polling loops are your responsibility, the server does not push updates.

**Polling:** a `next_action` of `wait` is paired with `poll_after_seconds`
whenever there is a tool to poll — re-check that many seconds later (300 for
intake threads, 10 for provisioning). A `wait` carrying **no**
`poll_after_seconds` has nothing to poll over MCP. Four cases produce one: a
successful `run_security_review` (its findings land on the project's Security
review page rather than in a tool response), a successful
`run_legal_exposure_review` (same reason, on the project's Legal page), a
successful `run_control_gap_review` (same reason, on the project's Controls
page), and any rate-limited tool below. Report those to the user instead of
looping.

**Rate limits:** `create_project`, `run_security_review`,
`run_legal_exposure_review`, and `run_control_gap_review` each start billable
machine work, so every
workspace gets a bounded number of calls per hour — see [the table in `SECURITY.md`](SECURITY.md#rate-limits). Past the
limit the tool returns `ok: false` with `next_action: "wait"` and no
`poll_after_seconds`, and starts nothing. Tell the user and try again after
the hour rolls over. Every other tool is a read or a row update, and is
uncapped beyond your workspace's normal API limits.

## The `next_action` registry

`next_action` is **always a string**, drawn from this closed set. Structured
direction rides sibling fields (`next_tool`, `poll_after_seconds`) — the field
itself never changes type, so you can read it the same way on every tool.

| value | what it means | sibling fields |
| --- | --- | --- |
| `wait` | Nothing for you to do yet. | `poll_after_seconds` when there is a tool to poll; absent means report to the user and do **not** loop. |
| `answer_questions` | Intake needs clarification. | `questions` — call `answer_request` with one answer per question. |
| `done` | Terminal success. | — |
| `cancelled` | Terminal; nothing was produced. | — |
| `failed` | Terminal failure. | `intake_failure_reason` |
| `call_tool` | Call the MCP tool named in `next_tool`. | `next_tool` |
| `browser` | Send the user a URL to open in a browser. | the URL field named in `next_step` (`checkout_url` / `login_url` / `install_url`) |
| `collect_code` | Ask the user for the 6-digit code emailed to them. | — |
| `save_key_and_reconnect` | Store the reveal-once key in your client config, then reconnect. | `api_key` |
| `none` | Nothing to do. | — |

Thread tools (`submit_request`, `get_request_status`, `answer_request`,
`refine_request`) use only the first five values.

Two tools work with no key configured at all. Every other tool requires the
`Authorization: Bearer <api_key>` header described in [`SETUP.md`](SETUP.md)
and [`SECURITY.md`](SECURITY.md).

## Getting started (no account yet)

### `signup`

**Params:** `email: str`

Creates a Keelen account for `email` (or, if the email already has one,
silently reuses it — the response is identical either way, so this call also
works as agent-driven login for an existing user). Emails a 6-digit
verification code that expires in 15 minutes. Rate-limited per email and per
IP; call it again any time to resend a fresh code.

**Agents: ask the user for `email` and wait for their reply before calling
this.** Never infer the address from your client profile, the logged-in
account, `git config`, or any other ambient source; if you already hold a
candidate, echo it back and get an explicit yes first. Because this call also
logs an existing user in, a guessed address signs them in to whatever workspace
owns it, and the rest of setup then mints an API key on, and creates a project
in, an account they did not choose.

**Auth:** Unauthenticated.

### `verify_email`

**Params:** `email: str`, `code: str`

Redeems the 6-digit code from `signup` for a reveal-once workspace API key.
The code allows 5 attempts before it's dead — call `signup` again for a
fresh one if you run out. The returned `api_key` is shown exactly once and
must be stored only in your MCP client's own configuration (never in a repo
or file) as the `Authorization: Bearer <api_key>` header for this server;
reconnect after saving it.

**Auth:** Unauthenticated.

## Onboarding

### `get_onboarding_status`

**Params:** none

Reports where onboarding stands — `engine_connected`, `github_connected`,
`project_count`, `payment_status` — plus `next_action` / `next_tool` /
`next_step` telling you exactly what to do next: `next_action` is `call_tool`
(call the tool named in `next_tool`, following the `next_step` prose) at every
incomplete step, and `done` once onboarding is finished. Loop this between
every onboarding step until `step` reports `"done"`.

> **Deprecated:** before 2026-07-29 this tool returned `next_action` as an
> object (`{kind, tool, instructions}`) rather than a string — the only tool
> that did. That object is still echoed as `next_action_detail` for
> compatibility and **is removed on 2026-10-29**. Read
> `next_action` / `next_tool` / `next_step` instead. See
> [`CHANGELOG.md`](CHANGELOG.md). Also returns a `usage` block (monthly iteration pool used/included,
today's count vs the daily cap, machine-hours vs the monthly cap, and the
tier's concurrent-machines / scheduled-projects caps with the live
scheduled-project count) so capacity-aware clients can check headroom before
starting work; `0` means unlimited (`-1` for `scheduled_projects_cap`).

**Auth:** Bearer token.

### `open_dashboard`

**Params:** none

Mints a one-click, pre-authenticated dashboard sign-in link for the
workspace owner. Because this server already authenticated the owner, the
returned `login_url` (a `/login?token=…` deep link) signs the user straight
into the dashboard — no email round-trip, no password — and lands them where
onboarding left off (the engine-connect screen when that's the current step).
Use it whenever the user needs the dashboard: connecting an engine
subscription, updating a billing card, or opening a project page. The link is
single-use and expires in 15 minutes — call it again for a fresh one.

**Auth:** Bearer token.

### `connect_github`

**Params:** none

Mints a GitHub App install link (`install_url`) for the workspace. Send it
to the user to open in their browser; they pick the account/org and approve
the install, then land on a confirmation page. The link expires in 10
minutes — call this again for a fresh one. Poll `get_onboarding_status()`
afterward until `github_connected` is true.

**Auth:** Bearer token.

### `list_github_repos`

**Params:** none

Lists the repositories visible to the workspace's GitHub connection
(`full_name`, `default_branch`, `private`, `language`, `pushed_at` for each).
Use a returned `full_name` with `import_project`. Returns an error if GitHub
isn't connected yet — call `connect_github()` first.

**Auth:** Bearer token.

### `import_project`

**Params:** `repo_full_name: str`, `engine: str | None` (`claude_code` |
`codex` | `glm` | `kimi`, default `claude_code`), `build_description: str |
None`, `project_kind: str | None`, `stack: str | None`, `preview_command: str
| None`

Connects an existing GitHub repository (must be visible via
`list_github_repos`) as a Keelen project. This is the counterpart of
`create_project` — use `import_project` when the repository already exists,
and `create_project` only to scaffold a brand-new one.

`build_description` is optional but strongly recommended: it is submitted as
the project's first Request, so the loop has work. An imported project with no
Request sits idle until you call `submit_request(project_id, ...)`.

`project_kind` is optional and auto-detected when omitted. **Pass it when the
repository is a monorepo (apps in subdirectories) or uses a stack with no
standard root manifest (Java, Ruby, PHP, .NET, Elixir).** Detection resolves
those to `unknown`, and `unknown` blocks the project's dev lane until someone
overrides it. A value you pass is authoritative and is never overwritten by
later auto-detection. Allowed values: `library`, `node_library`,
`python_library`, `service`, `cli`, `web_app`, `godot_game`, `roblox_game`,
`unknown` (an explicit `unknown` is a deliberate reset that re-enables
auto-detection). `stack` is the optional language axis — `python`, `node`,
`rust`, `go`, `cpp` — for a kind that does not imply one.

`preview_command` is **required** when `project_kind` is `web_app` (the command
that serves the app locally, e.g. `npm run dev`); elsewhere it is optional and
overrides the detected command.

Idempotent — importing the same
repo twice returns the existing project with `already_exists: true` rather
than erroring. Follow the returned `next_step` and poll
`get_provisioning_status(project_id)` after the returned
`poll_after_seconds`. If your plan has no scheduled-project allowance the
project is still created but with the loop OFF, and `next_action` is
`call_tool` with `next_tool: "get_billing"` instead. (That arm also still
carries a deprecated `tool` key alongside `next_tool`; it is removed
2026-10-29 — see [`CHANGELOG.md`](CHANGELOG.md).)

**Auth:** Bearer token.

### `get_provisioning_status`

**Params:** `project_id: str`

Reports first-launch provisioning progress for a freshly created or imported
project: an overall status (`provisioning` | `ready` | `errored`), a
multi-stage checklist, and a human-readable error description if something
failed. Poll roughly every 10 seconds until `overall` is `"ready"`.

**Auth:** Bearer token.

### `get_billing`

**Params:** `plan: str` (`starter` | `pro` | `agency`, default `starter`)

Reports billing status and, when the workspace needs a new subscription,
returns a Stripe `checkout_url` — send it to the user to open in a browser
(the one setup step that can't happen in chat). Compute unlocks
automatically once payment completes; you don't need to block on it. If a
past payment failed (`payment_status` is `past_due`), this returns no
checkout link — the fix is to update the card on the dashboard billing page,
not to start a new subscription.

**Auth:** Bearer token.

## Projects & roadmap

### `list_projects`

**Params:** none

Lists the caller's Keelen projects (id, repo, status, kind, scheduler
on/off). Deleted projects are excluded, so a caller reconciling against
this listing never re-acts on a project it already removed.

**Auth:** Bearer token.

### `create_project`

**Params:** `name: str`, `build_description: str`, `project_kind: str`,
`private: bool` (default `true`), `preview_command: str | None`,
`org: str | None`, `engine: str | None`, `ci_runs_on: list[str] | None`,
`framework: str | None`

Scaffolds a brand-new GitHub repository from scratch, creates a bootstrap
Keelen project for it, and submits `build_description` as the project's
first Roadmap Request. `project_kind` is required (e.g. `web_app`,
`library`, `service`, `cli`, `godot_game`, `roblox_game`); `preview_command`
is required when `project_kind` is `web_app`. Pass `org` to create the repo
inside a GitHub organization instead of a personal account. Poll
`get_request_status` with the returned `thread_id`. On the rare arm where the
project was created but its first Request failed to submit, there is no
`thread_id` and `next_action` is `call_tool` with
`next_tool: "submit_request"` — call `submit_request(project_id, text)` with
the build description to start intake.

Pass `ci_runs_on` to point the scaffolded CI workflow at your own runners,
e.g. `["self-hosted", "linux", "x64", "my-fleet"]`. Omit it to inherit your
workspace default, which falls back to GitHub-hosted `ubuntu-latest`. Labels
that no registered runner in `org` carries are rejected up front, because
GitHub queues such a job indefinitely rather than failing it.

Pass `framework` to pick the frontend rails of a `web_app` — `vite` (the
default: a vanilla-TypeScript SPA) or `next` (a Next.js app-router app). The
scaffold ships the matching build config, entry shell, and dependencies, so the
first iteration starts from a repo that already builds and serves. `framework`
is only valid with `project_kind: "web_app"`; passing it with any other kind is
rejected.

For a `roblox_game` project, Keelen can auto-generate game assets (images,
audio, meshes, skyboxes, animated characters) from plain-language asset
requests — you describe the asset, the loop declares and generates it and wires
it into the game. See [`roblox-assets.md`](roblox-assets.md).

Rate-limited: **10 calls per hour per workspace**. Past the limit the call
returns `ok: false` with `next_action: "wait"` and creates nothing — no repo,
no project, no Request. Report it to the user; do not retry in a loop.

**Auth:** Bearer token.

### `archive_project`

**Params:** `project_id: str`

Archives a project (a reversible shelve) — stops any in-flight machine, then
drops the project out of the per-tier project cap (freeing a slot for a new
project) and stops the loop dispatching it. The project and its history are
kept and can be restored from the dashboard project page. Idempotent. Prefer
this over `delete_project` unless you specifically want the project gone.
Owner-scoped (an MCP key is owner-only); a project not in your workspace 404s.

**Auth:** Bearer token.

### `delete_project`

**Params:** `project_id: str`

Soft-deletes a project (the harder option) — stops any in-flight machine, then
removes it from `list_projects`, frees its slot, and stops the loop dispatching
it. The row is retained for audit, but unlike `archive_project` there is no
in-tool restore — re-import the repo to reconnect it as a fresh project.
Idempotent. Owner-scoped; a project not in your workspace 404s.

**Auth:** Bearer token.

### `submit_request`

**Params:** `project_id: str`, `text: str` (max 16,000 characters)

Submits one free-form Roadmap Request — a single feature or intent per
call — which enters the PM intake pipeline exactly like a web-submitted
request. Split a multi-feature ask into separate calls. Poll
`get_request_status` with the returned `thread_id`.

**Auth:** Bearer token.

### `get_request_status`

**Params:** `project_id: str`, `thread_id: str`

Reports intake status for a submitted request. Intake runs on an async
cadence (roughly every 5 minutes), so poll periodically rather than in a
tight loop. `next_action` is one of `wait`, `answer_questions`, `done`,
`cancelled`, or `failed` — the thread-tool subset of the
[`next_action` registry](#the-next_action-registry).

**Auth:** Bearer token.

### `answer_request`

**Params:** `project_id: str`, `thread_id: str`, `answers: list[{idx: int,
answer_md: str}]`

Answers a thread's clarifying questions (thread must be awaiting answers).
Answer every question exactly once, referencing the `idx` values from
`get_request_status`. Moves the thread back into intake; poll
`get_request_status` again afterward.

**Auth:** Bearer token.

### `refine_request`

**Params:** `project_id: str`, `thread_id: str`, `feedback_md: str`
(1–2000 characters)

Refines a completed thread's generated roadmap items with follow-up
feedback (maximum 5 refinements per thread). Poll `get_request_status` to
see the revised items.

**Auth:** Bearer token.

### `set_product_vision`

**Params:** `project_id: str`, `vision_md: str`

Sets a project's product vision, which is prepended to every PM/dev/QA
iteration prompt as project context.

**Auth:** Bearer token.

### `set_product_goal`

**Params:** `project_id: str`, `goal_md: str`

Sets a project's product goal, which is prepended to every PM/dev/QA
iteration prompt as project context alongside the vision.

**Auth:** Bearer token.

### `list_roadmap`

**Params:** `project_id: str`, `status: str` (default `queued`)

Lists a project's roadmap items filtered by status.

**Auth:** Bearer token.

### `reorder_roadmap`

**Params:** `project_id: str`, `ordered_ids: list[str]`

Reprioritizes a project's queued roadmap items to a new front-to-back order
(the first id becomes the highest priority — expanded into tasks next).
Items pinned to a specific point in time still dominate regardless of
position in `ordered_ids`; unknown or no-longer-queued ids are skipped.

**Auth:** Bearer token.

### `cancel_roadmap_item`

**Params:** `project_id: str`, `item_id: str`

Cancels / closes a single roadmap item (get its id from `list_roadmap`). Use
this to close a delivered or duplicate item that keeps re-parking — when the
work already shipped, every expand produces no dev-ready tasks and files a
recurring escalation. Cancelling drops the item out of the expand queue and
resolves any open expand-lane escalation for it. Idempotent for an already
expanded/cancelled item; refuses (409) while the item is actively being
expanded.

**Auth:** Bearer token.

### `clear_horizon_pin`

**Params:** `project_id: str`, `item_id: str`

Clears a queued roadmap item's horizon pin (now/next/later → none). A horizon
pin dominates the queue order, so `reorder_roadmap` cannot move a pinned item
out of its band; clearing the pin lets the item follow plain priority order
again. Only queued items carry a settable pin, so this refuses (422) on a
non-queued item — use `cancel_roadmap_item` to close a delivered item.

**Auth:** Bearer token.

### `rollback_roblox_place`

**Params:** `project_id: str`, `version_number: int`

Rolls a `roblox_game` project's place back to a previously-published version. It
re-publishes the retained build artifact for `version_number` (from the web
Roblox Publishing card) rather than rebuilding from source, minting a new Roblox
version that points at the old build. An unknown version returns 404; a version
with no retained artifact returns 422; a place open in Studio or rate-limited
returns 409 (retry); an invalid or unscoped Open Cloud key returns 409 / 403.

**Auth:** Bearer token.

### `project_status`

**Params:** `project_id: str`

Reports a project's open task count, iterations run today, verify pass
rate, open escalation count, pause state, and activity freshness — a
quick health snapshot for the loop.

**Auth:** Bearer token.

### `list_escalations`

**Params:** `project_id: str`

Lists a project's open escalations — the dead-ends that need a human
decision — each with its kind, reason, detail, and (when task-scoped) the
blocked task. Pass a returned `id` to `resolve_escalation`.

**Auth:** Bearer token.

### `resolve_escalation`

**Params:** `escalation_id: str`, `decision_md: str`

Marks an escalation handled, recording `decision_md` as an audit note.
Resolving a project-pause escalation resumes the project; other kinds are
simply acknowledged. Idempotent.

**Auth:** Bearer token.

### `control_scheduler`

**Params:** `project_id: str`, `action: str` (`enable` | `disable` |
`resume` | `process_now`)

Enables/disables a project's autonomous scheduler, resumes it from a pause,
or forces the next intake batch to run immediately. Returns the resulting
scheduler state.

**Auth:** Bearer token.

### `run_security_review`

**Params:** `project_id: str`

Kicks off a deep, whole-repo security review: a one-shot audit machine scans
the repo across a project-kind-aware taxonomy (secrets + git history,
vulnerable/abandoned dependencies, injection, SSRF, path traversal, unsafe
deserialization, crypto misuse, info-leak, plus web-app authz/session/CORS,
library API-misuse, game client-trust, or infra/CI modules as applicable) and
posts findings to the project's Security review. Findings are surfaced for human
triage — you review them and choose which to send into the loop as Requests; no
fix is applied automatically. Billable; one audit in-flight per project.
Triggering is disabled while the feature is hardened for production: it is gated
to an operator allowlist (empty by default), so a project that is not enabled
returns a message instead of spawning. Results from earlier reviews stay
visible.

This tool returns `next_action: "wait"` with **no** `poll_after_seconds` —
there is no MCP tool that reports review progress. Tell the user the review
started and point them at the project's Security review page; do not loop.

Rate-limited: **4 calls per hour per workspace**. Past the limit you get the
same `next_action: "wait"` shape, but with `ok: false` and no review started.
Read `next_step` to tell the two apart.

**Auth:** Bearer token.

### `run_legal_exposure_review`

**Params:** `project_id: str`

Starts a legal exposure review of a project's repository. A one-shot review
machine reads the checkout offline and maps what the code **does** onto
commonly cited legal obligations, filtered by the project's saved compliance
profile (its jurisdictions plus eleven product facts). Findings land on the
project's Legal page for a human to triage, and `get_legal_exposure_findings`
reads them. No change is ever applied automatically.

**This is not legal advice and it is not a legal clearance.** The result
carries a `disclaimer_md` field. Read that text to the user before you
summarise anything the review produced.

The project needs a saved compliance profile, a plan that carries the feature,
and a place on the operator allowlist (empty by default, because the lane is
still in a staged rollout). Billable; one review in flight per project. Every
refusal comes back as `ok: false` with an actionable `next_step`, and starts
nothing.

This tool returns `next_action: "wait"` with **no** `poll_after_seconds` —
there is no MCP tool that reports review progress. Tell the user the review
started, then read the findings later; do not loop.

Rate-limited: **4 calls per hour per workspace**. Past the limit you get the
same `next_action: "wait"` shape, but with `ok: false` and no review started.
Read `next_step` to tell the two apart.

**Auth:** Bearer token.

### `get_legal_exposure_findings`

**Params:** `project_id: str`, `limit: int = 25` (max 100),
`include_all: bool = false`

Reads a project's legal exposure findings, worst exposure first. Each finding
is an **observation** about the code, plus the obligation commonly cited over
that pattern, its citation, the date the citation was last checked, and
`counsel_reviewed` (false today for every registry entry). `exposure_order` is
an **order**, never a score: this output carries no grade, no percentage, and
no overall state.

Read the `not_determinable` list before you summarise. Those are checks that
could not reach a verdict, and dropping them turns a partial review into a
clean answer. An empty findings list is never proof that anything is in order,
and the `disclaimer_md` field must reach the user.

`include_all` adds resolved, dismissed, and out-of-scope rows to the default
actionable set. Read-only: no compute, no rate limit, and it keeps working
after a project leaves the trigger allowlist, so a project never loses its own
history.

**Auth:** Bearer token.

### `run_control_gap_review`

**Params:** `project_id: str`

Starts a security control gap review of a project's repository evidence. The
user first saves a framework selection and stack context. A one-shot review
records engineering observations under Cyber Essentials or CMMC Level 1 or
Level 2 context. It makes no code change automatically.

**This does not determine an organisation's standing under a security
framework.** The result carries `disclaimer_md`. Read that text to the user
before you summarise any observation. A missing observation does not establish
anything about a control.

The project needs a saved framework profile, an eligible plan, and a place on
the operator allowlist (empty by default). Billable; one review can run at a
time for a project. Refusals return `ok: false` with a next step and start
nothing.

This tool returns `next_action: "wait"` with **no** `poll_after_seconds`.
Tell the user the review started, then use `get_control_gap_findings` later;
do not loop.

Rate-limited: **4 calls per hour per workspace**. Past the limit you get the
same `next_action: "wait"` shape, but with `ok: false` and no review started.

**Auth:** Bearer token.

### `get_control_gap_findings`

**Params:** `project_id: str`, `limit: int = 25` (max 100),
`include_all: bool = false`

Reads security-control observations grouped by their evidence class, along
with records that the review could not determine and controls outside this
review's visibility. The response has no total and no overall framework
outcome. Read `not_determinable`, `outside_review_scope`, and `disclaimer_md`
before you summarise an observation. An empty evidence group does not establish
that a control is in place.

`include_all` adds resolved, dismissed, and out-of-scope records to the default
actionable set. Read-only: no compute quota, and it keeps working after a
project leaves the trigger allowlist.

**Auth:** Bearer token.
