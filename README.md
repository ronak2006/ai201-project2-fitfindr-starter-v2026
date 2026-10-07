# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->



---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Looks through all 40 listings and returns the ones that match what the user is looking for, filtered by size and price if given.
- **Inputs:** `description` (str) — the keywords from the user's query like "vintage graphic tee"; `size` (str or None) — size to filter by, case-insensitive, None means skip size filtering; `max_price` (float or None) — highest price to include, inclusive, None means no price limit.
- **Returns:** A list of listing dicts sorted by how well they match, best first, capped at `config.SEARCH_RESULT_LIMIT`. Each dict has: `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, `platform`. Note that `brand` can be None — a lot of thrift listings don't have one.
- **When it has nothing:** Returns an empty list. Not None, not an error, just `[]`. The loop checks for this.

### `suggest_outfit`

- **What it does:** Takes the item the user found and their wardrobe and asks the model to suggest one or two outfits using pieces they already own.
- **Inputs:** `new_item` (dict) — the listing dict for the item being considered; `wardrobe` (dict) — has an `items` key with a list of wardrobe item dicts, can be empty.
- **Returns:** A non-empty string from the model with outfit ideas. If the wardrobe has items it names specific pieces. If the wardrobe is empty it gives general styling advice instead.
- **When it has nothing:** If `wardrobe["items"]` is empty, still returns a string with general advice — not an empty string, not a crash.

### `create_fit_card`

- **What it does:** Writes a short caption about the thrift find that sounds like something you'd actually post.
- **Inputs:** `outfit` (str) — the suggestion string that came back from `suggest_outfit`; `new_item` (dict) — the listing dict for the item.
- **Returns:** A 2–4 sentence string that mentions the item, its price, and its platform once each. Should vary between runs since temperature is > 0 and cache is off.
- **When it has nothing:** If `outfit` is empty or just whitespace, returns a short message saying it couldn't make a card instead of crashing.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If `search_listings` returns an empty list, write a message into `session["error"]` telling the user to try different keywords or loosen the filters, then return the session right there. Don't call `suggest_outfit` with nothing. If there are results, take the first one, put it in `session["selected_item"]`, and keep going.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** The query gets passed to the model with a prompt asking it to pull out three things — a description string, a size if one was mentioned, and a max price if one was mentioned. The model returns those as structured fields which go into `session["parsed"]`.

**What moves through the session:** `query` is set at the start → `parsed` gets filled after the model reads the query → `search_results` comes back from `search_listings` → `selected_item` is the first result → `outfit_suggestion` comes from `suggest_outfit` → `fit_card` comes from `create_fit_card`. If anything goes wrong early, `error` gets set and the later fields stay None.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask '...'

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; results = search_listings('graphic tee', max_price=30); print(f'{len(results)} results'); [print(f'  {r[\"title\"]} — \${r[\"price\"]}') for r in results]"
7 results
  Y2K Baby Tee — Butterfly Print — $18.0
  Graphic Tee — 2003 Tour Bootleg Style — $24.0
  Mesh Long-Sleeve Top — Black — $15.0
  Vintage Band Tee — Faded Grey — $19.0
  Low-Rise Cargo Pants — Khaki — $27.0
  Oversized Crewneck Sweatshirt — Vintage Navy — $20.0
  Vintage Graphic Hoodie — Faded Black — $26.0
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import load_listings, get_example_wardrobe; print(suggest_outfit(load_listings()[5], get_example_wardrobe()))"
Here are two casual outfit combinations using the new graphic tee and pieces from your current wardrobe:

**Outfit 1: Grunge Streetwear**
*   **Top:** Graphic Tee — 2003 Tour Bootleg Style
*   **Bottoms:** Baggy straight-leg jeans, dark wash
*   **Outerwear:** Vintage black denim jacket (worn open)
*   **Shoes:** Black combat boots
*   **Accessories:** Black crossbody bag

**Outfit 2: Effortless Casual**
*   **Top:** Graphic Tee — 2003 Tour Bootleg Style (tucked in slightly)
*   **Bottoms:** Wide-leg khaki trousers
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Brown leather belt
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('baggy dark wash jeans and black combat boots', load_listings()[5]))"
Found the absolute holy grail of 2003 tour bootleg style tees while thrifting today and my grunge era heart is so happy.
Throwing this on with some baggy dark wash jeans and black combat boots for the ultimate effortlessly cool vibe.
Just listed this graphic tee over on my depop for $24.0—grab it before I change my mind and keep it!
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
