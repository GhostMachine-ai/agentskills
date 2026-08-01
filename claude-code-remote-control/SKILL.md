---
name: claude-code-remote-control
description: Use when working with Claude Code Remote Control, starting a remote session, driving a local Claude Code process from a browser or phone, running `claude remote-control` server mode, `claude --remote-control` (`--rc`) interactive mode, `/remote-control` (`/rc`) slash command, connecting from another device, enabling mobile push notifications, or troubleshooting Remote Control errors (subscription required, full-scope token required, not yet enabled for account, disabled by organization policy, remote credentials fetch failed, feature-flag evaluation, ANTHROPIC_BASE_URL, Trusted Devices).
compatibility: Requires Claude Code with Remote Control support. Some sub-features (session name reminders, `--continue`/`--session-id` in server mode, `/mcp` from mobile, others) have their own minimum versions documented on the Remote Control page.
license: Apache-2.0
---

# Claude Code Remote Control

Remote Control connects an already-running local `claude` process to claude.ai/code or the Claude mobile app. The local process still does the work (still touches files, runs tools, calls out from the machine); the remote client just drives it. Nothing about the workspace moves to the cloud.

The canonical docs live at <https://code.claude.com/docs/en/remote-control>. This skill is a router: quick decision tree, non-negotiables, and error catalog. When a specific version cutoff matters (a sub-feature or a fixed bug), consult the docs page directly, as version numbers move.

## Non-negotiables

