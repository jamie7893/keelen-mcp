# Keelen Portfolio Manager 0.3

Manage your public and private Keelen projects from one assistant. Set goals,
draft blueprints, submit Requests and follow progress. Connect your own Keelen
account, choose the projects it can access, and approve blueprint imports in
Keelen Settings. Keelen performs the engineering under each project's settings.
This release runs when you ask; it includes no scheduled routines.

**Published September 8, 2026.** Open
[Keelen Portfolio Manager](https://x.ai/bot/iCqviUzbFmWEKAdNqkXKm).
The Portfolio connection is available for existing workspace owners using their
own accounts and selected-project consent.

Third-party assistant template built by Keelen. The assistant platform does not
create, sponsor, endorse or operate this template. Review the full configuration
before adding it. No creator account or credentials are supplied.

Keelen permits anyone to use, copy and share this template, subject to applicable
platform terms. Each recipient must connect their own accounts. This permission
covers the template, not customer data or Keelen's private source repository.

## What you need

- An existing Keelen workspace where you are an active owner, with the existing
  projects you want to manage and current GitHub access to their repositories.
- Your own account on the supported assistant platform and access to its
  template and OAuth connection features.
- Authority to share the selected projects' necessary planning information with
  that provider, including information from private projects.

The shared template contains generic instructions and five skills:
`portfolio-interview`, `portfolio-charter`, `portfolio-blueprint`,
`portfolio-manage` and `portfolio-report`. It supplies no connected account,
project data, saved working state or routine. A recipient does not need Keelen's
private repository, Python or a separately installed schema.

## Connect and use

1. Open the published template linked above. Review the complete
   Description and all five skills before adding it to your own account.
   After Add, the provider may start skill setup automatically. Wait for setup
   to finish, then send your manual request. No scheduled routine is included;
   automatic continuation of management work is not promised.
   Adding the template again on the same account may create duplicate shared skills.
2. Explain what you want to manage in your authenticated conversation. Use the
   minimum necessary detail. Drafting can begin before connection, but a draft
   is unvalidated and the assistant cannot report live project state yet.
3. Connect your own Keelen account through OAuth at the portfolio resource,
   `https://keelen.ai/portfolio/mcp`. Follow the connection instructions shown in
   Keelen Settings. Never paste a token or API key into the conversation.
4. In Keelen's consent screen, select the exact existing projects. Review their
   public/private visibility, the data-sharing acknowledgment, charter, scopes
   and expiry before approving. Load additional project pages if needed. A new
   approval replaces the connection's previous selection; access expires 90 days
   after approval. Inaccessible or unknown repositories are refused. Legacy
   grants and repository or visibility changes require fresh consent.
5. Confirm the mapping from draft labels to exact live projects. Ask for a
   blueprint preview. The server validates and normalizes the draft and opens
   an approval card. Review replacements, defaults and omitted projects in
   Keelen Settings.
6. Approve that exact card yourself in Settings, then ask the assistant to
   continue. The assistant cannot approve its own import. Approval alone
   submits no work; the subsequent import applies the approved planning work.
7. Ask for the import receipt and created Request IDs, then inspect current
   Request status. An identical retry must preserve the exact preview arguments
   and operation identity. Unknown outcomes, lost state or expired receipts
   require checking current state; never repeat the action blindly with a new
   key.
8. Use manual follow-ups to review progress or make supported management
   changes. Revoke access in Keelen Settings → Connected apps when finished.
   Revocation blocks future access; it does not delete information already
   processed by the assistant provider.

A suitable first message is:

> Help me organize my existing Keelen projects. Start by asking what I want to
> achieve, then help me connect and review a blueprint before anything is imported.

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
removal. Management diagnostics and recovery receipts use structured status,
identifiers and digests; raw CI excerpts are excluded from those responses and
receipts. This does not set the assistant provider's conversation retention or
promise that deleting one Bot deletes shared resources. Review the provider's
applicable account and data controls before providing confidential material.

## Manual 0.3 and optional 0.4 management

This guide describes the accepted manual 0.3 instruction package and native
configuration. It contains five generic skills and no routines. Acceptance of
that flow does not guarantee another account's behavior or security isolation.

The separately merged optional 0.4 management features are documented in the
[portfolio tool reference](PORTFOLIO.md). Delegated actions require their own
owner opt-in and deployed tool availability. Copying the 0.3 template, approving
OAuth or approving a blueprint does not enable delegated actions. Management
reads use the selected-project consent and can operate with delegation off.
This guide does not claim native acceptance of a 0.4 template or scheduled routine.

Planning work accepted is different from code merged, deployed or verified.
Engineering execution follows the project's existing settings and protections.
Set explicit finite execution budgets before enabling it: zero in Keelen's
iteration or machine-hour cap fields means unlimited. No public control-room
rollout is included in this template release.

For connection help, contact **support@keelen.ai**. The [portfolio reference](PORTFOLIO.md)
describes tool policies, approval requirements and retry behavior.
