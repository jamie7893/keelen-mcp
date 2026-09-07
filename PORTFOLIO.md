# Keelen portfolio surface

The portfolio surface at `https://keelen.ai/portfolio/mcp` is the Bot-facing
tenant-management plane behind the Keelen Portfolio Manager. It is not
generally available yet — this document describes the interface it will
expose once it opens up. It accepts project-scoped OAuth grants only — a
workspace API key is refused — and it exposes every authenticated tool from
[`TOOLS.md`](TOOLS.md) (except `signup` and `verify_email`) under one policy
table plus the tools below.

Every tool carries a `Policy:` line in its description: `allow_read` runs
directly; `allow_recorded` needs `record_decision` first and a stable
`intent_key`; `owner_decision` needs a card the workspace owner answered in
the Keelen dashboard; `hard_deny` is listed so an agent can explain it and is
refused on call. A project outside the grant's allowlist answers 404. The
same `intent_key` with the same arguments replays the first result while its
receipt is retained; with different arguments it is refused (409).

Terminal decision and intent payloads are eligible for compaction after 90 days.
Live decisions and pending/unresolved actions retain their evidence, including
linked records. Historical decision results expose `payload_redacted_at` when
details have been removed. Compact keys, digests and provenance remain: this
bounds payload retention and sweep work, **not lifetime row count**.

After receipt compaction, an exact retry returns HTTP 409
`intent_result_expired`, including for a previously failed action:

> This action's detailed receipt has expired. This key will not execute again.
> Read current project state before deciding whether further work is needed.

The old key never becomes executable again. Changed arguments still return
`intent_conflict`. Receipt expiry is not an unknown outcome or a successful
empty result; do not blindly choose a new key to repeat the action.

Third-party Bot template built by Keelen. Not created, sponsored, endorsed,
or operated by xAI.

The initial release is manual, for existing workspace owners and explicitly
chosen existing public projects. Keep scheduled Bot routines disabled.
`create_project` and `import_project` are hard-denied on this mount while
disposable sandbox creation is deferred. Policy version 2 requires fresh
owner consent for blueprint preview/import. These changes do not affect the
primary MCP server's project-creation tools.

## The portfolio-only tools

These fourteen tools exist only on this surface — the primary `https://keelen.ai/mcp`
server does not register them. But the two keyword parameters below are not
scoped to these fourteen: every `allow_recorded` or `owner_decision` write on
this surface accepts them — the tools below AND every re-exposed primary
write such as `submit_request` or `close_task`. Both
parameters are added by this surface's governance wrapper:

- `intent_key: str` — a stable key you choose per intended action. Required
  on every write; the same key with the same arguments replays the first
  result instead of running twice until receipt compaction (then
  `intent_result_expired`), and the same key with different arguments is
  refused (409).
- `decision_id: str | None` — the id `record_decision` returned for this
  exact project, tool, and arguments. Required (400 if missing) on every
  write **except** `record_decision`, `open_decision`, and `preview_blueprint`, which
  ignore it — they are how a decision gets filed in the first place, so they
  need only `intent_key`.

### `preview_blueprint`

**Params:** `project_id: str`, `blueprint: dict`, `mapping: dict[str, str]`,
plus `intent_key` (required) and `decision_id` (accepted, ignored).

Validates and normalizes a version 1 blueprint, maps client-local project IDs
to explicitly chosen existing Keelen projects, and opens an owner approval
card. It creates no projects, changes no project brief and files no Requests.
Every mapped project must be in this connection's grant, public with a fresh
visibility check, active, and without an enabled native Owner agent. The
anchor `project_id` must appear among mapped targets. Unknown mapping keys
and duplicate targets are refused. Unmapped blueprint entries are shown as
omitted and will not be imported.

The blueprint has `schema_version: 1` and 1–25 `projects`. Each project has a
unique lowercase slug `local_id` (1–64 characters), `name` (1–120), optional
`vision`, `goal`, and `seven_day_plan` (each at most 4,000), up to ten
`owner_rules` (each 1–500), `priority` (1–100, default 1), and up to two
`requests` with `title` (1–120) and `text`. The composed Request
`title + "\n\n" + text` must be at most 16,000 characters. Unknown fields
and blank required text are refused. Omitted optional prose stays unchanged
on import; a provided empty vision or goal clears that value.

Visions and goals replace provided fields; Requests enter the planning queue.
Priorities, seven-day plans and owner rules are planning material only, with
no automatic scheduling or rule activation.

**Policy:** `allow_recorded` (bootstrap meta-write).

**Result keys:** `blueprint`, `mapping`, `omitted_local_ids`,
`import_arguments`, `decision_id`, `status`, `expires_at`,
`planning_only_fields`, `next_action`, `next_step`.

**`next_action` values:** `wait`. Show the preview to the owner and stop.

### `import_blueprint`

