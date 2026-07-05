---
name: create-whiteboard
description: >-
  Places live Salesforce records onto a Miro board as sticky notes, laid out to
  follow a given design-thinking activity blueprint (an Impact/Effort matrix,
  Rose/Bud/Thorn columns, an affinity grid, etc.) — or a simple grid if no
  activity is specified. Fetches the requested records, finds or creates the
  board, maps each record to the right zone by its field values, and creates the
  stickies in one batch. Use whenever someone wants to turn Salesforce records
  (Opportunities, Cases, Accounts, anything) into sticky notes on a Miro board,
  "get these records into Miro," or build the board for a session whose activity
  is already chosen. Called by `facilitate-session` after an activity is picked,
  or directly when you already know the activity and the records.
---

# Create whiteboard: Salesforce records → Miro sticky notes

This skill is the **mechanics** of building a board: given an activity blueprint and some Salesforce
criteria, it fetches the records and lays them out as stickies. It owns two integrations —
**Salesforce** (read) and **Miro** (write) — and nothing else.

Deciding *which* activity to run is a separate concern handled by `recommend-collaborative-activity`.
This skill assumes the activity is already chosen and works from its blueprint. If you're invoked
without one, see Phase 1 — ask which activity to run, or fall back to a simple grid.

## What this skill expects as input

- **An activity setup** — the *Digital Whiteboard Setup* for the chosen activity, in its standard
  slots: *frame & layout*, *axes*, *zones* (their labels and colors), and a *placing-items* rule.
  Usually handed over by `recommend-collaborative-activity` (often via `facilitate-session`).
  Optional — without it, you'll use a grid. You don't need to know the activity by name; just render
  the slots.
- **Salesforce criteria** — which records to fetch (object + filter + sort) and which fields to show.
- **A Miro board** — a name, a URL, or permission to create one.

## Two modes

- **ALWAYS** — Works standalone. The user can name any Salesforce object and any Miro board, or paste
  record data by hand, and you'll build the board.
- **CONNECTED** — With Salesforce and Miro tools both active, you fetch live records and place fully
  populated stickies directly, no copying.

## Output dialect

If your confirmation is going into Slack, use *mrkdwn*: `*bold*` (single asterisks), no `#` headers,
unicode emoji are fine. In a terminal or Claude.ai, normal markdown is fine. Match the surface.

## Working with the Miro connector

These rules are general to any board you build through the Miro connector (the layout-DSL–based MCP).
Most build failures come from missing one, so keep them in mind across Phases 3–5.

- **Creation is DSL-only — there is no per-item tool.** There's no "create sticky / shape / frame"
  call. You build items by writing DSL and passing it to `layout_create`. Call `layout_get_dsl`
  **once** at the start to load the current syntax, then create the frame and all its children in a
  single `layout_create` batch.
- **Coordinates, and the bounds rule that fails whole batches.** Frames and other board-level items
  use board-absolute coordinates (board center = `(0,0)`). Items with `parent=<frame>` use
  coordinates from the frame's **top-left corner**, and x,y mark the item's **center**. Every child's
  center must fall inside the frame (`0 ≤ x ≤ frame_w`, `0 ≤ y ≤ frame_h`) — one out-of-bounds item
  fails the **entire** call. Plan the layout to fit before sending, and check the returned
  `failed_items`.
- **Stickies.** Give a sticky **either a width or a height, never both** (aspect is fixed). Use a
  color from the connector's fixed palette (e.g. `yellow`, `orange`, `dark_blue`, `light_blue`,
  `green`, `pink`, `gray`). Content is required — pass `" "` for an intentionally blank slot.
- **Text and fonts.** Miro's default brand font `roobert` is **not** selectable in the DSL and fails
  if you name it — omit `font` and let it default. Use `<strong>…</strong>` for bold and
  `<p>…</p><p>…</p>` for multiple lines in one text item.
- **Connectors are invisible to the item-listing tools — use `context_get` to see them.** Connectors
  (axis arrows, links) are **not** returned by `board_list_items` (its type list has none) and were
  missed by `layout_read` too; only `context_get`'s overview reveals them. So when inspecting a
  board, never conclude "there are no axes/arrows" from the item list — confirm with `context_get`.
  To create one, add `CONNECTOR from=<item> to=<item> shape=straight end_cap=arrow`; `from`/`to`
  accept full URLs of items already on the board, and `end_cap=arrow` gives a single arrowhead.
