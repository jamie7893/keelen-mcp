# Keelen Portfolio Manager 0.4.1 manual workflow

Submit a brief to your public and private Keelen projects, make a supported
goal change and return to the same work for current records and owner decisions.
One eligible project is enough. Connect your own Keelen account and choose the
projects it can access. Blueprint imports are optional and need their own
approval in Keelen Settings. Keelen performs engineering under each project's settings.
This release runs when you ask; it includes no scheduled routines.

**Published September 10, 2026.** Open
[Keelen Portfolio Manager 0.4.1](https://x.ai/bot/gvegGULyBESU9brTeOM_r).
The Portfolio connection is available for existing workspace owners using their
own accounts and selected-project consent.

The published 0.4.1 instruction package clarifies briefs, charter and budget
checks, and returning to the same work. It keeps management manual by default.
The new public template is native version 1; 0.4.1 identifies its instruction
package. Existing 0.3 copies are not upgraded automatically. Review the actual
configuration you add.

The 0.4.1 template uses five generic, versioned skill names:
`portfolio-v041-interview`, `portfolio-v041-charter`,
`portfolio-v041-blueprint`, `portfolio-v041-manage` and
`portfolio-v041-report`. The versioned names preserve older skills in the
account's shared library. They do not select projects or supply an account.
Older 0.3 copies use unversioned names. The new link above supplies 0.4.1.

Third-party assistant template built by Keelen. The assistant platform does not
create, sponsor, endorse or operate this template. Review the full configuration
before adding it. No creator account or credentials are supplied.

Keelen permits anyone to use, copy and share this template, subject to applicable
platform terms. Each recipient must connect their own accounts. This permission
covers the template, not customer data or Keelen's private source repository.

## What you need

- An existing Keelen workspace where you are an active owner, with the existing
  projects you want to manage and current GitHub access to their repositories.
- Your own Grok Bot access, the installed desktop app, and access to its
  template and OAuth connection features. Check the provider's current
  [access requirements](https://docs.x.ai/grok-bot/get-started); an organization
  may also restrict connectors, and
  [Legacy Privacy Mode blocks Bot access](https://docs.x.ai/grok-bot/approvals-security-and-privacy).
- Authority to share the selected projects' necessary planning information with
  that provider, including information from private projects.

The shared template contains generic instructions and five skills:
`portfolio-v041-interview`, `portfolio-v041-charter`, `portfolio-v041-blueprint`,
`portfolio-v041-manage` and `portfolio-v041-report`. It supplies no connected account,
project data, saved working state or routine. A recipient does not need Keelen's
private repository, Python or a separately installed schema.

Grok Bot access and a Keelen account are both required. Assistant-provider and
engineering-provider costs are separate from Keelen. A manual progress check
does not require starting project execution or buying a new provider plan.

## Connect your existing projects

1. Open the published template linked above. Review the complete
   Description and all five skills before adding it to your own account.
   After Add, the provider may start skill setup automatically. Wait for setup
   to finish, then send your manual request. No scheduled routine is included;
   automatic continuation of management work is not promised.
   Adding the template again on the same account may create duplicate shared
   skills. If the website says the app did not open, check Grok Bot itself
   before installing again: the website's fallback message can remain after
   the app opens. Use **open the app** in your desktop browser and check for a
   browser prompt to open Grok Bot if needed.
2. Open **Keelen Settings → Integrations** as the workspace owner. Check that
   the intended projects already exist and GitHub can access their repositories.
   The **Keelen Portfolio Manager → Connection instructions** section gives the
   Portfolio server URL. The separate **MCP access / Rotate MCP token** section
   is for editor setup; do not rotate or paste that token for Grok Bot.
3. In Grok Bot, ask it to add a remote MCP connection to
   `https://keelen.ai/portfolio/mcp` using OAuth. Give the connection a recognizable
   name. Use the account's authorization button when it appears. Keep Grok Bot
   open while the browser completes sign-in. Never paste a token or API key
   into the conversation.
4. Check the workspace named at the top of Keelen's consent screen **before
   approving**. The desktop browser may be signed in to a different Keelen
   account. Use the correct account's browser session or sign in as the intended
   workspace owner, then restart consent if it expired. Select the exact existing
   projects: choose both if you want a two-project check. Review their
   public/private visibility, the data-sharing acknowledgment, charter, scopes
   and expiry before approving. The charter states the manager's instructions;
   narrow it to the reads and management changes you want. Load additional
   project pages if needed. A new
   approval replaces the connection's previous selection; access expires 90 days
   after approval. Inaccessible or unknown repositories are refused. Legacy
   grants and repository or visibility changes require fresh consent.
5. After the browser says **Authorization complete**, return to Grok Bot and
   wait for that account to show **Connected**. Ask it to list the selected
   projects and their current states. Check exact repository names rather than
   assuming a friendly label identifies the right project. A successful browser
   consent alone does not prove the app finished connecting.

A suitable connection message is:

> Help me connect my own Keelen account using OAuth at
> https://keelen.ai/portfolio/mcp. I want to manage my existing project,
> [owner/repository]. Show the consent step, then
> read back which projects are connected. Do not import a plan or start work.

Reconnecting can reuse an existing grant or register a new client, depending
on the app. Check the actual client ID and selected projects again; do not
assume an old charter or operation history moved to the new connection. A new
connection is not a way to reset a limit. Do not revoke old grants blindly.

## Submit a brief and keep the receipts

Start with one eligible project and two distinct tasks, for example:

> For [owner/repository], submit two separate Requests: add an optional note to
> each reading-list entry, and add title sorting with ascending and descending
> choices. Check the live charter and existing Requests before writing. Give
> each accepted Request's full ID and project mapping, then read back its
> status. Keep execution settings and protections unchanged. Report any refused
> or pending item explicitly, even if the attention digest is quiet.

The manager checks scope and the live charter before submitting. An old
goal-only charter may exclude Requests; the owner must review the intended
permissions through normal consent before dependent work continues. A draft
brief, an answered approval card or a summary is not an accepted Request.
Each distinct task should have its own actual Request ID and correct project.
An identical replay is not new work. Add another selected project when you
have work for it; two projects are not a prerequisite for using the manager.

Accepted Requests can remain pending. Submitting work may trigger intake on an
eligible active project even when the development scheduler is off. Review
the actual lifecycle, provider and execution settings before submitting; a
paused or archived demonstration is not an engineering delivery example.

## Make a supported change and return

Ask **“What needs my attention across these projects?”** Then ask for the next
decision and supporting records if the answer is only a summary:

> What is waiting to move forward, and what decision should I make next for
> each project? Read the current goals and Requests and include the matching
> record IDs. Do not make any changes yet.

Zero attention flags does not mean work is progressing. A paused or archived
project can still have pending Requests. Ask about those explicitly. A useful
answer identifies the project, its current state, what is waiting, and your next
decision. Request IDs let you check the matching records in Keelen; opening the
dashboard is optional for routine management through Grok Bot.

For a supported management change, give one exact instruction, for example:

> Set [owner/repository-one]'s product goal to “[the exact new goal].” Keep its
> current execution state and make no other changes. Read back the saved goal
> and show the decision record.

An interactive goal edit within the approved charter uses a recorded decision;
it does not require a blueprint import or a new owner approval card each time.
If the connection is read-only or the charter excludes that edit, reconnect and
approve the intended scope and charter first. If Keelen requires an owner
decision for an action, review and answer that exact card yourself, then ask the
Bot to continue. The Bot must never answer its own card. Asking “handle what's
next” does not bypass the charter, owner decisions or project protections.

Changing a goal is planning work. It is not evidence that code was built, merged,
deployed or verified. Leave execution paused for a read/goal-only walkthrough.

If a Request asks a real clarification, answer from the intended brief or make
the required product decision yourself. No question means nothing to answer.
After a genuine interval, return in the same conversation and ask:

> Read the same Requests and project records again. What changed since the last
> observation, what is waiting or unknown, and what needs my decision? Include
> full IDs and observation timestamps. Separate Keelen's recorded work from
> actions I ask you to perform now. If nothing progressed, say so.

An event can be attributed to your absence only when its timestamp falls within
that interval. Otherwise it changed between reads at an unknown time. An intake
indicator such as `spawn` is not a machine-start or delivery receipt. The Bot
runs when asked; two manual reads do not establish continuous monitoring.

Default limits include eight mutation reservations per rolling 15 minutes and
two Requests per project per UTC day for the connection principal; live limits
may differ. A recorded decision and its write each reserve one mutation, so
four Request pairs can fill the window before a goal edit. Keep the exact
failure receipt and operation keys. Wait for the stated retry time if one is
returned; otherwise timing is unknown and admission decides on a later manual
attempt. Do not reconnect, raise limits, resubmit accepted work or schedule
automatic retries to get past a stop.

## Understand the permissions and approval steps

| Step | What it authorizes | What to check |
| --- | --- | --- |
| Keelen's GitHub connection | Keelen's access to the repositories used by your existing projects and engineering workflow | In GitHub's **Keelen Connect → Configure**, use **Only select repositories** and include the intended repositories. Review the actual permission list. |
| Portfolio OAuth consent | The assistant account's selected-project reads and supported management writes under your charter and Keelen's tool policy | Correct Keelen workspace, exact public/private projects, data sharing, requested scopes and expiry. This does not pass Keelen's GitHub token or repository-source access to the Bot. |
| Recorded management decision | Evidence for a supported change, such as a goal edit within the charter | Ask for the saved value and decision record. This is not automatically a card you must approve in Settings. |
| Owner decision card | An exact action that requires your separate approval, including a blueprint import | Review the actual project, action and arguments in Keelen. Approve it yourself, then return to Grok Bot to continue. |

GitHub's Keelen Connect installation can show **read and write** permissions for
code, pull requests, administration and workflows because it serves Keelen's
engineering workflow. Do not describe that GitHub installation as read-only.
Portfolio Manager is a narrower connection: it cannot use those credentials,
read repository source, approve/merge PRs or create/import projects. You do not
need access to Keelen's private development repository.

The OAuth client name is supplied by the connecting app. In the September 9
desktop walkthrough, Grok Bot registered as **Cursor**, marked **unverified**,
with a `localhost:8787` return address. That is an observation, not a trusted
identity rule: only continue with the connection you just initiated in the app
you trust. Do not approve an unexpected request merely because its name or
return address matches this example.

If a step is confusing:

- **A repository is missing or unavailable:** check the Keelen workspace first,
  then **Settings → Integrations → GitHub**. Check the GitHub installation's
  selected repositories. If an organization owns the repository, its owner may
  need to [approve the app installation or requested access](https://docs.github.com/en/apps/using-github-apps/requesting-a-github-app-from-your-organization-owner).
  After access is
  available, reconnect Portfolio and select the projects again. Making a
  repository public is not a workaround.
- **The authorize button is waiting:** look for the newly opened page in the
  desktop browser, including another window. Keep Grok Bot open. Use the card's
  reopen action if needed; complete consent in the correct Keelen account.
  If consent expires, start again from the current connector account.
- **There are several connector accounts:** choose the account you just
  authorized and confirm its projects. Old revoked account labels are not proof
  of current access. Avoid adding another template copy to fix a connection.
- **Approve is disabled:** choose at least one eligible project, retain a
  non-empty charter and tick the data-sharing acknowledgment.
- **Settings mentions blueprint requirements or paused projects:** those are
  import/execution requirements. Blueprint import is optional for a progress
  check or supported goal edit. Do not resume work merely to clear that message.
- **A supported change is denied:** read the reason and inspect the current
  scope, charter and any owner card. Do not switch connectors to bypass it.

## Optional: import a blueprint

Use this when you want to apply a structured plan to existing projects. Drafting
can begin before connection, but that draft is unvalidated and cannot report
live state.

1. Confirm the mapping from draft labels to exact live projects. Ask for a
   blueprint preview. The server validates and normalizes the draft and opens
   an approval card. Review replacements, defaults and omitted projects in
   Keelen Settings.
2. Approve that exact card yourself in Settings, then ask the assistant to
   continue. The assistant cannot approve its own import. Approval alone
   submits no work; the subsequent import applies the approved planning work.
3. Ask for the import receipt and created Request IDs, then inspect current
   Request status. An identical retry must preserve the exact preview arguments
   and operation identity. Unknown outcomes, lost state or expired receipts
   require checking current state; never repeat the action blindly with a new
   key.

Revoke access in **Keelen Settings → Integrations → Connected apps** when
finished. Revocation blocks future access; it does not delete information
already processed by the assistant provider.

## Keep private work private

Selected project names, visions, goals, Requests, roadmap content and progress
can reach the assistant provider in the authenticated conversation. Share only
information you are authorized to provide, and keep credentials and secrets out
of chat.

Bots on the same account can share skills, files, computer sessions and
connectors. A separately named Bot is not a separate security boundary. Do not
save project data, mappings, conversations, tokens or retry state into reusable
configuration, shared skills or memories. Do not share the working conversation
or a personalized copy as the generic release. Inspect the public preview before
sharing any modified template.

Management consent does not authorize a public showcase. Public showcases
continue to reject private repositories and require a separate owner publication
decision. The manager has no repository-source, credential or pull-request
approval/merge access and must not use other connectors to expand its authority.

Keelen's terminal decision and intent payloads become eligible for compaction
after 90 days. Live decisions and unresolved actions retain required evidence;
minimal import receipts and permanent retry protection remain after payload
removal. Management diagnostics and recovery receipts use classifications,
bounded check/test/file identities, strict GitHub URLs and non-reversible
digests; raw CI excerpts are excluded from those responses and receipts, and
unknown parsing falls back to `unavailable`, not raw text. Terminal recovery
evidence follows the same 90-day compaction window as decision/intent payloads.
This does not set the assistant provider's conversation retention or
promise that deleting one Bot deletes shared resources. Review the provider's
applicable account and data controls before providing confidential material.

## Manual 0.4.1 and optional delegated management

This guide describes the published manual 0.4.1 instruction package and native
configuration. It contains five generic skills and no routines. Acceptance of
that flow does not guarantee another account's behavior or security isolation.

The separately merged optional 0.4 management features are documented in the
[portfolio tool reference](PORTFOLIO.md). Delegated actions require their own
owner opt-in and deployed tool availability. Copying the template, approving
OAuth or approving a blueprint does not enable delegated actions. Management
reads use the selected-project consent and can operate with delegation off. An
unattended 0.4 routine additionally requires a non-default
`mcp:read mcp:write mcp:routine` Portfolio authorization; that principal sees
only currently delegated projects and can use only digest/context reads plus
exact server-bound `manage_blocker`. Generic planning mutations and decision-card
tools remain unavailable to it. If the provider cannot bind that authorization
to the native routine, keep the routine disabled. This guide does not claim
native acceptance of a scheduled routine or unattended delegated recovery.

Planning work accepted is different from code merged, deployed or verified.
Engineering execution follows the project's existing settings and protections.
Set explicit finite execution budgets before enabling it: zero in Keelen's
iteration or machine-hour cap fields means unlimited. No public control-room
rollout is included in this template release.

For connection help, contact **support@keelen.ai**. The [portfolio reference](PORTFOLIO.md)
describes tool policies, approval requirements and retry behavior.
