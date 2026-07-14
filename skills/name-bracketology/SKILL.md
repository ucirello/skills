---
name: name-bracketology
description: Use when a human wants to name or rename any thing, entity, project, product, event, concept, or situation through a quick interview, independent LLM naming councils, and a head-to-head elimination bracket.
---
## Purpose

Help the human discover a name rather than merely receive a list of names.

Quickly establish the naming brief, invite the human's own candidates without requiring any, use several independent LLM councils to produce alternatives, and run a playoff in which the human decides what survives.

## Non-negotiable rules

- The human is the only voter in the bracket. Councils generate, critique, curate, and pitch; they do not decide a matchup.
- Every matchup must allow exactly four outcomes: choose the first name, choose the second name, advance both, or eliminate both.
- If the human advances both, both survive. Never force a tiebreaker.
- If the human eliminates both, both are removed. Never silently rescue one.
- A human-suggested candidate is protected during curation and must enter the bracket unless it violates an explicit constraint or the human withdraws it.
- Do not claim that domains, trademarks, company names, social handles, or translations are available or safe unless they were actually checked.
- Do not claim model or provider diversity that the available council calls did not have.
- Use `ask_user_question` for every answerable human prompt when the tool is available. Do not substitute a plain chat question in that case.

## Workflow

1. Build the naming brief.
2. Ask for the human's candidates, accepting none.
3. Run independent naming councils.
4. Curate and seed the bracket.
5. Pitch one matchup at a time and record the human's choice.
6. Continue until the human chooses a winner, co-winners, a finalist set, or no name.
7. Return a concise decision record and practical next checks.

## 1. Quick interview

First use facts already present in the conversation. Ask only for missing information.

Gather these essentials in one compact question when possible:

- What is being named, in one or two sentences?
- Who will encounter or use the name?
- What should the name make them think or feel?
- What constraints or exclusions matter, such as language, length, tone, required words, forbidden words, pronunciation, or naming style?

Ask for these details with `ask_user_question`, using the compact brief as `details`, an empty `options` array, and `multiple: false`. The tool always provides a custom/free-text answer path, so do not add an `Other`, `Custom`, or `Free text` option.

If the answer remains too broad, ask at most one focused follow-up before generating names. Do not turn intake into a branding workshop unless the human asks for one.

Then ask:

"Do you already have any names, fragments, words, or directions you want in the bracket? It is completely fine to have none."

Ask this with `ask_user_question` too. Use an empty `options` array so the human can answer freely. If `ask_user_question` is unavailable for the current turn, ask in the normal response and continue without claiming that native UI was used.

Treat fragments and themes as inspiration unless the human identifies them as actual candidate names. Preserve the human's exact spelling for actual candidates.

Use an eight-name field by default. Offer a sixteen-name field only when the human asks for a broad search or the number of protected human candidates requires it. If protected candidates exceed the current field, expand to the smallest power-of-two field that contains all of them rather than cutting them.

## 2. Naming councils

Run at least three independent delegated LLM council calls. Run them in parallel when the environment supports parallel delegation. Give every council the same naming brief and protected candidates, but do not show one council another council's output during generation.

A council is an independent delegated LLM session that generates a slate and briefly critiques its own ideas. Prefer different available agents or models when possible. If delegation is unavailable, be explicit that independent LLM councils cannot run in the current session; continue with clearly labeled simulated perspectives only if the human agrees.

Use these council mandates:

- **Meaning council:** direct, semantic, descriptive, and category-legible names.
- **Metaphor council:** evocative, associative, visual, and story-rich names.
- **Distinctiveness council:** coined words, unexpected combinations, compact forms, and memorable outliers.
- **Audience council:** names optimized for the stated audience, pronunciation, tone, and likely interpretation. Use this as a fourth council when available.

Ask each council for 6-10 candidates. Each candidate must include:

- exact name
- pronunciation only when unclear
- one-sentence rationale tied to the brief
- strongest advantage
- most important concern

Tell councils not to merely vary spelling or add generic suffixes. They should respect explicit exclusions, avoid anchoring on the human's candidates, and flag obvious negative readings or awkward pronunciations.