- **Reading a board takes more than one tool.** Find it with `board_search_boards`; get exact
  geometry/style of stickies and text with `board_list_items`; get connectors **and** the semantic
  read (what the zones mean) with `context_get`; use `layout_read` (target a frame via
  `?moveToWidget=<id>`) to get items back as reusable DSL for cloning or editing.
- **Board lifecycle.** Creating a board is irreversible — confirm with the user before `board_create`.
  After building, `context_get` the result to verify it reads back as intended (and that connectors
  actually landed). Pass `is_repository=true` on these calls when operating inside a git repository.

---

## Phase 1: Confirm the blueprint and inputs

- **Blueprint.** If you were given an activity blueprint, use it as-is. If not, ask the user which
  activity they want (or, if they just want the records on a board, use a simple grid — rows of 4–5,
  single color). Don't invent an elaborate layout the user didn't ask for.
- **Salesforce criteria.** Confirm the object and filter (e.g. "10 most recent Opportunities," "open
  Cases assigned to me"). Default to **10 records** if no count is given. For fields, if unspecified
  use the 3–4 most meaningful for the object (see Key Principles).
- **Board.** Confirm the board by name or URL, or offer to create one. Always confirm *which* board
  before writing — stickies are hard to cleanly remove once placed.

## Phase 2: Fetch records from Salesforce

- Query for the requested records, ordered by `CreatedDate DESC` (or the user's preferred sort).
- Fetch **only** the fields that will appear on the stickies — don't over-fetch.
- Cap at **20 records**. If the user asks for more, ask them to narrow the filter rather than
  crowding the board past what a team can work with.
- If Salesforce isn't available, ask the user to paste the record data manually and continue.

## Phase 3: Find the Miro board

- Search by board name, or use the URL if provided.
- If a matrix frame already exists on the board, read its layout to learn the quadrant boundaries
  (the x/y center point) so your placements line up with what's drawn. Detect the axes with
  `context_get` — connector arrows don't appear in the item list (see Working with the Miro connector).
- If there's no matrix frame and the blueprint calls for a quadrant layout, create the stickies in a
  logical grid near the board center.
- If the board can't be found, ask the user to confirm the name or provide the URL.

## Phase 4: Map records to positions

Work purely from the setup you were handed — don't hard-code activity knowledge. The setup gives you
the zones (labels + colors), any axes, and a **placing-items** rule; follow those.

- For each record, apply the setup's placing-items rule to pick its zone. Usually that means reading
  the record's fields (or asking the user how to judge them when the setup calls for it) and matching
  the result to a zone. If the rule is team-driven — nothing maps cleanly — spread the records evenly
  in a holding area or a single zone for the team to sort live.
- Color each sticky with its zone's color. Where the setup draws no distinction between zones, use one
  consistent color — `light_yellow`.
- Within a zone, arrange stickies in a simple grid.

**No setup / grid fallback:** place stickies in rows of 4–5, evenly spaced, single color
(`light_yellow`).

## Phase 5: Create the stickies and return the board

- Create all stickies in **one batch operation** where the Miro tool allows it — it's faster and
  keeps the board from rendering piecemeal.
- Each sticky shows:
  - Line 1 (**bold**): the record name
  - Lines 2–4: key field values in `label: value` format
- Size 160×160px (square), centered text.
- When placing inside a frame, position relative to the frame's top-left corner and stay within its
  bounds.
- After creating, confirm how many stickies landed and on which board, and **return the board URL**
  so a caller (or the user) can share it. Posting that link to Slack is the caller's job, not this
  skill's.

## Output

Reply with a brief confirmation, matched to the surface (Slack mrkdwn vs. GitHub markdown). When the
setup has distinct zones, add a short per-zone breakdown so the reader sees where things landed:

```
✅ Placed [N] stickies on [Board Name].

By zone:
• [Zone label]: [record names]
• [Zone label]: [record names]
```

For a plain grid (no zones), just confirm the count and the board name. Include the board URL either way.

## Key principles

- **Cap at 20 records.** Never place more than 20 stickies in one run — a board past that is hard
  for a team to work with. Ask the user to narrow the filter instead.
- **Fetch only what you display.** Query just the fields that appear on the stickies. It's faster and
  avoids pulling data the session doesn't need.
- **Default to sensible fields.** Opportunities → Name, Amount, Stage, Close Date. Cases → Subject,
  Status, Priority. Accounts → Name, Industry, Annual Revenue. Adapt to whatever object you're given.
- **One board at a time.** Always confirm which board before writing to it.
- **Fail gracefully.** If Miro or Salesforce is unavailable, say so plainly and offer the fallback
  (paste records manually) rather than stalling.
