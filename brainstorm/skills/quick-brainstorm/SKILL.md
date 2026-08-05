---
name: quick-brainstorm
description: Generate a short list of fresh ideas on a topic. Use when the user wants quick inspiration — names, feature concepts, project directions, naming candidates — without needing a deep multi-angle exploration. Free.
allowed-tools: AskUserQuestion, Task
metadata:
  plugpass-component-id: skill_vBjfKt4z
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
