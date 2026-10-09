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

FitFindr lets a user describe a secondhand clothing item they want — like "vintage graphic tee under $30 size M" — and returns a styled outfit suggestion and a shareable caption. It searches a listings dataset for matching items, uses the user's existing wardrobe to suggest combinations, and formats the result as a short fit card the user could post. The agent runs a three-step planning loop: search, outfit, card.



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

- **What it does:** Filters the listings dataset by keyword, optional size, and optional price ceiling, then returns the best-matching items sorted by keyword overlap score.
- **Inputs:** `description` (str), `size` (str | None), `max_price` (float | None)
- **Returns:** A list of listing dicts, each containing `id` (str), `title` (str), `category` (str), `style_tags` (list), `size` (str), `condition` (str), `price` (float), `colors` (list), `brand` (str or None), `platform` (str) — sorted best match first, at most `config.SEARCH_RESULT_LIMIT` items.
- **When it has nothing:** Returns an empty list `[]` — never `None`, never raises.

### `suggest_outfit`

- **What it does:** Given a listing and the user's wardrobe, asks the model to suggest one or two outfits that pair the new item with pieces the user already owns.
- **Inputs:** `new_item` (dict — a listing dict as returned by `search_listings`), `wardrobe` (dict with an `'items'` key holding a list of wardrobe item dicts)
- **Returns:** A non-empty string containing outfit suggestions (e.g. "Pair these jeans with your white oversized tee and chunky sneakers for a relaxed streetwear look.").
- **When it has nothing:** If `wardrobe['items']` is empty, returns a string of general styling advice for the item — never an empty string, never raises.

### `create_fit_card`

- **What it does:** Writes a two-to-four sentence caption styled like a real social post, mentioning the item, its price, and its platform once each.
- **Inputs:** `outfit` (str — the suggestion string from `suggest_outfit`), `new_item` (dict — the listing dict)
- **Returns:** A non-empty string caption (2–4 sentences) that reads like a post, not a product description.
- **When it has nothing:** If `outfit` is empty or whitespace, returns a descriptive fallback message instead of raising.

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

**Branch rule:** If `search_listings` returns an empty list, put a "no results found" message in the session and stop. Otherwise, take the first result and pass it to `suggest_outfit`. If `suggest_outfit` returns a non-empty string, pass it along with the listing to `create_fit_card` and return its output.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex — `size\s+(\S+)` extracts size, `under\s+\$?(\d+)` extracts max_price, and both are stripped from the query string to produce the description.

**What moves through the session:** `listing` (the first result dict from `search_listings`), then `outfit` (the string from `suggest_outfit`), then `fit_card` (the string from `create_fit_card`).

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two specific outfit ideas using the Y2K Butterfly Baby Tee and pieces from your current wardrobe:

  Outfit 1: 2000s Streetwear Contrast
  Tuck the Y2K Baby Tee into your Baggy straight-leg jeans (dark wash). Cinch the waist with the Brown leather belt.
  Layer the Black cropped zip hoodie over top. Finish with Chunky white sneakers and the Black crossbody bag.

  Outfit 2: Casual Cinch & Grunge
  Pair the Y2K Baby Tee with your Wide-leg khaki trousers. Throw the Vintage black denim jacket over your shoulders,
  and anchor the outfit with the Black combat boots.

  Fit card: Found the ultimate Y2K streetwear piece for only $18.0! This butterfly baby tee looks so good styled
  with baggy denim and a zip hoodie, or dressed down with grunge combat boots. Snagged this on depop and I'm
  officially obsessed with the high-contrast 2000s vibe.
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; results = search_listings('graphic tee', max_price=30); print(f'Found {len(results)} results'); [print(f'  {r[\"title\"]} ${r[\"price\"]}') for r in results[:3]]"

Found 7 results
  Y2K Baby Tee — Butterfly Print $18.0
  Graphic Tee — 2003 Tour Bootleg Style $24.0
  Mesh Long-Sleeve Top — Black $15.0
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe())[:200])"

Here are two ways to style the vintage Levi's 501s using pieces already in your wardrobe, depending on the vibe you're going for:

Outfit 1: Effortless 90s Casual
* The Look: A balanced mix of...
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Nothing beats the feel of truly broken-in vintage denim for that effortless, off-duty model vibe. These medium wash
Levi's 501s are begging to be paired with crisp white sneakers and an oversized tee for the ultimate Sunday coffee
run fit. Grab them on my Depop right now for just $38.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* Help implementing `search_listings` with size filtering.
- *What came back:* A plain substring match — `size.lower() in listing["size"].lower()` — which caused `"s"` to match `"us 9"` and `"l"` to match `"xl"`.
- *What I changed:* Switched to a whole-word regex match using `re.search(r'\b' + re.escape(size_lower) + r'\b', ...)` so sizes only match at word boundaries.

**Moment 2**

- *What I asked for:* Help writing the empty-search error message in `run_agent`.
- *What came back:* A generic "No results found" string.
- *What I changed:* Rewrote it to name specific things the user could change — budget, size, or keywords — based on which filters were actually active in the session.

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
$ python app.py ask 'vintage graphic tee under $30' --trace

[1] parse_query
      in:  dict with keys: query
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
      →    took first result
[4] suggest_outfit
      in:  dict with keys: item, wardrobe_items
      out: Here are two specific outfit ideas using the Y2K Butterfly Baby Tee and pieces from your current wardrobe: ...
[5] create_fit_card
      in:  dict with keys: item
      out: Found the ultimate Y2K streetwear piece for only $18.0! ...
```

**Empty search**

```
$ python app.py ask 'designer ballgown size XXS under $5'

[1] parse_query
      in:  dict with keys: query
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
[3] branch
      →    empty results — stopping before suggest_outfit

  No listings matched your search. You could: raise your budget (currently under $5); or try a different size (currently 'XXS'); or use broader keywords.
```

**On the MCP move:** In `agent.py`, replaced the direct call to `search_listings(description, size, max_price)` with `call_tool("search_listings", {...})` from `mcp_client.py`. The original `.venv` used Python 3.15 alpha which is incompatible with the MCP library — created a new `.venv313` using Python 3.13 which resolved it. The return value was identical to the direct call; nothing downstream changed.



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
