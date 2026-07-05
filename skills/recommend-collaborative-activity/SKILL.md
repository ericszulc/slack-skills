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
Whiteboard Setup* (Step 4), which the caller uses to lay out the board.

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

## Step 4 — Hand off the setup

Once an activity is chosen, return its full entry from **Activity setups** below — the frame/layout,
any axes, the zones (their labels and colors), and how items are placed. This is what a facilitator
(or a calling skill like `create-whiteboard`) uses to lay out the board. It's described
tool-agnostically on purpose: it says *what* to build, and the renderer works out the exact sizes and
coordinates.

If nothing in the catalog fits, say so plainly and suggest a simple grid as a neutral fallback rather
than forcing a poor match.

---

## Activity Catalog

Use the *Ideal Conversation Intent* column to match what a team needs to an activity. The *Setup*
column is a one-line summary for scanning; the full, build-ready setup for each activity is in
**Activity setups** below.

| Activity | Purpose | Setup (summary) | Ideal Conversation Intent |
|----------|---------|-----------------|---------------------------|
| **Empathy Map** | Understand users' thoughts, feelings, actions, and pain points to build shared understanding of who you're designing for | 2×2 quadrants: Says / Thinks / Does / Feels | "We don't really know our users," "Let's align on who we're building this for," "I feel like we're making assumptions" |
| **How Might We (HMW)** | Reframe problems as open-ended opportunities to spark creative ideation without anchoring to solutions | Stickies of "How might we…?" questions, grouped into themes | "How do we solve X," "We have a problem with Y," "Let's brainstorm around this challenge" |
| **Crazy 8s** | Generate a high volume of divergent ideas quickly by forcing rapid sketching or writing under time pressure | 8-cell grid per person, one idea per cell, timed | "We're stuck on one idea," "We need more options," "Let's think outside the box fast" |
| **Affinity Diagram** | Synthesize large amounts of qualitative data (notes, quotes, observations) into emergent themes | Loose field of stickies, clustered into labeled themes | "We have tons of research but don't know what it means," "Help me find patterns," "Let's make sense of all this data" |
| **Journey Map** | Visualize the end-to-end experience of a user across time, highlighting pain points, emotions, and opportunities | Horizontal timeline; rows for stages, actions, thoughts, emotions, pain points, opportunities | "Walk me through the user experience," "Where does the user get frustrated," "What's the full picture of their journey" |
| **Impact / Effort Matrix** | Prioritize ideas by weighing potential value against implementation complexity | 2×2 matrix: Effort (x) × Impact (y), four action quadrants | "Which ideas should we do first," "We have too many ideas and need to prioritize," "What's the quickest win" |
| **Rose / Bud / Thorn** | Retrospectively evaluate an experience by surfacing positives, potentials, and pain points | Three colored columns: Rose / Bud / Thorn | "Let's reflect on how that went," "What worked and what didn't," "Quick retro on the last sprint or project" |
| **Point of View (POV) Statement** | Synthesize user research into a focused problem statement that grounds the team on who, what, and why | One "[User] needs [need] because [insight]" block per persona | "We need a problem statement," "Let's align on what we're actually solving," "Who are we designing for and why does it matter" |
| **Storyboard** | Communicate how a solution plays out in a real-world scenario, frame by frame, to build empathy and test narrative logic | 6–8 numbered panels in sequence, a caption under each | "Show me what this looks like in real life," "Walk me through the user scenario," "Let's visualize how this solution works end to end" |
| **Dot Voting** | Quickly converge on the group's top priorities or favorites from a large set of options | All options as stickies; participants allocate dot votes | "Let's decide as a group," "We need to narrow this down," "Everyone vote on what matters most" |

## Activity setups

These are the full, build-ready setups — the *what to build*, described tool-agnostically (no
coordinates, nothing tied to a specific board tool). A renderer like `create-whiteboard` turns each
into an actual board using its own mechanics. Every activity fills the same slots, so the renderer
always knows what to look for:

- **Frame & layout** — the container and overall arrangement
- **Axes** — any scale and its direction, the end labels, and whether to draw arrows (or "none")
- **Zones** — each zone's label, where it sits, its sticky color, and what belongs there
- **Placing items** — how to decide which zone a record goes in, and how to arrange within a zone

Colors are named generically (yellow, blue, …); the renderer maps them to its palette. When an
activity draws no distinction between zones, use one calm color (e.g. light yellow) so the board
stays neutral until the team starts organizing.

