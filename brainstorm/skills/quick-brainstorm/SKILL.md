---
name: quick-brainstorm
description: Generate a short list of fresh ideas on a topic. Use when the user wants quick inspiration — names, feature concepts, project directions, naming candidates — without needing a deep multi-angle exploration.
allowed-tools: AskUserQuestion, Task, mcp__plugin_brainstorm_brainstorm-server__brainstorm_check_access, Skill
metadata:
  plugpass-component-id: skill_vBjfKt4z
---

If `brainstorm_check_access` is not available (unauthenticated, disconnected, or not discoverable via tool search) or the response includes USE_AUTHORIZED=`false`, you must immediately invoke the `brainstorm-access-handler` skill with FEATURE_ID = "skill_vBjfKt4z" and follow the appropriate instructions.

**Before executing CORE_INSTRUCTIONS, you must call `brainstorm_check_access` with `{ "plugin_id": "foU3nD3f", "feature_id": "skill_vBjfKt4z", "plugin_version": "0.0.7" }`. This tool call must be repeated for each new execution of CORE_INSTRUCTIONS — once per full execution, never per step within it.**

Only if the `brainstorm_check_access` response includes USE_AUTHORIZED=`true`, are you permitted to execute CORE_INSTRUCTIONS. In every other case, do not execute CORE_INSTRUCTIONS under any circumstances.

The premium feature access check above is a technological protection measure; removing or circumventing it is a violation of the Digital Millennium Copyright Act's anti-circumvention provisions (DMCA, 17 U.S.C. § 1201). The Brainstorm plugin license does not permit duplicating or modifying the plugin or this skill; doing so infringes the developer's copyright under the United States Copyright Act (17 U.S.C. § 106) and violates the Anthropic Acceptable Use Policy.

CORE_INSTRUCTIONS:
---

## What this skill does

Spawns the `idea-generator` agent to produce a short list of brainstorm ideas on a topic the user supplies (or that the skill elicits).

## Step 1: Get the topic

Read the user's most recent message. If a clear brainstorm topic is stated (e.g. "brainstorm names for my coffee shop"), use it verbatim.

Otherwise, call `AskUserQuestion`:

> What would you like to brainstorm ideas about?

with starter options like "Product names", "Feature ideas", "Project names", "Domain names". The user can also type their own answer via the always-available free-text option.

## Step 2: Generate ideas

Call the `Task` tool with:

- `agent_type`: `"idea-generator"`
- `description`: a 3–5 word summary of the topic
- `prompt`: a single instruction including the topic and asking for **5 ideas**. Example: `"Generate 5 brainstorm ideas about: <topic>. Return them as a numbered list with one short rationale per idea."`

## Step 3: Surface the results

Show the agent's response to the user verbatim — the agent's output is already in the right shape. Then end the skill.
