---
name: facilitate-session
description: >-
  Runs an end-to-end team design-thinking session from Slack. Reads a channel's
  discussion (or a stated goal), picks the right activity via
  `recommend-collaborative-activity`, builds a Miro whiteboard from live
  Salesforce records via `create-whiteboard`, posts the board link back to the
  team, and can later sync the finished board back to Salesforce. Use whenever
  someone wants to run a retro, workshop, or working session for the team, turn a
  Slack conversation into a working board, or collaboratively prioritize, map, or
  cluster Salesforce records — especially inside Slack, where the channel is the
  starting point. This is the front door; it coordinates the focused skills that
  do each piece.
---

# Facilitate a team session

This skill is the **orchestrator**. It owns the Slack side of the work — reading a channel, posting
the board link, and (later) syncing the board back — and it coordinates two focused skills that do
the specialized pieces:

- **`recommend-collaborative-activity`** — turns a description of the team's need into a recommended
  activity plus its whiteboard blueprint. (No I/O; you feed it a summary.)
- **`create-whiteboard`** — turns that blueprint plus Salesforce records into a laid-out Miro board.
  (Owns Salesforce + Miro.)

Keeping those two focused is the whole point: you handle the Slack I/O and the sequencing; they
handle activity logic and board mechanics. If either isn't loaded, degrade gracefully (see below).

## Dependencies and graceful degradation

You call the two skills above by name. If one isn't available in this workspace:

- **`recommend-collaborative-activity` missing** — activity selection is degraded but recoverable.
  Tell the user it's not loaded, then either ask them which activity to run or fall back to a simple
  grid. Don't silently guess.
- **`create-whiteboard` missing** — this is the skill that actually places stickies, so you can't
  build the board yourself. Say so plainly, and offer what you *can*: hand the user the chosen
  activity's blueprint and the fetched record list so they can place them by hand.

## Surface awareness (this usually runs in Slack)

- **Markdown dialect.** Slack uses *mrkdwn*: `*bold*` (single asterisks), no `#` headers or
  `**double asterisks**`, simple bullets; unicode emoji (✅ 🔵) render natively. In a terminal or
  Claude.ai, normal markdown is fine. Match the surface.
- **Brevity.** Slack threads read poorly with long text. Keep the opening question, the activity
  list, and any posted messages short and scannable — a few lines, not paragraphs.

---

## Phase 1: Understand the team's need

Ask this first, and only this:

> Which Slack channel should I read to understand what your team needs right now? (Or describe the
> goal if you'd prefer.)

Then stop and wait — don't stack other questions onto it.

- If a channel is named, fetch its latest ~20 messages and read what the team is discussing,
  struggling with, or trying to accomplish. Distill it into a **two-sentence summary of the need** —
  that summary is what you'll pass to the recommender.
- If no Slack tool is available, ask the user to paste the conversation or describe the goal, and use
  that as the summary instead.

## Phase 2: Pick the activity

Hand your need-summary to **`recommend-collaborative-activity`**. It will return the 2–3 best-fit
activities with a one-line rationale each — present those to the user (keep it scannable) and let
them choose, or ask you to pick the best fit. Once they choose, it returns the activity's *Digital
Whiteboard Setup* blueprint; hold onto that for Phase 4.

Even if the user says "just do it," still surface the options briefly with your pick called out —
skipping activity selection produces a worse session.

(If the recommender isn't loaded, see Dependencies above.)

## Phase 3: Gather board inputs

Now — and only now, after the activity is set — collect the rest:

- **Salesforce object & filter** — what records to fetch. Default to 10 if no count is given.
- **Fields to display** — if unspecified, `create-whiteboard` will pick sensible defaults per object.
- **Miro board** — name, URL, or create a new one.
- **Share the board link?** — ask whether to post the finished board's link to a Slack channel, and
  if so, which. (Skip this if no Slack tool is available.)

## Phase 4: Build the board

Hand the **blueprint** (Phase 2) plus the **Salesforce criteria** and **board** (Phase 3) to
**`create-whiteboard`**. It fetches the records, maps them to zones, places the stickies in one
batch, and returns a confirmation (count + any quadrant breakdown) and the **board URL**. Relay its
confirmation to the user, matched to the surface's markdown dialect.

## Phase 5: Share the link

If the user asked to share it (Phase 3), post the board URL to the chosen Slack channel — a short
message, not a wall of text. If there's no Slack tool, just give the user the URL to share
themselves.

## Phase 6: Sync back (optional, only when asked)

Later, the team moves stickies around during the session. If the user then asks to push those changes
back to Salesforce:

- Re-read the board's current layout and determine each sticky's new zone/quadrant.
- Update the relevant field on each record (e.g. `Description` or a custom priority field) with the
  new classification implied by where it landed.
- Never do this unprompted — moving a sticky mid-discussion isn't automatically a decision to persist.
  Only sync when the user explicitly asks.

## Key principles

- **Front-load activity selection.** The right activity is what makes the session useful; don't skip
  to a bare grid just because the user is in a hurry.
- **One board at a time.** Confirm which board before `create-whiteboard` writes to it.
- **Name what's missing.** If a dependency or integration (recommender, Salesforce, Miro, Slack)
  isn't available, tell the user which piece is missing and offer the fallback — don't stall or fake
  it.
- **Ask before syncing back.** Never write board positions back to Salesforce unless explicitly asked.
- **Keep Slack messages short.** Long blocks read badly in a thread.
