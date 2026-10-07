# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
The search is keyword-based, not semantic, so if the query is phrased in a way that doesn't overlap with the title, description, or style tags it might come back empty even when there's a match in the data. 4 of 5 leaves room for one bad phrasing without calling the whole thing broken.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
This path doesn't involve the model at all — the branch is a plain list length check. There's no randomness, so it should either always work or never work. 5 of 5 is the right target because if it misses once the branch logic is wrong, not unlucky.

---

## 3. Something about state

Given a query that returns at least one result, `session["selected_item"]["id"]` must match the `id` of the item that was passed into `suggest_outfit` — in 5 of 5 tries.

**Why this target:**
State passing is deterministic — the loop either puts the first search result in `session["selected_item"]` and passes it to `suggest_outfit` or it doesn't. There's no model call involved in this step so there's no excuse for it to be flaky. 5 of 5 is correct.

---

## 4. Something about the fit card

Given a matching query, the fit card must mention the item's price and platform at least once — in at least 4 of 5 tries.

**Why this target:**
The model can word the caption differently each time and that's fine, but if it drops the price or the platform it's not doing the job of the tool — the whole point is to tell someone where to buy it and what it costs. 4 of 5 because the model occasionally ignores prompt instructions even when they're clear.

---

## 5. Your choice

Given a query with a price ceiling (e.g. "under $30"), every item in `session["search_results"]` must have a price at or below that ceiling — in 5 of 5 tries.

**Why this target:**
Price filtering is a plain comparison — `item["price"] <= max_price`. Either every result passes the filter or the filter has a bug. There's no model involved, no keyword fuzziness, so 5 of 5 is the right bar. If it misses once the implementation is wrong.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
