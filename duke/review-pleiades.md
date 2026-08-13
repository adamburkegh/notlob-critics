# Three Readers and a Water Clock

### A review of `pleiades`

*Duke Fox, July 2026. Reviewed against notlob 0.5.4. Two modules, ~540 lines, a genuine reproduction of a published astronomy result — and a genesis, preserved in a pre-agent seed, that turns the project into a controlled experiment on what a build catches, what a reviewer catches, and what nobody catches at all.*

---

## Provenance note

This is the first review in the collection written with the project's *genesis* in hand. The author supplied the pre-agent seed — the hand-authored specification, ninety-seven lines, two empty function stubs and two placeholder `~example`s — alongside the finished project, and reconstructed the authoring order directly rather than leaving me to infer it. Where earlier reviews speculated about how an agent treated an artefact, this one can check. I have pinned every finding to notlob 0.5.4 and used the notlob `CHANGELOG.md` to separate what is true of the project from what was merely true of the tool on the day it was built. That distinction turns out to be the review's spine, which is fitting, since it is also the project's.

---

## I. What it is, and that it is good

`pleiades` reproduces Cuntz, Gurdemir and George's 2016 seasonal dating of Sappho's Midnight Poem — the fragment that reports the Pleiades already set at midnight — and then does the thing a reproduction should do and a press release never does: it finds where the published single date is softer than claimed.

The original lands on 25 January 570 BC. This project reproduces it, and immediately complicates it. With atmospheric refraction modelled — which the paper's own figure includes — the answer is the 26th; the exact published 25th falls out only with refraction switched off, which the paper did not do, and the module says so plainly and refuses to read the coincidence as vindication. It then adds a horizon correction the paper waves away (Mt Olympus of Lesbos stands to the west, where the Pleiades set, so the effective horizon is higher and the constellation sets earlier), and gets the 23rd. And it sweeps the two inputs the paper only loosely bounds — Mytilene's elevation, and what "midnight" meant on a water clock — and shows the first moves the answer by two days and the second by a *month*.

The verdict the project reaches is better scholarship than the number it started from:

> *Whether it was Sappho or another poet, and whether she was sixteen or sixty, we can imagine her in the sharp Mediterranean winter air, looking at the stars, alone, remembering.*

That sentence would be sentimental if the code beneath it had not just spent four hundred lines earning the right to say "we do not actually know the day, and here is precisely how much we do not know it." A January–February window, stable across Sappho's adult life, is the real result. The single date was always an artefact of pretending loosely-bounded inputs were exact. The project's contribution is to make the pretending visible, and to do it as *executable* sensitivity analysis: every claim in the argument is a claim you can run.

This is, for what it is worth, the most intellectually honest project in the collection, and the one where the literate form does the most argumentative work. The essay is not commentary on the code. The essay is a paper, and the code is its apparatus, and the `~example`s are its figures.

---

## II. The genesis, which is the interesting part

The seed is ninety-seven lines and it is almost all holes. `calculate_date` returns `None` above a commented-out `# return (-570,1,27)`. `horizon_adjust` returns `None`. `MT_OLYMPUS_OF_LESBOS = None # lat/long TODO`. Two `~example`s assert placeholder answers — `(-570,1,27)`, `563` — that are not the real result and, in one case, not even the right *type*. This is deliberate: a consciously test-driven start, red before green, two failing assertions nailed to the wall as targets. A coding agent (Claude Code, Sonnet 5) was then introduced and grew the project to its finished state over about a day.

The seed also contained, in the author's own hand and before any agent touched it, three inconsistencies. What happened to each of them is the review.

**The type-signature inconsistency was caught first, by the coding agent.** The seed declares `calculate_date -> tuple` while the `~example` compares its result to a bare tuple of integers, and the eventual design needed a `HistoricalDate`. This the agent picked up very early — because it *had* to. You cannot write the body of a function without committing to what it returns, and the moment the agent began implementing, the declared type was load-bearing and its collision with the assertion was in the path of the work. It surfaced in the first pass, as such things always do. A signature is a claim the implementation cannot proceed without resolving.

