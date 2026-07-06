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

Durable facts about this connector; the build recipe (a fill-in skeleton) is in Phase 5. Call
`layout_get_dsl` once up front for the exact syntax.

- **Place everything at board-absolute coordinates — do not parent items to a frame.** Frame-parenting
  is unreliable across drivers (frame-parented children have come back HTTP 400) and isn't needed — a
  board-absolute layout looks identical. Board center is `(0,0)`; x,y are each item's center.
- **Only three item types are needed:** `TEXT` (title, labels, headings), `STICKY` (records), and
  `CONNECTOR` (axes/lines). **No `SHAPE`** — no background fills, no divider lines. Zones are conveyed
  by sticky color + heading text + axis arrows, nothing else.
- **Axes and lines are connectors between two items** (e.g. the two axis-label texts), never shapes:
  `CONNECTOR from=<item> to=<item> end_cap=arrow`. `from`/`to` are aliases of items defined in the
  same batch (or existing item URLs).
- **`layout_create` is all-or-nothing per item, and its response is your source of truth.** An invalid
  item (a `font=roobert`, a near-zero shape) fails and lands in `failed_items`; check `created_count`
  and `failed_items` rather than trusting a bare success, and re-send only what failed.
- **Colors & fonts.** Sticky colors come from a fixed palette (`yellow`, `orange`, `dark_blue`,
  `light_blue`, `green`, `pink`, `gray`, …); give a sticky either a width or a height, never both.
  Never set `font=` — the default is fine and `roobert` fails.
- **Verifying/reading.** The create response's `failed_items` is authoritative. `board_list_items`
  lists stickies and text (not connectors). Note `context_get` only summarizes boards that have a
  frame — a **frameless board returns "no high-level items,"** so rely on the create response, not
  `context_get`, to confirm a build.
- **Board lifecycle.** Creating a board is irreversible — confirm before creating one. Pass
  `is_repository=true` in a git repo.

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

## Phase 5: Build the board and place the records

Compose the DSL by **filling in this skeleton**. It places everything at **board-absolute coordinates
with no frame** — the arrangement that lands reliably. The rules are baked into its shape; don't add
item types that aren't here (**no `SHAPE`, no `FRAME`, no `parent=`**):

```
# Board-absolute coordinates: board center is (0,0); x,y are each item's CENTER.
# Only TEXT, STICKY, CONNECTOR. No SHAPE (no background fills, no divider lines).
# No FRAME and no parent=. Never set font=. Sticky content is required.
title TEXT x=0 y=<top edge> w=640 size=28 align=center "<p><strong><board title></strong></p>"

# One TEXT per label/heading the setup names (axis ends at the outer edges, headings above each zone):
lblLo TEXT x=<left edge> y=0 w=90 size=24 align=center "<low-axis label>"
lblHi TEXT x=<right edge> y=0 w=90 size=24 align=center "<high-axis label>"

# One STICKY per record at its zone's position; color = the zone's color:
s1 STICKY x=<x> y=<y> w=161 color=<zone> "<p><strong>Record name</strong></p><p>value</p><p>value</p>"
# … one line per record …

# Axes LAST — connectors between the label texts, arrowhead toward the 'high' end:
axisA CONNECTOR from=lblLo to=lblHi end_cap=arrow
```

Lay the zones out around the center (a ~2000-wide × 1200-tall spread works well): each zone's stickies
in a small grid, its heading just above, and the axis labels out at the edges. If the setup says "a
titled frame," render the title as a `TEXT` at the top and skip the frame object.

Sticky content: line 1 is the record **name** (bold); the next 2–3 lines are its key field values.

**Pre-flight gate — before you call `layout_create`, re-scan the DSL you just wrote:**

1. Any `FRAME` or `parent=`? → remove it; everything is board-absolute.
2. Any `SHAPE` (background fill, divider, line)? → remove it; zones = sticky color + headings + axis connectors.
3. Any `font=` (especially `roobert`)? → delete it.
4. Any `CONNECTOR` whose `from`/`to` isn't a `TEXT`/`STICKY` defined above? → fix it.

Submit only once all four are clean. Then **verify from the response**: `failed_items` must be empty
and `created_count` must equal the number of items you sent; re-send anything that failed.

Finally, **return the board URL** so a caller (or the user) can share it — posting to Slack is the
caller's job, not this skill's.

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