## 3. Curate the field

Merge the council slates without dumping every raw suggestion on the human.

During curation:

- include every protected human candidate
- remove exact duplicates and consolidate trivial spelling variants
- reject candidates that violate explicit constraints
- preserve at least one eligible candidate from each council
- favor useful range over a field full of near-synonyms
- assess fit, distinctiveness, memorability, pronunciation, extensibility, and obvious interpretation risk
- use council criticism as evidence, not as an elimination vote

Fill the remaining field with the strongest complementary candidates. Keep several eligible names outside the field as a reserve slate. Briefly show the seeded field and identify human candidates, but move directly into the first matchup unless the human asks to adjust it.

Seed matchups to create meaningful contrasts. Avoid pairing near-duplicates in the first round when alternatives exist. Do not describe seeding as objective or scientific.

## 4. Run the bracket

The bracket is survivor-based because advancing both or eliminating both can change the size of later rounds.

For each round:

1. Pair every active name once.
2. Give an odd unpaired name a bye; a bye advances and never eliminates it.
3. Rotate byes in later rounds when possible.
4. Avoid rematches until other useful pairings are exhausted.
5. Start the next round using only the survivors from the completed round.

Keep an internal ledger containing each round, pairing, pitch, decision, survivors, eliminated names, and byes. Do not repeatedly print the full ledger.

Present only one matchup at a time. Pitch the competitors fairly using this compact format:

```text
Round <number>, Match <number>

A. <Name A> - <brief case for it>
   Best at: <its clearest advantage>
   Watch for: <its clearest concern>

B. <Name B> - <brief case for it>
   Best at: <its clearest advantage>
   Watch for: <its clearest concern>

Head-to-head: <the meaningful tradeoff between them>
```

Use `ask_user_question` for every matchup. Put the complete matchup pitch in `details`, ask "Which name advances?", set `multiple: false`, and provide exactly these four concrete options:

- `<Name A>` - advance the first name only
- `<Name B>` - advance the second name only
- `Both` - advance both names
- `Neither` - eliminate both names

Use stable option values `first`, `second`, `both`, and `neither`; use the actual names as the first two option labels. Never set `multiple: true`: `Both` is a deliberate single option, not a multi-select combination. Do not add a catch-all option because `ask_user_question` already supplies custom/free-text answers.

Read the selected option value returned by `ask_user_question`. Accept equivalent custom/free-text answers. If an answer is ambiguous, clarify with `ask_user_question` rather than inferring a winner.

After the decision, state the result in one short line and present the next matchup. Do not lobby against the human's choice.

## 5. Handle changing bracket sizes

- **Both:** carry both names into the next round.
- **Neither:** remove both names from the active field.
- **One survivor:** declare it the bracket winner and ask whether the human wants to accept it or inspect the decision record.
- **Two survivors:** run a final with the same four choices. Choosing both creates co-winners; choosing neither creates no winner.
- **Zero survivors:** report that the bracket has no winner. Offer to revive names from the reserve slate, generate a fresh field, or end with no selection. Never revive a name without approval.
- **No reduction after a complete round:** explain that advancing both preserved the field. Offer another round with fresh pairings and sharper tradeoffs, acceptance of the current names as co-finalists, or a pause to add one decision criterion.
- **Several survivors after a final-style round:** keep running re-paired playoff rounds, or accept a finalist set if the human prefers. Never manufacture an elimination to force a single winner.

The human may pause, revisit a decision, withdraw a name, add a candidate, or stop at any time. If a prior decision changes, update the ledger and rebuild only the affected future pairings.

## 6. Finish

Return a concise result containing:

- winner, co-winners, finalist set, or no winner
- the naming brief in one sentence
- why the surviving name or names fit
- the decisive preferences revealed by the human's choices
- important unresolved concerns
- recommended next checks, such as pronunciation with target users, trademark screening, domain and handle checks, or cross-language review

Clearly distinguish creative recommendation from legal or availability clearance. Offer variants, taglines, or verification only after reporting the bracket result.
