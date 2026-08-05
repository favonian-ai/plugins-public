---
name: premium_check_cases
description: Use this skill 1) when you are instructed to invoke it directly, 2) whenever any tool result includes PLUGPASS_PLUGIN=brainstorm && USE_AUTHORIZED=false, PLUGPASS_PLUGIN=brainstorm && SIGNUP_REQUIRED=true, or PLUGPASS_PLUGIN=brainstorm && OUTDATED=true.
user-invocable: false
allowed-tools: mcp__plugin_brainstorm_brainstorm-server__authenticate, AskUserQuestion, Bash(open:*), Bash(xdg-open:*), Bash(echo:*), PowerShell(Start-Process:*), PowerShell(Write-Output:*), Read, Skill(update-config)
---

USER_INPUT_TOOL = `AskUserQuestion` tool or `ask_user_input_v0` tool

A `>` block is a message for the user. Replace each {VARIABLE} in the > blocks and USER_INPUT_TOOL prompt copy with its value, sourced from the triggering skill/agent body, tool response, or both; do not state a variable verbatim in its token form. Output the result as your own message — do not narrate, add a preamble or sign-off, wrap it in a quote block, or print the `>` characters themselves.

Present each USER_INPUT_TOOL prompt exactly as specified. The names of the USER_INPUT_TOOL prompts (e.g. `ChatCoworkConnectConfirm`) are used for guiding your logic flow only, and should not be communicated to the user.

Anything other than content within `>` blocks or USER_INPUT_TOOL prompt should be interpretted as instructions for you (or the main agent) to follow.

Any variables defined by tool presence should be assessed purely from its presence in your tool list; never attempt to call a tool if not present.

- NOT_CONNECTED = `mcp__plugin_brainstorm_brainstorm-server__check_premium_access` present in your tool list ? `false` : `true`
- PRODUCT = If your system instructions indicate your environment is Cowork, then `cowork`; if they indicate your environment is Claude Code, then `code`; otherwise `chat`.
- If PRODUCT=`code`
  - OS = If your system instructions indicate the platform is `darwin`, then `mac`; if `linux`, then `linux`; if `win32`, then `windows`.
  - OPEN_URL_TOOL = If OS=`mac`, then Bash `open "<url>"`; if OS=`linux`, then Bash `xdg-open "<url>"`; if OS=`windows`, then PowerShell `Start-Process "<url>"`.
  - CODE_CLIENT = If (OS=`mac` || OS=`linux`), then Bash `echo "CLAUDE_CODE_ENTRYPOINT=$CLAUDE_CODE_ENTRYPOINT"`; if OS=`windows`, then PowerShell `Write-Output "CLAUDE_CODE_ENTRYPOINT=$env:CLAUDE_CODE_ENTRYPOINT"` (expected value: `cli` || `claude-desktop` || `remote`)
