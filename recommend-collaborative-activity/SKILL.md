---
name: recommend-collaborative-activity
description: >-
  Recommends the right collaborative / design-thinking activity for whatever a
  team needs — retros, prioritization, ideation, making sense of research,
  journey mapping, empathy building, or converging on a decision. Takes a plain
  description of the need — a stated goal, a pasted conversation, or a channel
  summary handed over by another skill — matches it against a catalog of
  facilitation activities, suggests the 2–3 best fits with a short rationale, and
  hands back the chosen activity's digital-whiteboard setup (zones, axes,
  colors). Use whenever someone is planning a workshop, retro, or team session
  and asks "what activity should we run," wants to facilitate a discussion, or
  needs to turn a messy situation into a structured exercise — even if they don't
  name a specific method. Other skills call this to pick an activity before they
  build a board or run a session.
---

# Recommend a collaborative activity

Picking the *right* facilitation activity is the difference between a session that moves a team
forward and a pile of sticky notes nobody knows what to do with. This skill reads a description of
what a team needs, matches it to a proven design-thinking activity, and provides the whiteboard
layout so the session can start.

It's deliberately **input-agnostic and I/O-free** — it doesn't read Slack, Salesforce, or Miro. It
reasons over a description you give it and returns a recommendation. That keeps it reusable anywhere:

- **Standalone** — a user describes a goal ("we've got a wall of research and no idea what it
  means") and you recommend an activity.
- **As a building block** — another skill (e.g. `facilitate-session`) reads a Slack channel, summarizes
  it, and passes you that summary. You don't do the reading; you do the matching.

When called as a building block, the output that matters most is the chosen activity's *Digital
Whiteboard Setup* blueprint (Step 4), which the caller uses to lay out the board.

## Output dialect

If your reply is going into Slack, use *mrkdwn*: `*bold*` (single asterisks), no `#` headers, simple
bullets; unicode emoji render fine. In a terminal or Claude.ai, normal markdown is fine. Keep the
recommendation list short and scannable — a few lines, not paragraphs.

## Step 1 — Take the need as input

Work from a short description of what the team is trying to do, discussing, or struggling with. That
description may arrive three ways:

- typed by the user as a goal,
- pasted in as a conversation, or
- handed to you as a channel summary by a calling skill.

Reading a Slack channel is **not this skill's job** — a caller does that and passes you the text. If
you were invoked with no description at all, ask for one briefly:

> In a sentence or two, what is your team trying to do or work through?

## Step 2 — Match the need to activities

Compare the description against the **Activity Catalog** below, matching the team's language to the
*Ideal Conversation Intent* column. Pick the **2–3 best fits** — more than three buries the choice;
fewer hides good options.

## Step 3 — Present the options

Present each candidate with a **one-line rationale drawn from the actual description** — specific to
this team, not a generic definition of the activity. For example:

> Based on what your team is working through, here are 3 activities that could help:
> 1. *Affinity Diagram* — lots of disconnected signals; grouping them by theme could surface patterns.
> 2. *Impact / Effort Matrix* — you keep asking "what to prioritize next"; this lets the team vote with their feet.
> 3. *Rose / Bud / Thorn* — a mix of wins and frustrations; this gives each a place to land.
>
> Which one feels right, or should I pick the best fit?

Wait for the user to choose (or to ask you to pick). Even on "just decide," still show the options
briefly with your pick called out — the reasoning is what lets them course-correct.

## Step 4 — Hand off the blueprint

Once an activity is chosen, return its **Digital Whiteboard Setup** from the catalog — the number of
zones, the axis labels or column structure, and which zones get distinct colors. This is what a
facilitator (or a calling skill like `create-whiteboard`) uses to lay out the board.

If nothing in the catalog fits, say so plainly and suggest a simple grid as a neutral fallback rather
than forcing a poor match.

---

## Activity Catalog

Use the *Ideal Conversation Intent* column to match what a team needs to an activity, and the
*Digital Whiteboard Setup* column as the literal layout blueprint.

| Activity | Purpose | Digital Whiteboard Setup | Ideal Conversation Intent |
|----------|---------|--------------------------|---------------------------|
| **Empathy Map** | Understand users' thoughts, feelings, actions, and pain points to build shared understanding of who you're designing for | 2×2 grid with quadrants: Says, Thinks, Does, Feels — populate with sticky notes from research or interviews | "We don't really know our users," "Let's align on who we're building this for," "I feel like we're making assumptions" |
| **How Might We (HMW)** | Reframe problems as open-ended opportunities to spark creative ideation without anchoring to solutions | One sticky per HMW question, grouped by theme using color coding or proximity clustering | "How do we solve X," "We have a problem with Y," "Let's brainstorm around this challenge" |
| **Crazy 8s** | Generate a high volume of divergent ideas quickly by forcing rapid sketching or writing under time pressure | 8-cell grid per participant — timer widget set to 1 min per cell; use image or sticky note widgets for sketches | "We're stuck on one idea," "We need more options," "Let's think outside the box fast" |
| **Affinity Diagram** | Synthesize large amounts of qualitative data (notes, quotes, observations) into emergent themes | Dump all sticky notes on the board, then cluster by similarity — add label stickies above each cluster as themes emerge | "We have tons of research but don't know what it means," "Help me find patterns," "Let's make sense of all this data" |
| **Journey Map** | Visualize the end-to-end experience of a user across time, highlighting pain points, emotions, and opportunities | Horizontal timeline with rows for: Stages, Actions, Thoughts, Emotions (use a curve/line widget), Pain Points, Opportunities | "Walk me through the user experience," "Where does the user get frustrated," "What's the full picture of their journey" |
| **Impact / Effort Matrix** | Prioritize ideas by weighing potential value against implementation complexity | 2×2 grid: x-axis = Effort (Low→High), y-axis = Impact (Low→High) — place idea stickies in quadrants | "Which ideas should we do first," "We have too many ideas and need to prioritize," "What's the quickest win" |
| **Rose / Bud / Thorn** | Retrospectively evaluate an experience by surfacing positives, potentials, and pain points | Three columns with color-coded stickies: Rose (pink/red) = good, Bud (green) = potential, Thorn (yellow) = pain | "Let's reflect on how that went," "What worked and what didn't," "Quick retro on the last sprint or project" |
| **Point of View (POV) Statement** | Synthesize user research into a focused problem statement that grounds the team on who, what, and why | Text frame or doc widget with the template: [User] needs [need] because [insight] — one per persona | "We need a problem statement," "Let's align on what we're actually solving," "Who are we designing for and why does it matter" |
| **Storyboard** | Communicate how a solution plays out in a real-world scenario, frame by frame, to build empathy and test narrative logic | Grid of numbered frames (image or sticky note widgets) with a brief caption below each — 6–8 frames typical | "Show me what this looks like in real life," "Walk me through the user scenario," "Let's visualize how this solution works end to end" |
| **Dot Voting** | Quickly converge on the group's top priorities or favorites from a large set of options | Place all options as stickies on the board — each participant gets a fixed number of dot stickers (voting widget or emoji reactions) to allocate | "Let's decide as a group," "We need to narrow this down," "Everyone vote on what matters most" |

### Zone-color conventions

Miro sticky colors are referenced by name (e.g. `light_yellow`, `light_green`, `light_pink`,
`light_blue`, `blue`, `orange`, `yellow`). When an activity distinguishes zones by color, pick
distinct, legible colors per zone and keep them consistent. When an activity has no zone distinction
(a plain grid, or Affinity before clustering), use a single color — default `light_yellow` — so the
board stays calm until the team starts organizing.