1. **claude.ai OAuth only.** `claude auth login` (or `/login`), pick the claude.ai option. `ANTHROPIC_API_KEY`, `CLAUDE_CODE_OAUTH_TOKEN`, or workbench-scoped tokens fail with `Remote Control requires a full-scope login token`.
2. **The local process must stay alive.** Remote Control is a *bridge* to a running process, not a hosted runtime. Quitting `claude`, closing the terminal, or letting the container die ends the remote session. Sessions cannot be attached to retroactively. To keep a session alive on an SSH-reachable machine, run inside `tmux` or `screen`.
3. **~10 minute network outage kills the session.** If the machine is awake but can't reach the network for roughly that long, the session times out and the process exits. Start a new one.
4. **One remote session per interactive process.** Outside server mode, each `claude` instance supports one remote session at a time. To run many concurrent sessions from one process, use server mode.
5. **Workspace must be trusted.** Start `claude` in the project directory once to accept the trust dialog. Home directory trust never persists, so start Remote Control from a project directory.
6. **API endpoint must be `api.anthropic.com`.** Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry, and any custom `ANTHROPIC_BASE_URL` are excluded. Unset the routing variable or use a session that talks to Anthropic directly.
7. **Feature-flag evaluation must be on.** `DISABLE_TELEMETRY`, `DO_NOT_TRACK`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`, and `DISABLE_GROWTHBOOK` each disable the evaluation Remote Control availability depends on. Unset the one that's set (shell env or `env` block in `settings.json`).
8. **On Team/Enterprise, an Owner must enable Remote Control.** It's off by default. Owner toggles it at <https://claude.ai/admin-settings/claude-code>. Zero-Data-Retention orgs cannot enable it.

## Three ways to start a session (plus VS Code)

Pick by intent. All end at the same claude.ai/code remote client.

| Mode | Command | When to use |
|---|---|---|
| Interactive flag | `claude --remote-control` (alias: `--rc`) | Working locally, want the option to switch devices mid-session. Optional name argument. |
| Slash command | `/remote-control` inside `claude` (alias: `/rc`) | Didn't start with the flag, want to enable now for the current session, carrying over conversation history. |
| Server mode | `claude remote-control` (subcommand) | Walking away and later starting work *entirely* from the phone or browser. Hosts many sessions in one process (default capacity 32), spawned on demand. |
| VS Code | `/remote-control` in the extension prompt box | Working in the Claude Code VS Code extension. |

**Server mode is the only mode where the user is elsewhere the whole time.** The other three require you to be at the machine when work starts.

Useful server-mode flags: `--name "Project X"` (custom title), `--continue` / `--session-id <id>` (resume a prior server-mode session, both require v2.1.200+), `--spawn worktree` (each on-demand session gets its own git worktree), `--capacity <N>` (default 32), `--sandbox` (opt into filesystem/network isolation). `claude remote-control --help` shows the full list, but only if the account is eligible.

## Auto-connect for every interactive session

`/config` inside `claude` has an "Enable Remote Control for all sessions" toggle. Set to `true` and every future interactive session registers a remote session automatically. Set to `false` to never auto-connect. Set to `default` to defer to the org admin default or Claude Code's default. Multiple concurrent processes each get their own remote session; for many sessions in one process, use server mode instead.

## Connecting from another device

Three paths, same session:

- **Open the printed URL** in any browser.
- **Scan the QR code.** In `claude remote-control`, press `spacebar` to toggle the QR code display.
- **Open <https://claude.ai/code>** or the Claude mobile app (**Code** tab) and pick the session from the list. Online Remote Control sessions show a computer icon with a green dot.

Session title resolution order (documented, useful for finding a session in the list):

1. Name passed to `--name`, `--remote-control`, or `/remote-control`
2. `/rename` value
3. Last meaningful message in existing history
4. Auto-generated (`<hostname>-graceful-unicorn`), where `<hostname>` is the machine hostname or the prefix set with `--remote-control-session-name-prefix` (env var equivalent: `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX`; that variable is an **input** for customizing the prefix, not a marker child processes can read to detect Remote Control mode)

Don't have the app? Run `/mobile` inside `claude` to display a download QR code.

## Push notifications

Claude decides when to push (typically at the end of a long-running task or when input is needed). You can also request one in a prompt: "notify me when the tests finish."

`/config` has two toggles:

- **Push when Claude decides** (proactive notifications from Claude)
- **Push when actions required** (permission prompts and questions)

Enable one, the other, or both. Beyond these two toggles, there is no per-event configuration.

Skipped-push behavior: Claude Code skips pushes while you're typing in or focused on the connected terminal. To extend this to any time you're at the machine (even in another window), set `CLAUDE_CLIENT_PRESENCE_FILE` to a marker file path; pushes are skipped while the file exists. Requires v2.1.181+.

If pushes don't arrive: `/config` will show "No mobile registered" if the app never refreshed its push token. Open the app once and it clears. iOS Focus modes and Android battery optimization commonly delay or suppress pushes.

## Local-only vs. mobile/web commands

Some slash commands only work in the local terminal. Attempting them from mobile/web silently no-ops or errors. Local-only: `/plugin`, `/resume`, and anything that needs a picker or interactive UI.

Mobile/web supported:

- Text-output commands: `/compact`, `/clear`, `/context`, `/usage`, `/exit`, `/usage-credits` (prints billing URL), `/recap`, `/reload-plugins`
- Argument-required: `/model <name>`, `/effort <level>`, `/fast`, `/color`, `/rename`
- Special cases: `/mcp` (v2.1.166+) returns server status summary from mobile, opens connector directory on web; `reconnect`/`enable`/`disable` subcommands work from both. `/config` (v2.1.181+) accepts `key=value` from mobile, opens web settings in the browser.

## Anti-patterns

| Anti-pattern | What actually happens | Fix |
|---|---|---|
| Setting `ANTHROPIC_API_KEY` and expecting Remote Control | `Remote Control requires a full-scope login token` | `claude auth login` with the claude.ai account. Unset the key. |
| Closing the terminal after `/remote-control` | Session ends the moment the process exits | `claude remote-control` server mode, or run inside `tmux` / `screen` |
| Expecting a remote client to attach to a session that wasn't started with Remote Control | No attach path exists; sessions are opt-in at start | Restart with `--rc` or `/remote-control`, or enable auto-connect in `/config` |
| Running Remote Control alongside Ultraplan | Ultraplan takes over the claude.ai/code interface, disconnecting Remote Control | Pick one per session |
| Guessing an env var to detect "am I in Remote Control?" | No documented signal exists. `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` is an **input**, not a marker | Don't detect; design tools that don't depend on knowing |
| Assuming inbound ports must be opened | None are; the client polls outward | Don't touch the firewall |
| On Team/Enterprise, trying Remote Control before the Owner enables it | `Remote Control is disabled by your organization's policy` | Owner enables the toggle at `claude.ai/admin-settings/claude-code` |
| Trusting a workspace just to make Remote Control work | Trust also grants tool auto-approval defaults for that workspace | Only trust workspaces you own or have vetted |

## Reference files

| File | Read when |
|---|---|
| `references/troubleshooting.md` | An exact error string appears, or `claude doctor` reports something specific |