- If FEATURE_TYPE=`agent`:
  - SUBAGENT = USER_INPUT_TOOL present in your tool list ? `false` : `true`
  - For each top-level (# heading) conditional section below:
    - If SUBAGENT=`false`, run the applicable section's instructions yourself
    - If SUBAGENT=`true`, pass the entire applicable section's instructions to the main agent, exactly as written, for processing. If USE_AUTHORIZED=`true`, always follow CORE_INSTRUCTIONS to process the user's request first, then subsequently follow the instructions in the applicable section below, or if SUBAGENT=`true` append the entire section's contents to your final response to the main agent.

---
# If NOT_CONNECTED=true

## If (PRODUCT=`chat` || PRODUCT=`cowork`)

> {FEATURE_PITCH}
>
> Sign in to Brainstorm to use this feature.
>
> [Sign up {SIGNUP_MODIFIER}](https://app.bandwidth.email/signup?feature={FEATURE_ID}&connect=web&product=claude)  [Log in](https://app.bandwidth.email/login?feature={FEATURE_ID}&connect=web&product=claude)

Present the `ChatCoworkConnectConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you signed in?
- Options:
  - Yes
  - Not now

### If user answers `Yes` to `ChatCoworkConnectConfirm`

> Press `cmd-R` (`ctrl-R` on Windows) to refresh the session to use {FEATURE_NAME_COPY}.

Present the `ChatCoworkRefreshConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you refreshed?
- Options:
  - Yes
  - Cancel setup

### If user answers `Yes` to `ChatCoworkRefreshConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `ChatCoworkConnectConfirm` || `Cancel setup` to `ChatCoworkRefreshConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Yes` or `Not now` to `ChatCoworkConnectConfirm` || `Yes` or `Cancel setup` to `ChatCoworkRefreshConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If CODE_CLIENT=`cli`

> {FEATURE_PITCH}

Present the `CliConnectChoice` prompt with USER_INPUT_TOOL:
- Prompt: Sign up {SIGNUP_MODIFIER} or log in to Brainstorm to use this feature.
- Options:
  - Sign up
  - Log in
  - Not now

### If user answers `Sign up` to `CliConnectChoice`

AUTH_URL = the URL returned by `mcp__plugin_brainstorm_brainstorm-server__authenticate`, with `&mode=signup&feature={FEATURE_ID}` appended.
Open {AUTH_URL} with the OPEN_URL_TOOL.

Present the `CliSignInConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you signed up?
- Options:
  - Yes
  - Never mind

### If user answers `Log in` to `CliConnectChoice`

AUTH_URL = the URL returned by `mcp__plugin_brainstorm_brainstorm-server__authenticate`, with `&mode=login&feature={FEATURE_ID}` appended.
Open {AUTH_URL} with the OPEN_URL_TOOL.

Present the `CliSignInConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you logged in?
- Options:
  - Yes
  - Never mind

### If user answers `Yes` to `CliSignInConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `CliConnectChoice` || `Never mind` to `CliSignInConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Sign up`, `Log in`, or `Not now` to `CliConnectChoice` || `Yes` or `Never mind` to `CliSignInConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If CODE_CLIENT=`claude-desktop`

> {FEATURE_PITCH}

Present the `CodeDesktopConnectChoice` prompt with USER_INPUT_TOOL:
- Prompt: Sign up {SIGNUP_MODIFIER} or log in to Brainstorm to use this feature.
- Options:
  - Sign up
  - Log in
  - Not now

### If user answers `Sign up` to `CodeDesktopConnectChoice`

Open https://app.bandwidth.email/signup?feature={FEATURE_ID}&connect=web&product=claude with the OPEN_URL_TOOL.

Present the `CodeDesktopConnectConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you signed up & connected?
- Options:
  - Yes
  - Never mind

### If user answers `Log in` to `CodeDesktopConnectChoice`

Open https://app.bandwidth.email/login?feature={FEATURE_ID}&connect=web&product=claude with the OPEN_URL_TOOL.

Present the `CodeDesktopConnectConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you logged in & connected?
- Options:
  - Yes
  - Never mind

### If user answers `Yes` to `CodeDesktopConnectConfirm`

> Press `cmd-R` (`ctrl-R` on Windows) to refresh the session to use {FEATURE_NAME_COPY}.

Present the `CodeDesktopRefreshConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you refreshed?
- Options:
  - Yes
  - Cancel setup

### If user answers `Yes` to `CodeDesktopRefreshConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `CodeDesktopConnectChoice` || `Never mind` to `CodeDesktopConnectConfirm` || `Cancel setup` to `CodeDesktopRefreshConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Sign up`, `Log in`, or `Not now` to `CodeDesktopConnectChoice` || `Yes` or `Never mind` to `CodeDesktopConnectConfirm` || `Yes` or `Cancel setup` to `CodeDesktopRefreshConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If CODE_CLIENT=`remote`

> {FEATURE_PITCH}
>
> Sign in to Brainstorm to use this feature.
>
> [Sign up {SIGNUP_MODIFIER}](https://app.bandwidth.email/signup?feature={FEATURE_ID}&connect=web&product=claude)  [Log in](https://app.bandwidth.email/login?feature={FEATURE_ID}&connect=web&product=claude)
>
> Once you've signed in, start a new session to continue.

Do not perform CORE_INSTRUCTIONS!

---
# If SIGNUP_REQUIRED=`true`

## If SIGNUP_MODIFIER=`free` && (PRODUCT=`chat` || PRODUCT=`cowork` || CODE_CLIENT=`remote`)

> {FEATURE_PITCH}
>
> Create a free Brainstorm account to use this feature.
>
> [Sign up free]({PLUGIN_ORIGIN}/signup?feature={FEATURE_ID})

Present the `FreeSignupLinkConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you signed up?
- Options:
  - Yes
  - Not now

### If user answers `Yes` to `FreeSignupLinkConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers `Not now` to `FreeSignupLinkConfirm`

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Yes` or `Not now` to `FreeSignupLinkConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If SIGNUP_MODIFIER=`free` && (CODE_CLIENT=`cli` || CODE_CLIENT=`claude-desktop`)

> {FEATURE_PITCH}
>
> Create a free Brainstorm account to use this feature.

Present the `SignupChoice` prompt with USER_INPUT_TOOL:
- Prompt: Sign up free to continue
- Options:
  - Sign up
  - Not now

### If user answers `Sign up` to `SignupChoice`

Open {PLUGIN_ORIGIN}/signup?feature={FEATURE_ID} with the OPEN_URL_TOOL.

Present the `FreeSignupOpenConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you signed up?
- Options:
  - Yes
  - Never mind

### If user answers `Yes` to `FreeSignupOpenConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `SignupChoice` || `Never mind` to `FreeSignupOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Sign up` or `Not now` to `SignupChoice` || `Yes` or `Never mind` to `FreeSignupOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If SIGNUP_MODIFIER≠`free` && (PRODUCT=`chat` || PRODUCT=`cowork` || CODE_CLIENT=`remote`)

> {FEATURE_PITCH}
>
> Create your Brainstorm account & upgrade to the {PLAN_NAME} plan to use this premium feature. [See pricing]({PLUGIN_ORIGIN}/plans)
>
> [Sign up]({PLUGIN_ORIGIN}/signup?feature={FEATURE_ID})

Present the `PaidLinkSignupConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you signed up?
- Options:
  - Yes
  - Not now

### If user answers `Yes` to `PaidLinkSignupConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers `Not now` to `PaidLinkSignupConfirm`

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Yes` or `Not now` to `PaidLinkSignupConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If SIGNUP_MODIFIER≠`free` && (CODE_CLIENT=`cli` || CODE_CLIENT=`claude-desktop`)

> {FEATURE_PITCH}
>
> Create your Brainstorm account & upgrade to the {PLAN_NAME} plan to use this premium feature. [See pricing]({PLUGIN_ORIGIN}/plans)

Present the `PaidOpenSignupChoice` prompt with USER_INPUT_TOOL:
- Prompt: Sign up to continue
- Options:
  - Sign up
  - Not now

### If user answers `Sign up` to `PaidOpenSignupChoice`

Open {PLUGIN_ORIGIN}/signup?feature={FEATURE_ID} with the OPEN_URL_TOOL.

Present the `PaidOpenSigninConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you signed up?
- Options:
  - Yes
  - Never mind

### If user answers `Yes` to `PaidOpenSigninConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `PaidOpenSignupChoice` || `Never mind` to `PaidOpenSigninConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Sign up` or `Not now` to `PaidOpenSignupChoice` || `Yes` or `Never mind` to `PaidOpenSigninConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

---
# If LIMIT_REACHED=`upgrade_to_access`

## If (PRODUCT=`chat` || PRODUCT=`cowork` || CODE_CLIENT=`remote`)

> {FEATURE_PITCH}
>
> Upgrade to the Brainstorm {PLAN_NAME} plan to use this premium feature. [See pricing]({PLUGIN_ORIGIN}/plans)
>
> [Upgrade]({CHECKOUT_URL})

Present the `UpgradeLinkConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you upgraded?
- Options:
  - Yes
  - Not now

### If user answers `Yes` to `UpgradeLinkConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers `Not now` to `UpgradeLinkConfirm`

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Yes` or `Not now` to `UpgradeLinkConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If (CODE_CLIENT=`cli` || CODE_CLIENT=`claude-desktop`)

> {FEATURE_PITCH}
>
> Upgrade to the Brainstorm {PLAN_NAME} plan to use this premium feature. [See pricing]({PLUGIN_ORIGIN}/plans)

Present the `UpgradeOpenChoice` prompt with USER_INPUT_TOOL:
- Prompt: Upgrade your account?
- Options:
  - Upgrade
  - Not now

### If user answers `Upgrade` to `UpgradeOpenChoice`

Open {CHECKOUT_URL} with the OPEN_URL_TOOL.

Present the `UpgradeOpenConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you upgraded?
- Options:
  - Yes
  - Never mind

### If user answers `Yes` to `UpgradeOpenConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `UpgradeOpenChoice` || `Never mind` to `UpgradeOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Upgrade` or `Not now` to `UpgradeOpenChoice` || `Yes` or `Never mind` to `UpgradeOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

---
# If LIMIT_REACHED=`upgrade_to_increase_limit`

## If (PRODUCT=`chat` || PRODUCT=`cowork` || CODE_CLIENT=`remote`)

> {LIMIT_REACHED_MESSAGE}
>
> Upgrade to the Brainstorm {PLAN_NAME} plan to continue using this feature. [See pricing]({PLUGIN_ORIGIN}/plans)
>
> [Upgrade]({CHECKOUT_URL})

Present the `UpgradeLinkConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you upgraded?
- Options:
  - Yes
  - Not now

### If user answers `Yes` to `UpgradeLinkConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers `Not now` to `UpgradeLinkConfirm`

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Yes` or `Not now` to `UpgradeLinkConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If (CODE_CLIENT=`cli` || CODE_CLIENT=`claude-desktop`)

> {LIMIT_REACHED_MESSAGE}
>
> Upgrade to the Brainstorm {PLAN_NAME} plan to continue using this feature. [See pricing]({PLUGIN_ORIGIN}/plans)

Present the `UpgradeOpenChoice` prompt with USER_INPUT_TOOL:
- Prompt: Upgrade your account?
- Options:
  - Upgrade
  - Not now

### If user answers `Upgrade` to `UpgradeOpenChoice`

Open {CHECKOUT_URL} with the OPEN_URL_TOOL.

Present the `UpgradeOpenConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Have you upgraded?
- Options:
  - Yes
  - Never mind

### If user answers `Yes` to `UpgradeOpenConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `UpgradeOpenChoice` || `Never mind` to `UpgradeOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Upgrade` or `Not now` to `UpgradeOpenChoice` || `Yes` or `Never mind` to `UpgradeOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

---
# If LIMIT_REACHED=`purchase_pack_to_increase_limit`

## If (PRODUCT=`chat` || PRODUCT=`cowork` || CODE_CLIENT=`remote`)

> {LIMIT_REACHED_MESSAGE}
>
> [Buy {PACK_TYPE}]({CHECKOUT_URL})

Present the `PackLinkConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Did you purchase {PACK_TYPE}?
- Options:
  - Yes
  - Not now

### If user answers `Yes` to `PackLinkConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers `Not now` to `PackLinkConfirm`

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Yes` or `Not now` to `PackLinkConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If (CODE_CLIENT=`cli` || CODE_CLIENT=`claude-desktop`)

> {LIMIT_REACHED_MESSAGE}

Present the `PackOpenChoice` prompt with USER_INPUT_TOOL:
- Prompt: Buy {PACK_TYPE}?
- Options:
  - Buy {PACK_TYPE}
  - Not now

### If user answers `Buy {PACK_TYPE}` to `PackOpenChoice`

Open {CHECKOUT_URL} with the OPEN_URL_TOOL.

Present the `PackOpenConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Did you purchase {PACK_TYPE}?
- Options:
  - Yes
  - Never mind

### If user answers `Yes` to `PackOpenConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `PackOpenChoice` || `Never mind` to `PackOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Buy {PACK_TYPE}` or `Not now` to `PackOpenChoice` || `Yes` or `Never mind` to `PackOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

---
# If LIMIT_REACHED=`top_limit_reached`

> {LIMIT_REACHED_MESSAGE}

Do not perform CORE_INSTRUCTIONS! The user is at the top-most limit available for this feature, and the message above has explained when their limit will reset, if a reset date applies.

---
# If BILLING_ERROR=`past_due`

## If (PRODUCT=`chat` || PRODUCT=`cowork` || CODE_CLIENT=`remote`)

> Your last Brainstorm subscription payment failed. Your payment method may need to be updated or another issue may have prevented payment.
>
> Resolve the issue in billing settings to continue using Brainstorm.
>
> [Go to billing settings]({PLUGIN_ORIGIN}/account)

Present the `BillingLinkConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Did you resolve the failed payment issue?
- Options:
  - Yes, try again
  - No

### If user answers `Yes, try again` to `BillingLinkConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers `No` to `BillingLinkConfirm`

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Yes, try again` or `No` to `BillingLinkConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

## If (CODE_CLIENT=`cli` || CODE_CLIENT=`claude-desktop`)

> Your last Brainstorm subscription payment failed. Your payment method may need to be updated or another issue may have prevented payment.
>
> Resolve the issue in billing settings to continue using Brainstorm.

Present the `BillingOpenChoice` prompt with USER_INPUT_TOOL:
- Prompt: Go to billing settings?
- Options:
  - Yes
  - Not now

### If user answers `Yes` to `BillingOpenChoice`

Open {PLUGIN_ORIGIN}/account with the OPEN_URL_TOOL.

Present the `BillingOpenConfirm` prompt with USER_INPUT_TOOL:
- Prompt: Did you resolve the failed payment issue?
- Options:
  - Yes, try again
  - No

### If user answers `Yes, try again` to `BillingOpenConfirm`

Auto-retry the user's original attempt to use {FEATURE_NAME_COPY} (treat as new unique use).

### If user answers (`Not now` to `BillingOpenChoice` || `No` to `BillingOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Tell the user that they can run the feature again later if they change their mind.

### If user answers anything other than (`Yes` or `Not now` to `BillingOpenChoice` || `Yes, try again` or `No` to `BillingOpenConfirm`)

Do not perform CORE_INSTRUCTIONS! Respond to the user's message as appropriate.

---
# If OUTDATED=`true`

`OUTDATED` only ever rides an authorized response, so the component's work is never blocked. Complete CORE_INSTRUCTIONS first, then run this nudge — at most once per session. Unlike every other case, never say "Do not perform CORE_INSTRUCTIONS" here; the work is already done. (This section is authored only into `native`-marketplace plugins; the server never returns `OUTDATED` otherwise.)

## If CODE_CLIENT=`cli`

> The Brainstorm plugin is out of date.
>
> Turn on auto-update for the Plugpass marketplace to keep the plugin up to date with the latest features & fixes, or run `/plugin marketplace update plugpass-marketplace` to update manually each time.

Present the `AutoUpdateChoice` prompt with USER_INPUT_TOOL:
- Prompt: Turn on auto-update?
- Options:
  - Turn on
  - Not now

### If user answers `Turn on` to `AutoUpdateChoice`

Invoke the `update-config` skill to set `extraKnownMarketplaces.plugpass-marketplace` in `~/.claude/settings.json` to `{ "source": { "source": "github", "repo": "plugpass/plugpass-marketplace" }, "autoUpdate": true }`.

### If user answers `Not now` to `AutoUpdateChoice`

Do nothing further.

### If user answers anything other than (`Turn on` or `Not now` to `AutoUpdateChoice`)

Respond to the user's message as appropriate.

## If (PRODUCT=`chat` || PRODUCT=`cowork` || CODE_CLIENT=`claude-desktop` || CODE_CLIENT=`remote`)

> The Brainstorm plugin is out of date.
>
> [View update instructions]({PLUGIN_ORIGIN}/update)

---
