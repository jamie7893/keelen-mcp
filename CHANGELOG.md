# Changelog

Notable changes to the Keelen MCP server's public contract. Newest first.

Dates are when the change reached the named hosted MCP surface. Deprecations name
the date the old shape is removed; nothing is removed without appearing here
first.

## Unreleased — Manual recipient setup clarification

The [Portfolio Manager guide](PORTFOLIO-MANAGER.md) separates connecting existing
projects, checking progress and making a supported goal edit from optional
blueprint import. It explains GitHub repository selection, the Keelen workspace
on OAuth consent, recorded decisions versus owner approval cards, and a useful
follow-up when the attention digest is quiet. The walkthrough used two existing
fictional projects on an existing owner account; it is not fresh-account
acceptance. This documentation update does not enable execution or routines.

## 2026-09-08 — Portfolio Manager 0.3 is available

The [manual recipient guide](PORTFOLIO-MANAGER.md) now describes the accepted
0.3 template configuration: five generic skills, no scheduled routines,
recipient-owned accounts and explicit policy 3 consent for selected existing
public or private projects. Owners review blueprint previews and approve exact
imports in Keelen Settings. The template share link and public availability are
confirmed: [open Keelen Portfolio Manager](https://x.ai/bot/iCqviUzbFmWEKAdNqkXKm)
and follow the guide to connect your own account.

This is the separate `https://keelen.ai/portfolio/mcp` surface. It does not change
the primary MCP setup or enable the separately merged optional 0.4 delegated
actions. Those actions require additional owner permission and deployed
availability; manual management reads use selected-project consent. This entry
makes no claim of native acceptance for a 0.4 template or routine.

## 2026-09-03 — the ui-review scenario budget is settable

**What changed.** `set_ui_review_scenario_cap` was added. It sets a `web_app`
project's ui-review scenario budget, or resets it to the default of 12, without
requiring an open escalation.

**Why it matters.** The budget had one writer:
`resolve_platform_policy_conflict` with `raise_project_cap`, which needs an open
platform-policy card on a blocked task. A project could be repaired but never
configured, so an operator watching the manifest fill had to wait for a task to
fail against the cap and spend an iteration producing nothing before the budget
could move at all.

Raising always succeeds, including from a full manifest. Lowering is refused
when the default branch already exceeds the smaller budget on either the
scenario or the screenshot axis, and refused when that manifest cannot be read:
both caps are enforced at push time and not in your CI, so a manifest left over
cap fails every push while CI stays green.

**Compatibility.** Additive. `resolve_platform_policy_conflict` is unchanged and
remains the right call when a platform-policy card is open, because it also
replans and requeues the blocked task.

## 2026-08-31 — closed tasks stop counting as open work

**What changed.** `project_status.open_tasks` no longer counts a `cancelled`
task. `close_task` now echoes `open_tasks` in its response, counted after the
close commits.

**Why it matters.** `close_task` writes the `cancelled` status and commits, but
`open_tasks` counted everything that was not `done`, so it reported the same
number immediately before and immediately after a successful close. The write
had always landed; the count had never learned the status. With no other
number moving, a close that did nothing looked exactly like one that worked —
including the second, idempotent close of a ticket already closed. The echoed
count makes the effect verifiable from the response alone.

**Compatibility.** Additive. `open_tasks` drops by the number of tasks closed
since a caller last read it; a caller that stored an earlier value should
re-read rather than compare across the change.

## 2026-08-31 — escalation recovery transitions are explicit and atomic

**What changed.** Three idempotent tools were added:
`rearm_roadmap_item`, `resolve_platform_policy_conflict`, and `replan_task`.
`list_escalations.next_tool` now routes each card to the operation that actually
moves its task or roadmap item. `resolve_escalation` no longer acknowledges a
blocked task's final visible recovery card or a platform-policy conflict.

**Why it matters.** An acknowledgement used to hide a Needs You card while its
task stayed blocked. Roadmap dependency parks could be re-armed only from the
dashboard, and stale dependency wording could outlive the exact prerequisite
that had already merged. The new operations share the dashboard's locked state
transitions, use structural lineage and exact PR identities, and keep the card
open if its corresponding transition fails.

**Compatibility.** Additive except for the intentional fail-closed change to
`resolve_escalation`. Clients should follow each row's `next_tool`; a 409 on an
old generic acknowledgement means the task still needs its typed transition.

## 2026-08-28 — `project_status` reports delivery and planning-lane health

**What changed.** `project_status` gains two fields. `prs_merged_today` counts
the distinct pull requests whose merge landed today. `pm_lane` is
`{last_pm_iter_at, repo_access}`, where `repo_access` is `"ok"` or `"failing"`.

**Why it matters.** `pass_rate` is a local verification measure over the last
14 days of dev/QA runs and is independent of merges, so a project shipping
every day could read a low rate with nothing in the tool saying what actually
shipped. And the planning lane's repository reads fail quietly: a run that saw
nothing still finishes successfully and still moves `last_pm_iter_at` forward,
so a planning lane locked out of GitHub looked healthy on every field.

**Caveats to read with them.** `prs_merged_today` counts a merge timestamp that
is only recorded for projects on the automatic merge policy; on manual or
branch-only projects it reads 0 even while pull requests are merged by hand.
`get_onboarding_status`'s `github_connected` remains a PRESENCE flag — read
`pm_lane.repo_access` for live health.

**Compatibility.** Both fields are additive; existing clients do not need to
change. `pass_rate` is unchanged and is now documented for what it measures.

## 2026-08-28 — read the product goal and vision before you replace them

**What changed.** `get_product_goal(project_id)` and
`get_product_vision(project_id)` return the stored document and the timestamp
it was last set. Both are read-only.

**Why it matters.** Only the setters existed, and `set_product_goal` /
`set_product_vision` REPLACE the whole document rather than appending to it. An
agent with no way to read first could only overwrite blind, silently discarding
what the user had already recorded. Read, edit, then set the full result back.

**Compatibility.** Additive; existing clients do not need to change.

## 2026-08-28 — close a task that should not be built

**What changed.** The additive `close_task(project_id, task_id, reason,
superseded_by_task_id?, superseded_by_pr_number?)` tool closes a duplicate or
already-shipped task. The task is marked `cancelled` and keeps its acceptance
criteria, QA steps, and iteration history — this is not a delete.

**Why it matters.** There was no way to close a task at all: the board could
only move a task to `done` (which requires a merged pull request) or delete it
outright, which destroys the record of what was asked for. Duplicates therefore
stayed open, and unfinished tasks count against a project's planning capacity,
so they quietly stopped new roadmap items from being expanded.

**Operator confirmation.** Closing takes work off the board, so confirm with
the operator first. Pass a superseding PR number or task id whenever the work
really shipped elsewhere; that records provenance rather than a bare abandon.
A close does not claim the content reached the default branch, so tasks that
declared a dependency on the closed one keep waiting.

**Compatibility.** Additive; existing clients do not need to change. Replaying
the same close is a no-op.

## 2026-08-26 — explicitly retry a recovery-budget-blocked task

**What changed.** The additive `retry_blocked_task(escalation_id, decision_md)`
tool returns one eligible blocked task to the queue after you fix its recorded
recovery cause. `list_escalations` now reports server-derived
`task_retry_available` and routes each card to the correct tool. Eligibility is
rechecked under locks: the card must be an open `recovery_budget_exhausted_*`
task block, the task must still be blocked, and no other task blocker may remain.

**Operator confirmation.** This action spends compute and grants exactly one
task a fresh recovery episode. Confirm the retry with the operator. Merely
acknowledging a card with `resolve_escalation` remains non-retrying and removes
that card's retry authority.

**Compatibility.** This is additive; existing clients do not need to change.
Response-loss replay is idempotent and returns `requeued: false` after the first
successful retry.

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
