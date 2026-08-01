# Remote Control troubleshooting

Indexed by the exact error string the CLI prints. Match the string, apply the fix.

`claude doctor` is the general-purpose diagnostic for Remote Control eligibility. It reports which individual eligibility check failed, which the summary error message often doesn't.

## `Remote Control requires a claude.ai subscription`

Not authenticated with a claude.ai account.

**Fix:** `claude auth login`, choose the claude.ai option. If `ANTHROPIC_API_KEY` is set, unset it first.

Note (pre-v2.1.206): running `/remote-control` while signed out reported `Unknown command: /remote-control` instead.

## `Remote Control requires a full-scope login token`

Authenticated with a long-lived token from `claude setup-token` or the `CLAUDE_CODE_OAUTH_TOKEN` env var. Those tokens can only make model requests; they cannot establish Remote Control sessions.

**Fix:** `claude auth login` to authenticate with a full-scope session token.

## `Unable to determine your organization for Remote Control eligibility`

Cached account information is stale or incomplete.

**Fix:** `claude auth login` to refresh it.

## `Remote Control is not yet enabled for your account`

Rollout hasn't reached the account, or cached entitlements are out of date.

**Fix:** if the plan changed recently, `claude auth logout` then `claude auth login` to refresh. Run `claude doctor` to see which check failed. This error means the rollout gate itself: environment-variable conflicts, unreachable checks, and org policy each produce their own dedicated message.

Note (pre-v2.1.154): a feature-flag-disabling env var also produced this message. On v2.1.154+, that configuration produces `Remote Control requires feature-flag evaluation` instead.

## `Couldn't verify Remote Control eligibility` / `Couldn't verify your organization's Remote Control policy`

Feature-flag or policy service unreachable. Typically offline or a proxy is blocking the request. Both messages added in v2.1.178.

**Fix:** retry once network is available, or `claude doctor` for details.

## `Remote Control requires feature-flag evaluation`

One of these variables is set: `DISABLE_TELEMETRY`, `DO_NOT_TRACK`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`, or `DISABLE_GROWTHBOOK`. The full message names which one Claude Code found.

**Fix:** unset that variable in the shell env or in the `env` block of `settings.json`. Pre-v2.1.154, this configuration produced `Remote Control is not yet enabled for your account`.

## `Remote Control is only available when using Claude via api.anthropic.com`

The session isn't talking to `api.anthropic.com` directly. Causes: Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry, or (v2.1.196+) `ANTHROPIC_BASE_URL` pointing at an LLM gateway/proxy.

**Fix (v2.1.219+):** the message names the specific variable (e.g. `CLAUDE_CODE_USE_BEDROCK`, `ANTHROPIC_BASE_URL`). Unset it, remove from the `env` block in `settings.json`, restart the session.

**Fix (pre-v2.1.219):** the message doesn't name the variable. Check the environment yourself for `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX`, and `ANTHROPIC_BASE_URL`.

## `Remote Control is disabled by your organization's policy`

Four distinct causes. Run `/status` first to see the login method and subscription.

1. **Authenticated with an API key or Console account.** Remote Control requires claude.ai OAuth. `/login`, pick the claude.ai option. Unset `ANTHROPIC_API_KEY`.
2. **Owner hasn't enabled it for the organization.** Off by default on Team and Enterprise. Owner enables at <https://claude.ai/admin-settings/claude-code>.
3. **Admin toggle is grayed out.** Data-retention or compliance configuration is incompatible with Remote Control. Not changeable from the admin panel. Contact Anthropic support.
4. **The error mentions `disableRemoteControl`.** IT administrator has disabled Remote Control on this device through managed settings (independent of the org-wide toggle).

## `Remote credentials fetch failed`

Local Claude Code could not obtain a short-lived credential from the Anthropic API.

**Fix:** re-run with `--verbose` for the full error:

```bash
claude remote-control --verbose
```

Common causes:

- Not signed in: `claude` then `/login` (claude.ai account, not API key).
- Network or proxy blocking outbound HTTPS to `api.anthropic.com:443`.
- If accompanied by `Session creation failed - see debug log`, the failure was earlier in setup. Check the subscription is active.

## `Couldn't reconnect to your Remote Control session`

Resuming a conversation (`claude --resume` or `--continue`) tried to reconnect to the Remote Control session recorded in that conversation, and reconnection failed for a possibly-temporary reason (network glitch, server error).

**Fix:** local session keeps running without Remote Control. `/remote-control` to retry, or start without `--resume` to create a new Remote Control session. If the server confirms the previous session is gone, Claude Code creates a new one silently (no message).

Pre-v2.1.200: a reconnection failure silently created a new session instead of showing this message, leaving orphan sessions in the list.

## `Your organization requires Trusted Devices for Remote Control, but this device is not enrolled`

Org has Trusted Devices enabled and this machine hasn't enrolled.

**Fix:** `/login` in Claude Code. Enrollment happens as part of sign-in; no separate enrollment command.

## `session expired for trusted-device check`

Sign-in is more than 18 hours old (Trusted Devices policy).

**Fix:** `/login` again, or confirm with Face ID / Touch ID / Windows Hello / passkey when claude.ai or the mobile app prompts. See the "Trusted Devices" section of the Remote Control docs page.

## Push notification never arrives

Not an error string, but a common issue.

Order of causes:

1. `/config` shows "No mobile registered": open the Claude mobile app once so it refreshes its push token. Warning clears on next Remote Control connect.
2. Push wasn't enabled: `/config` on the local process, toggle "Push when Claude decides" and/or "Push when actions required".
3. OS-level notification permission not granted for the Claude app.
4. Different account signed into mobile vs. the account hosting the Remote Control session.
5. iOS Focus modes or Android battery optimization suppressing the push.

Skipped-push behavior is intentional while you're typing at the local terminal. `CLAUDE_CLIENT_PRESENCE_FILE` (v2.1.181+) extends this to any presence signal you configure (e.g., screen unlocked).