**Params:** the preview's exact `import_arguments` (`project_id`,
`preview_token`, normalized `blueprint`, normalized `mapping`), plus
`decision_id` (the preview's owner-approved card) and `intent_key` (required).

After the owner approves in Keelen Settings or Decision cards, imports the
exact preview once. Changing the blueprint, mapping, anchor or preview token
requires a new preview and approval. A generic decision card does not replace
a server-created blueprint preview. Revoked or expired connections and
projects that have become ineligible are refused before changes.

Mapped brief changes, all Requests, the durable import receipt, the completed
intent and card consumption commit together. Every Request counts toward the
project's daily limit; the mutation window counts every mapped Request and
provided vision or goal replacement, with a minimum cost of one per import.
Split an oversized batch into smaller separately approved previews. A second
intent cannot spend the same approval. Preserve the original intent key for
an exact retry. An unresolved outcome must be reconciled against current
state; do not blindly retry it under a new key.

The owner-only Settings activation status uses the minimal import receipt
after detailed intent payloads expire. It contains IDs, counts and a digest,
not blueprint text. Import success records planning work and never proves
product delivery.

**Policy:** `owner_decision`.

**Result keys:** `import_id`, `status`, `projects` (each `local_id`,
`project_id`, `request_ids`), `omitted_local_ids`, `planning_only_fields`,
`next_action`, `next_step`.

**`next_action` values:** `done`.

### `record_decision`

**Params:** `project_id: str | None`, `rationale_md: str`, `refs: list[str]`,
`tool: str`, `canonical_args: dict`, `category: str | None = None`, plus
`intent_key` (required) and `decision_id` (accepted, ignored).

Records WHY you are about to call `tool` with exactly `canonical_args` on
`project_id`. `project_id` may be omitted only when `tool` is `create_project`
or `import_project`. The binding is exact: a later call to `tool` must carry
the same `project_id`, `tool` name, and arguments (excluding `decision_id`
and `intent_key`) or the mutation is refused. Valid for one hour, single use.

**Policy:** `allow_recorded` (a bootstrap meta-write — see above).

**Result keys:** `decision_id`, `status`, `expires_at`, `next_action`,
`next_tool`, `next_step`.

**`next_action` values:** `call_tool`.

### `open_decision`

**Params:** `project_id: str | None`, `question_md: str`,
`options: list[dict]`, `recommendation: str | None`, `evidence: list[str]`,
`tool: str | None = None`, `canonical_args: dict | None = None`, plus
`intent_key` (required) and `decision_id` (accepted, ignored).

Asks the workspace owner and stops. Only a human answers, in the Keelen
dashboard — this surface has no answer operation. Pass `tool` and
`canonical_args` when the card authorises one exact mutation (its options
become approve/reject); omit them for an informational question with up to
four options of your own. `project_id` may be omitted only when `tool` is
`create_project` or `import_project`. Open for seven days.

**Policy:** `allow_recorded` (a bootstrap meta-write — see above).

**Result keys:** `decision_id`, `status`, `expires_at`, `next_action`,
`next_step`.

**`next_action` values:** `wait`.

### `list_decisions`

**Params:** `project_id: str | None = None`, `status: str | None = None`.

Lists the open and answered cards, and the recorded decisions, visible to
this connection. Poll this rather than looping on an open card.

**Policy:** `allow_read`.

**Result keys:** `decisions`, `next_action`.

**`next_action` values:** `done`.

### `portfolio_digest`

**Params:** none.

Every project this connection manages, with only the items that need action:
the server-held charter (version, digest, and text), and per allowlisted
project its status, scheduler state, pause reason, counts (`awaiting_answers`,
`failed_requests`, `open_escalations`, `queued_items`, `machines_in_flight`,
`open_cards`, `answered_cards`), and a bounded, ordered attention list. Call
this once per management run and stop once `attention_total` is 0.

**Policy:** `allow_read`.

**Result keys:** `generated_at`, `charter` (`version`, `digest`, `md`),
`projects` (each with `project_id`, `repo`, `status`, `scheduler_enabled`,
`scheduler_paused_until`, `pause_reason`, `counts`, `attention`),
`attention_total`, `next_action`, `next_step`.

**`next_action` values:** `done`, `none`.

### `list_requests`

**Params:** `project_id: str`, `status: str | None = None`, `limit: int = 50`
(clamped to at most 200).

Every Request thread on one allowlisted project, newest first.

**Policy:** `allow_read`.

**Result keys:** `requests` (each with `thread_id`, `status`, `created_at`,
`first_line`, `source`, `client_label`, `questions_open`,
`generated_roadmap_item_ids`, `intake_failure_reason`), `next_action`.

**`next_action` values:** `done`.

### `get_roadmap_item`

**Params:** `project_id: str`, `item_id: str`.

One roadmap item on an allowlisted project, plus its tasks and each task's
delivery state. 404s if the item belongs to a different project than
`project_id` names.

**Policy:** `allow_read`.

**Result keys:** `item` (`id`, `title`, `status`, `priority_int`,
`estimated_complexity`, `review_state`, `created_at`, `expanded_at`), `tasks`
(each with `id`, `title`, `status`, `github_pr_url`, `pr_number`,
`delivery_state`, `delivery_label`, `landed`, `merged_at`,
`pr_closed_unmerged_at`, `completed_at`), `next_action`.

**`next_action` values:** `done`.

### `list_pull_requests`

**Params:** `project_id: str`, `limit: int = 20` (clamped to at most 100),
`include_ci: bool = True`.

Every task with a PR on one allowlisted project, newest-pushed first, with
delivery state and a bounded, best-effort live CI check summary per PR (a
merged PR's CI is settled history and is never looked up live). CI reads use
a separate, tenant-owned single-repository installation token with only
`metadata:read`, `checks:read`, and `statuses:read`. If either check runs or
commit statuses cannot be read completely, CI is unavailable (`ci: null`),
never an incomplete green summary. No broader credential fallback is used.

**Policy:** `allow_read`.

**Result keys:** `pull_requests` (each with the `get_roadmap_item` task
fields plus `task_id`, `pr_url`, `head_sha`, `pushed_at`,
`closed_unmerged_at`, `verify_status`, `ci`, `ci_error`), `merge_policy`,
`next_action`.

**`next_action` values:** `done`.

### `list_machines`

**Params:** `project_id: str | None = None` (omit for every allowlisted
project at once).

In-flight machines across every allowlisted project, or one of them.

**Policy:** `allow_read`.

**Result keys:** `machines` (each with `id`, `project_id`, `role`, `state`,
`started_at`, `last_heartbeat_at`), `next_action`.

**`next_action` values:** `done`.

### `get_loop_activity`

**Params:** `project_id: str`, `after: str | None = None` (an OPAQUE cursor —
pass back a previous call's `next_after` verbatim; only entries after it are
returned), `limit: int = 100` (clamped to at most 500).

The Loop Activity feed for one allowlisted project, oldest-of-the-page
first, rendered the same way the dashboard's Insights tab renders it, and
filtered to a customer-safe action whitelist.

**Policy:** `allow_read`.

**Result keys:** `lines` (each with `at`, `action`, `line`), `next_after`,
`next_action`.

**`next_action` values:** `done`.

### `get_project_usage`

**Params:** `project_id: str`, `days: int = 30` (clamped to 1-90).

A rollup of iterations run on one allowlisted project over a bounded
trailing window.

**Policy:** `allow_read`.

**Result keys:** `window_days`, `iterations_by_role`, `tokens_in`,
`tokens_out`, `cost_usd_estimate`, `machine_minutes`, `next_action`.

**`next_action` values:** `done`.

### `get_definition_of_done`

**Params:** `project_id: str`.

The only completion answer this surface gives for one allowlisted project.
`met` is `true` only on a fresh, enabled-Owner attestation — fresh meaning
it postdates the latest tracked activity — with every tracked count STILL
zero right now, at read time (not merely when the attestation was made). A
project managed through this Bot-facing mount can never carry that
attestation: a project with an enabled Owner agent is refused before this
tool ever runs (one manager per project), so `basis` is always `rows` and
`met` is always `false` here. Read `all_tracked_work_landed` and `unmet`
instead, and never tell the workspace owner a product shipped on a `rows`
basis. A closed-unmerged PR, or a task whose delivery cannot be confirmed on
its merge policy, is never counted as landed. A successful call to any
other tool on this surface is never evidence a product shipped.

**Policy:** `allow_read`.

**Result keys:** `met`, `basis` (`rows` or `owner_lane`),
`all_tracked_work_landed`, `unmet`, `counts` (`open_threads`,
`queued_or_parked_items`, `unlanded_tasks`, `open_escalations`,
`failed_threads`, `closed_unmerged_tasks`, `unknown_delivery_tasks`),
`next_action`, `next_step`.

**`next_action` values:** `wait` (not complete — never `none`, which means
something different on `portfolio_digest`). `done` is a real value this
tool's code can emit on an `owner_lane` `met=true` verdict, but it is
UNREACHABLE through this Bot-facing mount today: that basis needs an
enabled Owner agent, and this mount refuses any project that has one before
this tool ever runs.

### `set_sprint_goal`

**Params:** `project_id: str`, `goal_md: str`, plus `intent_key` (required)
and `decision_id` (required — this is a non-bootstrap write).

Replaces the goal of the project's open sprint. If no sprint is open, nothing
is written and `next_step` says to call `set_product_goal` instead.

**Policy:** `allow_recorded`.

**Result keys:** `sprint_id`, `sprint_goal_md`, `next_action`, `next_step`
(present only when no sprint was open).

**`next_action` values:** `done`, `none`.
