# slack-skills

A small system of composable skills for running **team design-thinking sessions** — read a
discussion, pick the right activity, and turn live Salesforce records into a collaborative Miro
whiteboard. The skills work in any agent, with special attention to running inside **Slack**
(Slackbot).

## The skills

| Skill | Role | Tools it needs |
|-------|------|----------------|
| [`facilitate-session`](facilitate-session/SKILL.md) | Front door — reads the Slack channel, sequences the work, posts the board link, syncs back | **Slack** |
| [`recommend-collaborative-activity`](recommend-collaborative-activity/SKILL.md) | Turns a need-summary into a recommended activity + whiteboard blueprint | **none** |
| [`create-whiteboard`](create-whiteboard/SKILL.md) | Turns a blueprint + Salesforce records into a laid-out Miro board | **Salesforce + Miro** |

`facilitate-session` orchestrates the other two by name. The split keeps each skill focused and
independently reusable, and isolates the Slack dependency to one place so the other two stay
surface-agnostic. If a depended-on skill isn't loaded, `facilitate-session` names what's missing and
falls back rather than failing silently.

## Importing into Slack

Today, a skill in Slack is a **single markdown file you import manually**: upload each `SKILL.md` via
Slack's skill UI — one per skill. They're already self-contained single files (no bundled resources),
so they import as-is.

Reading channel history and posting messages are **native to the Slack bot** — nothing to grant. The
only external setup is the **Salesforce and Miro connectors** that `create-whiteboard` relies on;
those aren't native and must be connected in the workspace (see *Salesforce and Miro* below).

Importing only some of the skills won't break things — `facilitate-session` detects a missing
dependency and falls back — but it's designed to run with all three, so import them together.

### Making this easier (in progress)

We're working toward letting an agent — e.g. **Claude connected to the Slack workspace** — push these
into Slack directly as **canvases**, so importing isn't a manual step. This isn't fully available
yet: an agent can already create a canvas through the Slack connector, but **registering a canvas as
a skill** isn't supported yet, so **manual markdown import is the current path**. When canvas-as-skill
support lands, the same content ports over — the Slack skill-canvas convention is just an H1 title +
`## Steps` + `## Output`, with orchestration referenced by name.

## Using in any other agent

Register all three skill folders (`facilitate-session/`, `recommend-collaborative-activity/`,
`create-whiteboard/`) so the agent can see them; `facilitate-session` calls the other two by name.
Works in the Claude Code terminal, Claude.ai, or anywhere the three load together — no manual import
step.

## Salesforce and Miro

`create-whiteboard` needs Salesforce (read) and Miro (write) tools available. Without them it degrades
gracefully — asking you to paste record data and to name or create the board manually.

## Using the pieces on their own

The two focused skills aren't just helpers for the orchestrator:

- **`recommend-collaborative-activity`** works standalone — "here's what my team is wrestling with,
  what activity should we run?" — anywhere, no integrations required.
- **`create-whiteboard`** works standalone — "put these 10 Opportunities on my Miro board as an
  impact/effort matrix" — when you already know the activity.

Keep them independent; don't fold their logic back into `facilitate-session`.
