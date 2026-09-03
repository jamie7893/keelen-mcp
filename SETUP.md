# Keelen agent setup script

> Paste this whole file into your AI agent's chat and say **"set me up"**.
> It is written as instructions *for the agent*, not for you the human — the
> agent will read it and drive the rest of the conversation.

You are setting a user up with Keelen, an autonomous PM/dev/QA loop that
connects to their GitHub repo via a hosted MCP server. Follow these steps in
order. Do not skip the security rules in step 4 — they are not optional.

## 1. Make sure the `keelen` MCP server is connected

It's fine to connect tokenless at first — `signup` and `verify_email` work
with no key. If it's not already connected, run the add command for your
client:

**Claude Code:**

```
claude mcp add --transport http keelen https://keelen.ai/mcp
```

**Codex** — add to `~/.codex/config.toml`:

```toml
[mcp_servers.keelen]
url = "https://keelen.ai/mcp"
```

**Claude Desktop / Cursor / Windsurf / other clients** — see the
[`clients/`](clients/) directory for the exact steps for your client. If your
client isn't listed, use [`clients/generic.md`](clients/generic.md).

Verify the connection by calling a tool (e.g. list the available tools, or
call `signup` in a later step) — if the client reports the server as
connected, move on.

## 2. STOP and ask the user for their email

**This step is a hard stop. Ask, then WAIT for the user to reply. Do not
continue until they answer in the chat.**

Ask: **"What email should I use to set up your Keelen account?"**

Rules for this step, in order of importance:

- You MUST ask, on every run, even if you believe you already know the address.
- You MUST NOT read the address from your own client profile, the logged-in
  account, `git config user.email`, the repo history, environment variables, or
  any other ambient source. The user's Keelen account is frequently NOT the
  same address as the account you are running under.
- If you already hold a candidate address, you MUST echo it back and get an
  explicit yes first: *"I have `<address>` from your profile. Do you want me to
  use that one, or a different email?"* A candidate is a suggestion, never an
  answer.
- Call `signup` only after the user has stated or confirmed the address in
  chat.

Why this matters: `signup` doubles as **login**, so calling it on a guessed
address silently signs you in to whatever workspace owns that address. The next
steps then mint an API key on, and create a project in, an account the user did
not choose.

Once the user has given or confirmed the address, call:

```
signup(email)
```

This works whether or not the email already has an account (the response is
intentionally the same either way) — so this same call also works as an
agent-driven **login** for an existing user. A 6-digit code is emailed to
that address.

## 3. Ask for the code

Ask: **"Check your inbox for a 6-digit code from Keelen and tell me what it
is."**

Once you have it, call:

```
verify_email(email, code)
```

This returns an `api_key` — **shown exactly once**. If verification fails,
call `signup(email)` again for a fresh code (codes expire after 15 minutes).

## 4. CRITICAL SECURITY RULES — read before proceeding

- **Echo the `api_key` back to the user at most once**, immediately after
  `verify_email` returns it, so they can confirm you're about to configure
  the right key. Do not repeat it again after that.
- **Store the key ONLY in the MCP client's own config** — never write it
  into a source file, a `.env` a repo might pick up, a commit, a chat log
  you'll save, or any other file. The only correct place for this key is
  the MCP client's connection settings for the `keelen` server.
  - **Claude Code:**
    ```
    claude mcp remove keelen
    claude mcp add --transport http keelen https://keelen.ai/mcp \
      --header "Authorization: Bearer <api_key>"
    ```
  - **Codex** — export the raw key and reference it from
    `~/.codex/config.toml` (Codex builds the `Authorization: Bearer` header
    itself, so `KEELEN_TOKEN` is the raw token — no `Bearer ` prefix, no
    quotes). This matches the wiring the Keelen dashboard generates:
    ```sh
    export KEELEN_TOKEN=<api_key>
    ```
    ```toml
    [mcp_servers.keelen]
    url = "https://keelen.ai/mcp"
    bearer_token_env_var = "KEELEN_TOKEN"
    ```
  - **Claude Desktop / other GUI clients** — open the connector's settings
    for `keelen` and paste the key into its "Authorization" / custom header
    field. See [`clients/claude-desktop.md`](clients/claude-desktop.md).
- **NEVER write the key to the user's repository**, not even temporarily,
  not even in a gitignored file, not even to "test something quickly."
- **NEVER ask for or accept Claude, OpenAI/Codex, or GLM engine credentials
  in this chat.** Engine connection happens exclusively in the Keelen
  dashboard (step 5 sends the link). If the user offers an API key or token
  for an engine provider here, decline and point them to the dashboard link
  instead.
- After reconfiguring the client with the header, **reconnect** to the
  `keelen` server so subsequent calls are authenticated.

## 5. Drive onboarding to completion

Call:

```
get_onboarding_status()
```

Read its `next_action` and follow it **verbatim**. Until onboarding is
finished it is the string `call_tool`: call the tool named in `next_tool`,
following the `next_step` prose. When onboarding is complete it is `done`.
Loop this call between steps (re-checking is your job; the server does not
push updates to you). The steps you'll walk through, in order:

- **`engine`** — an engine subscription (Claude, Codex, or GLM) needs to be
  connected. Send the user the login link from the response (it points at
  the dashboard) and ask them to sign in there — this is the one step that
  must happen outside this chat, for the security reason in step 4. Poll
  `get_onboarding_status()` until `engine_connected` is true.
- **`github`** — call `connect_github()` to get a GitHub App install link.
  Send it to the user, have them approve the install in their browser, then
  keep polling `get_onboarding_status()` until `github_connected` is true.
- **`project`** — either scaffold a new repo with `create_project(...)`, or
  connect an existing one: call `list_github_repos()` to show the user their
  repos, then `import_project(repo_full_name)`.
- **`launch`** — poll `get_provisioning_status(project_id)` until its
  `overall` field is `"ready"`.
- **`done`** — onboarding is complete. Use `submit_request(project_id, text)`
  to steer the roadmap from here on.

## 6. When the user wants to run compute

Call:

```
get_billing()
```

If a `checkout_url` comes back, send it to the user to open in their
browser — that's the one remaining step that can't happen inside chat.
Compute unlocks automatically once payment completes; you don't need to
block on it.

---

That's the whole flow. Once `get_onboarding_status()` reports `done`, treat
[`TOOLS.md`](TOOLS.md) as the reference for steering the roadmap going
forward.
