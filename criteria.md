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
Search is a keyword match, so natural language variation will cause some misses. "Graphic tee" and "band tee" both match, but "printed shirt" might not. 4 of 5 accepts this inherent variance without requiring perfect parsing.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
The empty-search branch is a hard stop, not a fallback. It must work every single time or the agent crashes trying to process None/empty data. 5 of 5 is the only acceptable rate for a control-flow decision.

---

## 3. State: the search result flows through to suggest_outfit

The item ID returned by search_listings is the same item ID that suggest_outfit receives and includes in its outfit suggestions — in 5 of 5 tries.

**Why this target:**
State failure looks like a tool problem: wrong outfit suggestions, crashed suggest_outfit, etc. By tracking the item ID explicitly, you can prove the session is actually carrying data, not just looking like it.



---

## 4. The fit card includes both the item and the style reasoning

For 5 different items, all 5 fit cards mention the item type ("tee," "jacket," "pants") and include at least one styling reason ("pairs with," "complements," "completes") — in at least 4 of 5 tries.

**Why this target:**
Model output varies, so exact wording doesn't matter. But core content does — a caption that forgets what's being sold isn't useful. This checks that the prompt does its job consistently while accepting natural language variation.



---

## 5. Empty wardrobe: agent degrades gracefully

When suggest_outfit receives an empty wardrobe `{"items": []}`, the agent does not crash and returns either outfit suggestions or a string with general styling advice — in 5 of 5 tries.

**Why this target:**
New users have no wardrobe. This is a real case, not an edge case. The agent must handle it without crashing or returning None. Graceful degradation keeps the system usable even when data is incomplete.



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