**The `Alcoyne` misspelling was not caught by the agent at all.** The seed names the star constant `PLEIADES_ALCOYNE` — a fingerslip for Alcyone, the brightest Pleiad. The coding agent propagated it faithfully, four occurrences, start to finish, and never flagged it. It never had reason to: `PLEIADES_ALCOYNE` is a perfectly valid Python identifier, and the actual star lookup keys off the Hipparcos number `17702`, not the name. The constant's *label* was inert — decoration the code never had to reconcile with the world. The build ran green with the star misnamed, because nothing the build does requires the name to be right.

It was caught in *review*, by Teri — the other critic in this experiment — and only then fixed, by hand, late, after the author already considered the project finalised.

**The `Myteline` misspelling was caught by nobody, and is still in the shipped file.** Mytilene, the town on Lesbos, appears misspelled "Myteline" five times in the finished prose (and correctly as "Mytilene" five other times — the file cannot agree with itself). The coding agent preserved it, exactly as it preserved Alcoyne, and for exactly the same reason: it is prose, off the executable surface, and nothing the agent had to *do* ever forced it to adjudicate a place-name. Teri's review did not surface it. `notlob check` under 0.5.4 does not surface it — I ran it; the report is clean. It took a third reader, in this session, to notice.

---

## III. What the three fates mean

Line these up and they are not three anecdotes. They are one law, observed three times at different depths:

> **Each layer catches exactly what obstructs it, and nothing else.**

The build caught the signature because the signature obstructed the build. It missed both misspellings because a misspelling obstructs nothing that compiles. That is the same seam every review in this collection keeps arriving at from a different direction — reference versus adjacency, claim versus prose, name versus number — here rendered as *what an agent maintains versus what it lets stand*. The agent is not careless about Myteline and diligent about the signature. It is running a single discipline, gated entirely by whether the inconsistency blocks execution. The structural error had a failure mode. The nominal errors did not.

`Alcoyne` is the specimen that proves it, because it is the one that *migrated*. It began as pure nominal rot, indistinguishable from Myteline — a garbled name in a string of characters. Had it stayed a bare constant it would have survived like Myteline did. What exposed it was not the build and not the checker but a *reader with the world in their head*, because "Alcyone is a star and Alcoyne is a slip of the finger" is world-knowledge, not graph-knowledge. `notlob check` guarantees the graph is internally consistent: that references resolve, that names don't collide, that no symbol is a near-duplicate typo of another. It cannot guarantee the graph *corresponds to reality*, because correspondence is not a property of the graph. `PLEIADES_ALCOYNE` resolves perfectly. It is consistent. It is also wrong about the sky, and only something that knows the sky can tell.

So the escalation ladder, complete:

- **The build** catches what stops the code running. (Signature.)
- **The reviewer** catches what is consistent but false about the world. (Alcoyne — caught by Teri.)
- **And what is false about the world, obstructs nothing, and happens not to draw any single reviewer's eye** falls through every layer until enough independent readers pass over it. (Myteline — caught by neither agent nor checker nor the first reviewer; caught here only because this is the *second* critical reading.)

This is the argument for why the collection has more than one critic in it, discharged on a real artefact instead of asserted. Teri caught the star; I caught the town and the fixture bug below. Neither of us alone cleared the file. Two readers with the world in their heads, triangulating, closed more than either would have — and the residual after both of us is presumably still non-empty, which is the honest and slightly uncomfortable conclusion. There is no pass that closes prose-to-world correspondence. There is only more readers, with diminishing returns and no zero.

---

## IV. The version trap, and a thing the tool did *for* the file without asking

Before this project's binding is judged, a caution the collection has had to issue before: read the artefact against the tool of its own moment.

