---
name: flowchart-rules
description: Layout rules for flowcharts and architecture diagrams (inline SVG, mermaid, any medium) - orthogonal arrow routing, minimum segment lengths, anchor slots by edge count, clearance and gap minimums (card-to-container on all four sides), node interior (badge + label arrangement, true vertical centring), lane structure, captions as numbered lists, screenshot verification loop. Load BEFORE drawing OR editing any flowchart/diagram, and when iterating on one after feedback. Triggers on "Flowchart", "Diagramm", "flow chart", "draw the architecture", diagram requests in reports/READMEs/artifacts.
---

# Flowchart Rules

Layout rules for every flowchart and architecture diagram, in any medium: hand-written
inline SVG, mermaid, a diagram library, a slide. The ruleset GROWS: when the user corrects
a diagram, the correction lands here as a new dated rule in the same session, never
silently. "Arrow routing", "Crossing discipline" and "Node interior" are binding.
"Structure defaults" are strong defaults that yield to an explicit request.

**Unit:** one *head* = the drawn arrowhead length (≈ 8 px at 1:1 scale). Minimums are
given in heads so they survive scaling.

## Arrow routing (binding)

1. **Orthogonal only.** Every edge is made of horizontal and vertical segments joined by
   90° bends. A diagonal line reads as noise and crosses unrelated content at random
   angles. Curves only when the user asks for one explicitly.
2. **Minimum segment length = 2 heads.** Every segment of a bent edge, the last one
   before the tip especially, is at least twice as long as the arrowhead (≥ 16 px at 1:1).
   A four-pixel stub with a tip on it is a bug, not an arrow.
3. **Anchor slots by edge count, symmetric about the center** (2026-09-02). Edges attach
   to a node's side at evenly spaced slots: with n edges on one side, slot k sits at
   k/(n+1) of that side's length. One edge → center. Two → ⅓ and ⅔. Three → ¼, ½, ¾.
   Never one edge in the center and the other pushed to a side, never an arbitrary offset,
   never a corner. Hand out the slots in the order that avoids crossings: the edge coming
   from the left takes the left slot. When two nodes of different width are joined by
   parallel straight edges, compute the slots on the NARROWER side and let the edges run
   straight; the wider node takes them where they land (still symmetric when the nodes
   are center-aligned). A jog (down, sideways, down) only when a straight run is
   impossible; a jog's short segment obeys rule 2.
4. **Clearance ≥ 2 heads** between an edge and every foreign card border and container
   border (2026-09-02). An arrow hugging the side of a card reads as part of the card.
   A channel between or beside cards therefore needs 2 × clearance of free width before a
   line may run through it; a narrower corridor is not a channel. Clearance does not
   apply to an edge's own endpoints, touching the node it connects to is the point.
5. **Minimum gap between two cards, and between two containers or lanes, = 2 heads** as
   the absolute floor (2026-09-02). A gap that must HOST an edge or a routing channel
   needs more: clearance + line + clearance ≈ 36 px at 1:1. Otherwise the arrow between
   the two cannot render properly (the squashed band-to-lane stub was the incident).
6. **The same 2-head floor applies between a card and the container it sits IN — on ALL
   FOUR sides** (2026-09-09). Left and right get checked because that is where the eye
   starts; **the bottom is the one that gets forgotten**, and a card sitting 10 px above
   its lane's floor reads as falling out of it. Padding may be asymmetric (a lane with a
   title column often has more room at the top) — 2 heads is the floor, not the target.
   Check it numerically for every card, never by eye: `card.y − lane.y`,
   `(lane.y + lane.h) − (card.y + card.h)`, and the same pair horizontally.

## Crossing discipline (binding)

- An edge NEVER passes through a node or card. Route through the free channels instead:
  the gaps between lanes (horizontals), the gaps between cards and the inner margins of
  containers (verticals).
- Piercing a container's border to enter or leave it is fine; that is what borders are for.
- Two edges crossing at 90° is acceptable when unavoidable. Minimize by choosing
  channels, and never let two edges run collinear on top of each other, except as a
  merge trunk (next rule).
- Parallel edges in one channel: separate by ≥ 1 head, never share the exact y or x.
- **Text is an obstacle too** (2026-09-02). An edge never runs through a lane title, an
  edge label, a badge or a caption inside the diagram; keep the same 2-head clearance as
  for cards. A geometry check sees only boxes and lines, so this one is caught on the
  rendered screenshot: read every title and label once and look for a line through it.
- **Merge trunks (funnel), the one allowed overlap** (2026-09-02). When several edges
  end at the SAME input of the same node and mean the same thing (same encoding, same
  kind of label, e.g. "re-dispatch"), they may merge into one trunk. Each branch joins
  the trunk at a 90° tee; the trunk carries ONE arrowhead into ONE anchor slot and
  counts as one edge for the slot rule. Conditions: (a) tees are ≥ 1 head apart and the
  last tee sits ≥ 2 heads before the tip; (b) branches join in arrival order, so no
  branch crosses another to reach the trunk; (c) each branch keeps its own label before
  the tee, the trunk carries at most one summary label ("3 triggers"); (d) edges with a
  different meaning or a different target input never merge, they get their own slots.
  Fan-out mirrors this: one trunk leaves the source and splits at tees. Any other
  overlap is still a bug. A small junction dot at each tee is allowed and helps.

