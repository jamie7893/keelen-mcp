# keelen-mcp

**Steer Keelen, the AI coding agent for GitHub, from your chat.**

Build apps and internal tools, or keep a defined backlog moving with the
approval and merge controls you choose. Keelen turns requests into a standing
roadmap, plans tasks, runs checks and opens pull requests in your repository.

Paste [SETUP.md](SETUP.md) into a compatible MCP client and say **"set me up."**
Your agent guides account setup and project creation. You confirm your email
and complete AI credential connection, GitHub approval and any billing steps
in the browser. Once connected, use chat to submit requests, set priorities
and answer questions about the work.

Use your own supported **AI subscription or API key** with Claude Code,
Codex, GLM, Kimi or Grok. Keelen adds no markup to AI usage.
[See how the MCP workflow works](https://keelen.ai/mcp-server/?utm_source=mcp-server&utm_medium=github-readme&utm_campaign=readme).

This repo is documentation only. It describes the hosted [Model Context
Protocol](https://modelcontextprotocol.io) (MCP) server at
`https://keelen.ai/mcp`. Keelen runs the coding workflow as a hosted service;
this repository contains its documentation and client configuration examples.

![Recorded Keelen MCP setup walkthrough](keelen-setmeup.gif)

The recording shows an earlier setup flow. Follow the current instructions
below for account verification and browser approvals.

## Work you can hand over

- Build an app or progress the fixes, tests and small features on a solo
  project's roadmap.
- Build an internal request portal, reporting dashboard or integration while
  you focus on your main work. Deployment, colleague sign in and access rules
  belong to the software you build and the hosting you choose.
- Queue defined work for a team or client repository, including outside
  staffed hours, then review the resulting pull requests and decisions.
- Run Security, Legal and Controls reviews, triage findings and send selected
  code work into the roadmap. These are bounded reviews, not penetration
  tests, legal advice or certification. Repeated reviews can be triggered
  through MCP by your own agent or external scheduler; Keelen has no built in
  scan scheduler.

These are example uses, not customer deployments or ready made templates.
You set priorities and choose plan approval, manual merge or other supported
controls for each project. Keelen checks changes before they can merge and
holds failed checks. Capacity, provider limits, missing information and
decisions can pause progress; a queue is not a promise of finished work by a
particular time.

## AI engines and credentials

Connect credentials in the Keelen dashboard, never in the MCP chat. The
workspace API key used by your MCP client is separate from these credentials.

| AI engine | Supported connection |
| --- | --- |
| Claude Code | Claude Pro or Max sign in, or an Anthropic API key |
| Codex | ChatGPT sign in, or an OpenAI API key |
| GLM | Z.ai API key |
| Kimi | Kimi Code membership key or Moonshot Open Platform API key |
| Grok | SuperGrok sign in or xAI API key |

Engine availability can differ by role. Grok here means the coding engine;
it does not establish Grok Bot compatibility as an MCP client. OpenRouter
is currently limited to Keelen's own test and benchmark projects and is not
enabled for customer repositories.

## Connect and set up

For Claude Code, connect the server without a token, then hand your agent
the setup instructions:

```
claude mcp add --transport http keelen https://keelen.ai/mcp
```

Then paste the contents of [`SETUP.md`](SETUP.md) into your agent and say
**"set me up"**. It will ask for your email and the verification code that
lands in your inbox. It then guides you through connecting an AI engine,
approving GitHub access, creating or importing a project, and provisioning.
Follow the dashboard links for the steps that require your browser.

The whole flow on one page, with the 50-second demo:
<https://keelen.ai/mcp-server/?utm_source=mcp-server&utm_medium=github-readme&utm_campaign=setup>

Using a different client? See the per-client guides:

- [Claude Code](clients/claude-code.md)
- [Claude Desktop](clients/claude-desktop.md)
- [Codex](clients/codex.md)
- [Cursor](clients/cursor.md)
- [Windsurf](clients/windsurf.md)
- [Generic / other MCP clients](clients/generic.md)

Building a Roblox game? Keelen can generate art, audio, meshes, skyboxes, and
animated characters from plain-language requests — see
[`roblox-assets.md`](roblox-assets.md).

## Tools (abbreviated)

The server exposes 41 tools. Two work with no key at all (`signup`,
`verify_email`); the rest need the bearer key `verify_email` gives you. Full
reference with parameters and behavior: [`TOOLS.md`](TOOLS.md).

| Group | Tools |
| --- | --- |
| Getting started | `signup`, `verify_email` |
| Onboarding | `get_onboarding_status`, `open_dashboard`, `connect_github`, `list_github_repos`, `import_project`, `get_provisioning_status`, `get_billing` |
| Projects & roadmap | `list_projects`, `create_project`, `archive_project`, `delete_project`, `submit_request`, `get_request_status`, `answer_request`, `refine_request`, `set_product_vision`, `set_product_goal`, `get_product_vision`, `get_product_goal`, `list_roadmap`, `reorder_roadmap`, `cancel_roadmap_item`, `clear_horizon_pin`, `project_status`, `list_escalations`, `resolve_escalation`, `retry_blocked_task`, `rearm_roadmap_item`, `resolve_platform_policy_conflict`, `replan_task`, `close_task`, `control_scheduler`, `set_ui_review_scenario_cap`, `rollback_roblox_place` |
| Reviews | `run_security_review`, `run_legal_exposure_review`, `get_legal_exposure_findings`, `run_control_gap_review`, `get_control_gap_findings` |

The server speaks MCP protocol revision `2026-07-28` as well as the earlier
handshake revisions, from the same endpoint — no client action is required
either way. Details in [`clients/generic.md`](clients/generic.md#protocol-support);
notable contract changes are recorded in [`CHANGELOG.md`](CHANGELOG.md).

[`PORTFOLIO.md`](PORTFOLIO.md) documents the separate portfolio surface,
which is not generally available. Its scoped OAuth interface is distinct
from the primary MCP server described here.

## FAQ

**Is my code touched before I agree to anything?** No. Connecting the MCP
server tokenless only lets you sign up and verify your email — it can't see,
read, or modify any repository. `connect_github` requires an explicit
GitHub App install you approve in your browser, and every following build
runs inside an isolated per-tenant sandbox scoped to your own workspace.

**What does it cost?** Signing up, connecting GitHub, and creating a project
are all free. Actually running the autonomous loop (compute) requires an
active subscription — `get_billing()` returns a Stripe checkout link the
moment you're ready. A free/unpaid workspace can do everything up through
project creation. Work can start once billing is active,
the project's loop is enabled and its setup requirements are satisfied.

**How do I revoke access?** Every key minted through this flow (or the
dashboard) can be individually revoked. Go to your Keelen dashboard →
**Settings → API keys** and delete the key. Revoking takes effect
immediately; your agent will need a fresh key (rerun the signup flow, or
mint a new one from the dashboard) to keep calling the server.

See [`SECURITY.md`](SECURITY.md) for the full security model: what the MCP
server can and can't reach, rate limits, and key handling.

## Support

Questions or issues: **support@keelen.ai**