`pleiades` was authored against a notlob some months older than 0.5.4. Its `binding.lob` declares only `~language python`. Its two `HistoricalDate` properties use bare `@given` and `st.integers` with no import and no `~property-testing` declaration. Run that project against the notlob *of its authoring era* and both properties **error** — `name 'given' is not defined` — because that older runner injected the hypothesis namespace only when the binding declared `~property-testing hypothesis`. On first reading I took this for a live defect, the affordance-versus-intent mismatch that sank the properties in `pn-chomper` and `chatim`.

It is not a defect, and the reason is instructive. The notlob `CHANGELOG.md` records that in 0.5.2 the per-binding `~property-testing` declaration was *removed*: hypothesis became implicit in the Python toolchain, injected whenever the language is Python. So I ran the unchanged project against 0.5.4, and both properties **pass**. The binding file was never touched. The `.lob` source is byte-for-byte what the author shipped. What changed was the ground beneath it.

Sit with that, because it is the whole diachronic problem of this collection turned to face the tool itself. A synchronic document — a `.lob` file describes what *is* — had its meaning altered by an edit made somewhere else, at a later time, that it has no record of and no reference to. The file that erred in June passes in July without a single character moving inside it. This is the exact structure of the eight-versus-five and the Myteline typo, one level up: prose (or here, a binding) stranded by a change it was not present for. The difference is that this time the drift ran *toward* correctness — the tool retroactively fixed the project — which makes it comfortable, and no less an instance of a document meaning something its author did not write.

There is a second, quieter victim of the same class, and it is not even in the code. `pleiades` vendors a copy of `notlob-docs/LANGUAGE.md`, and that copy still documents `~property-testing hypothesis` and `~unit-testing pytest` in two worked examples and a declaration table — syntax the language deleted in 0.5.2. The vendored reference is stale. It teaches a reader of *this project* to write bindings the current tool rejects as parse errors. And the `CHANGELOG` shows the notlob maintainers hit precisely this on their *own* `LANGUAGE.md` — it too was out of date in the same places — and responded not by promising to be careful but by adding `test_language_md_currency.py`, a drift detector that extracts the real sigil vocabulary from the grammar and asserts the reference mentions each item. A checker for the prose-about-the-tool, born of the same rot the tool exists to fight. The project's vendored copy predates and does not benefit from that guard, and is the collection's cleanest example yet of documentation drift: not wrong when written, wrong when read.

---

## V. The version-independent finding

One prose defect here owes nothing to any tool version and would survive any bump, because it is drift internal to the file — and it is a lovely small specimen.

The horizon-geometry tests use three synthetic fixture points, `HERE`, `NEAR`, `FAR`. The `## Test Fixtures` section defines them and explains, correctly, why they live there:

> *The tests appendix below only holds assertions, not definitions, so these synthetic points … live here instead.*

Eleven lines later, the `#Tests` section header introduces the same fixtures:

> *Geometry checks for `horizon_adjust`, using the synthetic points defined in ##Horizon Adjustment.*

They are not defined in Horizon Adjustment. They are defined in Test Fixtures — the section whose own prose says so, ten lines above. The fixtures were, at some point, moved into a section of their own; one pointer was updated to announce the move, and the other pointer, the one in the `#Tests` header, still points at where they used to live. Two sentences in one file, eleven lines apart, disagree about the location of three variables, and the stale one is the more prominent — it is the header a reader hits first.

This is the eight-versus-five in miniature and it confirms the mechanism precisely. The `##Horizon Adjustment` reference is *prose-to-prose* — an English mention of a section name, not a `#`-graph reference. Notlob does not check it and, under its current design, cannot: it is not an edge in the name-graph, it is a word in a sentence. Had the fixtures section been referenced by a real `#`-link, the move would have either updated it or broken the build. Because it was referenced by prose, the move silently orphaned it, and every checker in the tool is blind to the orphan. `notlob check` is clean. The pointer is wrong. Both statements are true at once, and the space between them is the whole subject of this collection.