## Node interior (binding, 2026-09-09)

1. **A label alone is centered in its node — horizontally AND vertically.** Not "roughly",
   not "near enough": a label a few pixels off center is the most visible defect a diagram
   can have, and the reader feels it without being able to name it.
2. **Vertical centering is measured on the text's VISUAL center, never on a baseline.**
   A baseline sits below the ink. For n lines of font-size `s` at leading `L`, centered on
   `cy`, the baselines are
   `y_k = cy + 0.35·s − L·(n−1)/2 + k·L`, for k = 0 … n−1.
   Example — 3 lines of 11.5 px at leading 14 in a box whose center is y = 130: baselines
   120 / 134 / 148, not 127 / 141 / 155, which is a full half-line low. SVG has no vertical
   auto-centering, so this arithmetic IS the job; in HTML/CSS use flex centering instead.
3. **A node that also carries a step badge picks ONE of two arrangements and keeps it for
   every node in the diagram:**
   - **stacked** — badge above the label, both horizontally centered on the node's
     vertical axis; or
   - **side by side** — badge in a left gutter, badge and label both vertically centered on
     the node's middle line, the label horizontally centered in the space that is left.

   A badge parked in a corner while the label floats somewhere else is a layout bug, not a
   style (from a real review comment, 2026-09-09: "the number top left and the text
   somewhere half bottom right").
4. **Make the node fit the arrangement, not the other way round.** Side by side needs
   `padding + badge + gap + longest line + padding`; stacked needs
   `padding + badge + gap + n·L + padding`. If it does not fit, **widen or heighten the
   node** — never shrink the type below the diagram's base size, never fall back to the
   corner. Resizing a row of nodes moves every edge anchor hanging off them, so recompute
   the anchors (rule 3) instead of nudging them by eye.

## Structure defaults (proven 2026-09-01/02, orchestrator-worker flow)

- **Actors are swimlanes; authority is geometry.** Put the hub actor in the MIDDLE lane
  so every routed edge visibly passes through it. Humans on top (the only initiators),
  executors below.
- **Passive stores never act.** Files and databases go in a separate bottom strip; a
  store never originates an edge and never sits on an actor-to-actor edge. Split "ours"
  vs. "platform substrate" into two labeled containers.
- **Recurring mechanisms are bands, not actors.** A hook or cron that fires everywhere
  is a dotted ambient band with a ⟳ marker. Giving it a lane invents a daemon that does
  not exist.
- **Loops are routed below or around at 90°** and visually distinct (own color for a
  cycle across lanes; plain for a local retry loop). Label what re-triggers them.
- **Forbidden edges** are drawn and crossed out (red X + one-word label). One glyph
  replaces a paragraph.
- **≤ 2–3 edge encodings** (e.g. solid = action, dashed = file I/O, thick = manual human
  act) plus a legend in the caption. Numbered step badges on the nodes or edges.
- **The caption walks the numbers as a LIST — one row per badge**, the number in its own
  left column like a bullet (2026-09-09). A running sentence-chain makes the reader
  hunt through prose for the number they are looking at; a list lets them jump to it. The
  legend of encodings stays as one line above the list.
- **No decorative endcap glyphs** (⏹ / ● / emoji) on terminators (2026-09-02). A rounded
  node or plain text ends a flow.

## Process

1. **Concept before pixels.** For any non-trivial diagram, first answer (subagent for big
   ones): which actors, which system boundaries as containers or lanes, which ~10
   numbered steps, what stays OUT (and where that content lives instead).
2. **Screenshot-verify EVERY iteration.** Render the real artifact (Playwright against
   the served page or file://), READ the screenshot, fix collisions and overlaps, shoot
   again. A hand-built SVG is never done until a rendered screenshot was inspected.
   Crop-zoom crowded regions.
3. **Hand-authored inline SVG beats diagram libraries** when exact geometry carries
   meaning (mermaid cannot guarantee lane order or segment lengths). For GitHub READMEs,
   export the SVG with an explicit dark background card so it reads on both themes.
4. **Rendering a library diagram (mermaid etc.) that cannot meet the binding rules is a
   hint, not an excuse:** rebuild it by hand when the diagram matters; keep the original
   source collapsible next to it so nothing the author wrote gets lost.
5. **Text stays text** (2026-10-13). Every label is a real SVG `<text>` element, never
   converted to paths/outlines and never baked into a raster. In HTML deliverables inline
   the `<svg>` in the page instead of `<img src=…svg>` or a PNG, so the text stays
   selectable, searchable and translatable. PNG only where the medium forces it (e.g.
   README images on GitHub), with the SVG source committed next to it.