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

FitFindr is an agent that helps thrift shoppers decide whether to buy something and how to style it. A user describes what they want — "vintage graphic tee under $30, size M" — and the agent searches for matches, suggests outfits using pieces they already own, and writes a short caption for posting. If nothing matches, it tells them what to try instead. The system carries information from one tool call to the next, so the item the search found is the exact one that reaches the styling tool.



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

- **What it does:** Filters the listings file by text description, size, and max price. Returns matches only.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float)
- **Returns:** A list of listing dicts, each with keys: id, title, description, price, size, platform, colors, style_tags, brand, category, condition
- **When it has nothing:** Returns an empty list `[]` — never None, never crashes.

### `suggest_outfit`

- **What it does:** Takes a new listing and the user's wardrobe, suggests which wardrobe pieces would pair well with the new item.
- **Inputs:** `new_item` (dict with listing fields), `wardrobe` (dict with `items` key containing list of wardrobe dicts, or empty `{"items": []}`)
- **Returns:** A list of outfit suggestions, each a dict with: `wardrobe_item_id`, `reason` (why it pairs well)
- **When it has nothing:** If wardrobe is empty, return general styling suggestions as a string: "No wardrobe entered yet. This [item type] would pair well with [general style advice]."

### `create_fit_card`

- **What it does:** Writes a short social-media caption for a complete outfit (the new item plus suggested wardrobe pieces).
- **Inputs:** `outfit` (dict with new_item and suggested_wardrobe_items), `new_item` (the listing dict)
- **Returns:** A string caption, 1-3 sentences, suitable for posting to social media. Mentions the item, the styling, and why it works.
- **When it has nothing:** If outfit is incomplete or empty, return: "Could not create a caption — outfit information was incomplete."

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

**Branch rule:**
If search_listings returns an empty list, put a message in the session saying "No listings match that description. Try different keywords, size, or price." and stop. Otherwise, take the first result and pass it to suggest_outfit.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex — extract size with `\b(XS|S|M|L|XL|XXL)\b` and price with `(?:under\s+)?\$?(\d+(?:\.\d{2})?)`

**What moves through the session:** description → search_results → selected_item → outfit_suggestion → fit_card

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'

found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
outfit:   Here are two specific outfit ideas using the new butterfly baby tee and pieces from your wardrobe that lean into that authentic Y2K aesthetic:

### Outfit 1: The Casual Everyday Y2K Look
This outfit balances the fitted, cropped silhouette of the baby tee with relaxed denim for a classic early-2000s street style vibe.

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Footwear:** Chunky white sneakers

fit card: Just scored this cutest little butterfly baby tee on Depop for only $18 and I am officially obsessed! 🦋✨ I love how easy it is to style—you can totally lean into the nostalgic street style with baggy denim and chunky sneakers, or grunge it up with khaki trousers and combat boots. Which vibe are we wearing today? 👇
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are two specific outfit ideas using the new butterfly baby tee and pieces from your wardrobe that lean into that authentic Y2K aesthetic:

### Outfit 1: The Casual Everyday Y2K Look
This outfit balances the fitted, cropped silhouette of the baby tee with relaxed denim for a classic early-2000s street style vibe.

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Footwear:** Chunky white sneakers
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Just scored this cutest little butterfly baby tee on Depop for only $18 and I am officially obsessed! 🦋✨ I love how easy it is to style—you can totally lean into the nostalgic street style with baggy denim and chunky sneakers, or grunge it up with khaki trousers and combat boots. Which vibe are we wearing today? 👇
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* Write the spec for search_listings based on what fields exist in the data file
- *What came back:* A complete tool spec with input types, return value, and empty case
- *What I changed:* Understood that empty list (not None) is what the branch checks, so that's the requirement

**Moment 2**

- *What I asked for:* Help implement the three tools in tools.py following the spec
- *What came back:* Working implementations for search_listings (keyword scoring), suggest_outfit (model call with empty wardrobe handling), create_fit_card (caption generator)
- *What I changed:* Confirmed the branch logic catches empty search results before calling the next tool

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