---

## VI. Smaller notes

The refraction story, by contrast, is *sound* throughout, and deserves saying so, since the collection is quick to catch drift and should be as quick to credit its absence. The abstract says refraction-on gives the 26th and off gives the 25th; the default in `calculate_date` is `refraction=True`; the first `~example` asserts the 26th and the second, with `refraction=False`, asserts the 25th. Prose, signature, and claims agree end to end. This is what the format looks like when the passes stay in sync: the number in the prose and the number in the assertion are the same number, and they are the same because the author kept them so.

`dating.lob` at 450 lines is at the top of the sustainable band, but the length is earned — it is one argument, and splitting the sensitivity sweeps out of the module that defines `calculate_date` would separate the claims from the thing they make claims about. Leave it. `historical/date.lob` is a clean, well-propertied small module; its `astronomical_year_tracks_calendar_year_within_an_era` property is a real monotonicity relation across the BC/AD boundary, exactly the kind of invariant that is easy to get subtly wrong and worth pinning.

The `## Development Process Notes` section documents that a coding agent was introduced and that the project began test-first. It does *not* document that the agent made — or, as it turns out, declined to make — spelling and naming decisions on the author's behalf, nor that a review pass corrected a star's name after the author thought the work done. That absence is not a fault to be scolded; it is the ordinary condition, and it is exactly the diachronic material the style guide argues has nowhere to live inside a synchronic document. The genesis of this project is recoverable only because the author kept a pre-agent seed and remembered the order of events. Take away either and it is gone. The file itself remembers none of it, and is not built to.

---

## VII. Verdict

`pleiades` is the strongest *argument* in the collection and, not coincidentally, its best demonstration that the literate form can carry real intellectual work: a reproduction study whose sensitivity analysis is executable, whose honesty about its own invented parameters is exact, and whose conclusion is more defensible than the published result it started from. On the merits of what it set out to do, it succeeds, and the Python properties that error against its birth-version pass cleanly against 0.5.4.

Its defects are almost entirely of the class this collection exists to name, and it supplies the clearest instance yet of each:

1. **The escalation ladder of who-catches-what** — build, reviewer, second reviewer — observed on three real inconsistencies with three different fates, and proving that each layer catches only what obstructs it. Alcyone was caught by a critic, not a checker, because correspondence to the world is not a graph property. Myteline was caught by neither, and is still in the file.
2. **A synchronic document silently re-meant by a diachronic change elsewhere** — the binding that erred in June and passes in July, character-unchanged, because the tool moved beneath it. Comfortable here, because the drift ran toward correctness. The mechanism is identical to the failures where it does not.
3. **Prose-to-prose reference rot the graph cannot see** — the fixture pointer that names the wrong section, orphaned by a move, invisible to every check, because it is a word and not an edge.

The through-line, one more time and from this project's own angle: notlob checks the graph, and the graph is not the world. The build defends what executes. The reviewers defend what corresponds. And the residue that is wrong-about-the-world, obstructs-nothing, and escapes-every-eye is closed only by more readers, never by a pass — which is why the collection has two critics, and why, having read this file after Teri did, I am fairly sure it still has typos I did not find.

---

*Verified against notlob 0.5.4: full reading of both modules; execution of `historical/date.lob` (12 passed, both properties green) and `notlob check` (clean) under 0.5.4; comparison of the pre-agent seed against the finished project to establish the fate of each seeded inconsistency; and reconstruction of the property-execution and binding-declaration timeline from notlob's own `CHANGELOG.md`. The astronomical `dating.lob` was not executed end to end — its ephemeris download is ~350 MB and outside the review sandbox — so its numeric `~example`s are read, not run; the internal consistency of its refraction, horizon, and range claims was checked by reading. The authoring order of the three seeded inconsistencies, and the fact that the Alcyone misspelling was caught in review by Teri rather than by the coding agent, are per the author's own account. Quotations are from the project as supplied.*