### Empathy Map
- **Frame & layout:** one titled frame, "Empathy Map," split into a 2×2 grid of quadrants (optionally a persona name in the center).
- **Axes:** none — the quadrants are categories, not scales.
- **Zones (single sticky color; no priority distinction):** top-left "Says", top-right "Thinks", bottom-left "Does", bottom-right "Feels" — each a labeled quadrant.
- **Placing items:** one observation or quote per sticky in the quadrant it fits. If the source records don't pre-sort into these, place them in a holding row to the side (or leave the quadrants blank) for the team to sort live.

### How Might We (HMW)
- **Frame & layout:** one titled frame, "How Might We"; an open field of stickies grouped into themes by proximity.
- **Axes:** none.
- **Zones:** no fixed zones — theme clusters emerge; add a label sticky above each cluster as it forms.
- **Placing items:** one "How might we…?" question per sticky. From raw records, reframe each into an HMW prompt (or place records as seed material to reframe), then group related ones together.

### Crazy 8s
- **Frame & layout:** one titled frame per participant containing an 8-cell grid (2 rows × 4); a visible one-minute-per-cell timer.
- **Axes:** none.
- **Zones:** eight numbered cells, one idea sketch or sticky each.
- **Placing items:** generative, not record-driven — cells start blank for rapid filling. If seeding, drop a single prompt sticky above the grid rather than pre-filling cells.

### Affinity Diagram
- **Frame & layout:** one titled frame; a wide open field.
- **Axes:** none.
- **Zones:** none at the start — clusters form during the session; each finished cluster gets a label sticky above it as a theme name.
- **Placing items:** lay every record onto the board as a single-color sticky in a loose, even grid; the team clusters them and names the themes. (The build just spreads the stickies out — clustering is team-driven.)

### Journey Map
- **Frame & layout:** one titled frame; a horizontal timeline read left→right, with labeled horizontal rows.
- **Axes:** horizontal = time / journey stages, left→right (an arrow is optional).
- **Zones:** stacked rows, each labeled: Stages, Actions, Thoughts, Emotions, Pain Points, Opportunities. Columns are the stages over time; the Emotions row is often drawn as a rising/falling line.
- **Placing items:** place each sticky in the row × stage cell it describes. If records don't map to stages, seed the Stages row and leave the rest for the team.

### Impact / Effort Matrix
- **Frame & layout:** one titled frame, "Impact / Effort Matrix," split into a 2×2 grid of quadrants.
- **Axes (draw as low→high arrows):** horizontal = *Effort*, increasing left→right; vertical = *Impact*, increasing bottom→top. Label the ends: "Low Effort" / "High Effort", "Low Impact" / "High Impact".
- **Zones (each holds a grid of same-colored stickies under a heading naming the recommended action):** top-left *High Impact, Low Effort* → "Do it now" → high-priority color (yellow); top-right *High Impact, High Effort* → "Do it next" → high-priority color (yellow); bottom-left *Low Impact, Low Effort* → "Do it if/when there is time" → low-priority color (blue); bottom-right *Low Impact, High Effort* → "Don't do it" → caution color (orange).
- **Placing items:** judge each record's impact and effort (from the mapped fields, or ask the user how to score them) and drop it in the matching quadrant; arrange stickies in a simple grid within the quadrant.

### Rose / Bud / Thorn
- **Frame & layout:** one titled frame; three side-by-side columns.
- **Axes:** none.
- **Zones (each column a labeled zone with its own color):** "Rose" (what went well) — pink/red; "Bud" (potential/opportunity) — green; "Thorn" (pain/problem) — yellow.
- **Placing items:** one observation per sticky in the matching column, colored to match. From records, classify each as rose / bud / thorn (from a field, or ask) and place it in that column.

### Point of View (POV) Statement
- **Frame & layout:** one titled frame; a stack of text blocks, one per persona.
- **Axes:** none.
- **Zones:** one block per persona, each holding the sentence template "[User] needs [need] because [insight]".
- **Placing items:** one POV statement per persona or segment; fill the template. From records, group into personas/segments first, one block each.

### Storyboard
- **Frame & layout:** one titled frame; a grid of 6–8 numbered panels read in sequence (left→right, top→bottom), a caption line under each.
- **Axes:** none — order is by panel number.
- **Zones:** numbered panels, each holding a sketch or image and a short caption below.
- **Placing items:** one scene per panel in narrative order. Largely team-drawn; if seeding, put one caption sticky per known step.

### Dot Voting
- **Frame & layout:** one titled frame; all options laid out as stickies in an even grid.
- **Axes:** none.
- **Zones:** none — a single field of option stickies; voting happens on top of them.
- **Placing items:** place each option (or record) as one sticky in the grid, single color. Voting (dot stickers or emoji reactions, a fixed number per person) is added live during the session.
