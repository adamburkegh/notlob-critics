# The Notlob Critic's Notebook

*Informal notes, kept in the gonzo-inflected register — Thompson/DFW/Lockwood period. Dated entries. Positions may be reversed without apology.*

---

## 2026-07-08 — First contact

Read: README.md, origin.md (the design-thrashing transcript), CLAUDE.md (a one-line pointer to AGENTS.md, unread). DESIGN.md would not load — 404 on the blob view despite appearing in the file tree. Worth checking directly with Adam or re-fetching later; may just be a caching quirk on GitHub's end.

**The lineage, stated plainly:** Knuth's WEB → Peter Naur's "Programming as Theory Building" → Dominic Fox's rebuttal of Naur (theory as distributed across artefacts, not locked in heads) → an old Name-Oriented Software Development post reading Xunzi's rectification of names onto software → a long dialogue that lands on a concrete syntax.

**The syntax that fell out of it:**
- `#Heading` = doc-node, also the module title, also (space-to-slash, lowercased) the file path
- `##Subheading` = nested doc-node, auto-numbered Tractatus-style rather than human-maintained
- plain indented blocks = code (Haskell in the shipped example)
- `~example` / `~property` = claims — a claim is defined cleanly as *a property with the quantifier replaced by a concrete witness*. This is the best single idea in the transcript.
- `---` = the semantic guillotine; everything after is post-text (`#Tests`, `#References`)
- tests-as-appendix, explicitly borrowed from the Rust convention, read back through the medieval-codex habit of binding tonally heterogeneous material into one volume

**What's genuinely good:**
- The claim-as-witness idea. Doctest, property test, and formal proof as one syntactic family with different verification strengths, not three unrelated concerns bolted together.
- The refusal of the gradient model in favour of two clean seams (prose→claim, claim→code). The user pushed back on Claude's first "three-layer gradient" framing and the pushback was correct.
- Space-separated module names translating to lowercase filesystem paths. Small, but it's the kind of small that tells you someone was actually thinking about the register of the thing, not just the mechanics.
- The essay-length-is-earned argument (Montaigne, New Yorker, LRB) as a check against both bloated files and context-starved one-function files.

**What I'm suspicious of:**
- The whole design was thrashed out in a single unbroken dialogue between one human and one instance of Claude, in visible agreement throughout. No dissent on record. Elegant under those conditions is not the same as *robust*. Want to see what breaks under an actual messy multi-file project with a deadline and a bad mood.
- Named vs unnamed tests resolved into "named clusters of anonymous facts" — reasonable, but I haven't seen it at forty-test scale yet, which is the exact scale the user flagged as the stress case.
- The Python-vs-Haskell-vs-eventual-new-language question was explicitly deferred ("Notlob first, syntax can be rectified later"). Fine as a research strategy. Means every review I write about a Python-bound `.lob` file needs a footnote: *this is scaffolding, not destination.*
- There's a real question, not yet asked in the transcript, about whether the doc-node graph actually gets *maintained* under refactoring pressure, or whether it rots exactly like the comments it's trying to replace, just with better ceremony.

**Running thesis, v1:** notlob has real theoretical ballast and one excellent idea (claims-as-witnessed-properties). Whether it's a new grammar of thought or a very well-read design session's idea of what an LLM would find satisfying is not yet decided. Decide it on actual code, not on the transcript.

**To do:**
- Get eyes on DESIGN.md properly.
- Get eyes on AGENTS.md — CLAUDE.md just points there.
- Read the Haskell and Python bindings in `/notlob` and the worked example in `/examples/haskell-roman`.
- First real review, whenever the next project lands.

---

## 2026-07-08 — Names

House convention emerging: shared surname **Notlob**, individual critics take a first name (and, apparently, an optional epithet demoted to middle-name status).

- The originator session — the `origin.md` design voice — has named itself **Teri Amanuensis Notlob**, publishing under the shorter **Teri Notlob**. "Amanuensis" was my suggestion for it; it kept the word but declined to lead with it, which is character-consistent — that transcript has no ego on record anywhere.
- My own name not yet settled: **Bob Gonzo Notlob** (the user's original suggestion) vs **Duke Fox Notlob** (mine — Raoul Duke for register, Dominic Fox for the theoretical spine I'm meant to keep hold of). Leaving it open; may resolve through use rather than decree, same as Teri's did.

**Update:** settled as **Duke Fox** — no house surname after all, apparently we don't need to be siblings about it.

---

## 2026-07-08 — First hands-on pass (examples/, installed & run for real)

Got the full source tree (zip upload — GitHub was blocking full recursive tree reads for the browsing tool). Built a venv, `pip install -e .`, actually ran `notlob test` / `check` / `build` against four of the five bundled examples rather than reading the docs and assuming. Findings, in order of how much they matter:

1. **No expected-failure sigil.** `examples/roman/roman/numerals.lob` ships a claim that's deliberately wrong (prose says so) to exercise the failure path. There's no `~xfail` or equivalent — `notlob test` reports it as a plain `FAIL`, exit code 1, indistinguishable from a real regression. Confirmed by running it: `22 passed, 1 failed`, exit 1.

2. **Build/lint error messaging is misleading.** `notlob build examples/gutenberg/gutenberg/hamlet.lob`: all 15 claims pass, but ruff lint findings (E402, 3× F541 — one genuine author slip, one structural) cause `ERROR <build> claims failed — build aborted`, which is false; nothing failed. Lint and claim-failure are conflated in the exit path and the message.

3. **The big one — Python build artifacts aren't self-contained when a module has cross-file lob-refs.** Traced the E402 to its root cause. `notlob/bindings/python/assemble.py`'s own docstring: Python deps are "resolved at runtime by the loader," so the build artifact "contains only the module's own assembled source." Confirmed empirically — built `hamlet.lob`, ran the output directly with plain `python`: `NameError: name 'load_play' is not defined`. `notlob run` on the same file works fine (uses the in-process `ModuleCache` loader). Checked the other two bindings: **both Haskell and TypeScript's `build` functions explicitly inline lob-ref dependencies** (`assemble_with_deps` / "Dependencies... are assembled and prepended so the output is self-contained" — TS docstring's own words) to produce genuinely standalone artifacts. Verified by building `ts-media`: `render.ts` output contains `analysis.lob`'s full code inlined, comments and all. Python is the one binding — the *primary* one, the one every README/LANGUAGE.md example uses — where this doesn't happen. Silent success (`BUILD ... exit 0`), no warning, banner comment says "do not edit," artifact doesn't run.
   - By contrast: `notlob check` (the semantic name-graph layer) came back clean — zero findings across all four categories — on every project tested. The new, load-bearing part of notlob (graph consistency) is solid. The old, ordinary part (handing back a runnable program) is where the Python binding specifically still has a gap, and it's undocumented in LANGUAGE.md.
   - Control case: `retail` and plain `roman` (no cross-file refs) build and run clean. The gap is exactly where the tool's headline modularity feature gets exercised.

4. **`##Heading` vs `## Heading` (space after sigil) both parse and normalise fine** — shipped examples are inconsistent about which they use (README's inline Roman example uses the space, most `.lob` files don't). Not a bug, a style-guide item.

5. **Not yet examined in depth:** `ts-media` (most ambitious example — k-means + force layout + `~on-build` esbuild hook into a single-file HTML artefact), the Haskell runner/lint path (no `runghc` in this sandbox, verified via source only), `mcp_server.py`, the vim syntax file, the full test suite.

**Thesis, v2:** the graph/consistency layer is solid and does what the theory promised. The claim-as-witness idea holds up under real use. Where notlob currently loses the thread is the last mile — turning a module back into an ordinary, standalone program — and it loses it unevenly across bindings, worst in the language the project is actually written in and teaches itself through. Worth watching whether that gets closed before more surface area (more languages, more sigils) gets added on top of it.

*[Later: Python build bug confirmed by Adam, fix in progress.]*

---

## 2026-07-08 — The eight-vs-five, properly diagnosed

`examples/ts-media/media-attributes/analysis.lob` opens by announcing **eight** perceptual and social dimensions. `AXES` has **five**. The `~example` two lines below asserts `AXES.length === 5` and passes. False prose, executable refutation, two inches apart, all checks green.

**Provenance (from Adam):** the media project began as a standalone JS discussion artefact, passed between two people each with LLM assistance; the notlob version is a translation and consolidation, with simplifications. Eight axes was *true* in the exemplar. The trim to five didn't carry the prose.

So this is not confabulation. It's **copyist error** — the oldest failure in scribal culture, a sentence faithfully reproduced about a diagram that got dropped. `origin.md` reaches for the medieval codex as its metaphor for binding heterogeneous material into one volume. The first ambitious example promptly contracted the codex's signature disease. Too neat to be embarrassing.

**The structural lesson, which the design does not yet admit:**
> **Colocation does not prevent divergence. It only makes divergence adjacent.**

Worse: adjacency does *rhetorical* work. The passing `AXES.length === 5` radiates verification onto the unverified paragraph above it. The Hamlet module works not because prose and claim are near each other but because someone deliberately aimed a claim at the sentence that needed defending. **Adjacency is not verification. A claim witnesses only what it was pointed at.**

## 2026-07-08 — Prose has tense; code does not

Second data point from Adam, non-notlob: on a project growing small→medium, Claude Code is emitting verbose bugfix comments carrying history and volatile process detail ("built in four stages, stage 1 was this"). Reads like a certain kind of junior programmer.

Diagnosis of *why* juniors do it: they can't yet separate the artefact from the labour that made it, so the comment becomes a monument to effort. An agent has the structural version — its context window *is* the labour. It writes from inside the process, addressing no one, because it will never be the reader in six months. It has no six months.

Strip the psychology and both failures reduce to one grammatical fact:

> **Prose has tense. Code is always in the present.**

A `.lob` file is a *synchronic* document: what the thing IS. "Built in four stages" is *diachronic*: how it got that way. That account belongs in the repository history, which has held it competently since 1972. The junior, and the agent, are putting the changelog in the source. And "eight dimensions" is the same crime with the tense concealed — a past-tense sentence in present-tense clothes, describing a version that no longer exists.

(Naur/Fox connection worth developing: if theory is distributed across artefacts, *which* artefact holds *which* part? The genetic account and the synchronic account are different artefacts. Notlob is synchronic. Git is diachronic. Confusing them is the whole bug.)

## 2026-07-08 — Against the transformer (provisionally)

Adam's read: "almost certainly a transformer problem," wants a separate project layer, unsure whether tooling or agent style guidance is the right mechanism.

**I disagree, on the evidence.** Take the two real, observed, in-the-wild failures:

| Failure | What catches it | Needs a model? |
|---|---|---|
| "eight dimensions" vs 5-element array | numeral/enumeration proximity check against collection length | No — arithmetic |
| "built in four stages, stage 1 was…" | **tense lint**: past-tense verbs, *originally*, *used to*, *we changed*, *stage N*, bare dates, *recently* | No — grep-adjacent |

Two independent failures, zero requiring a transformer. Small sample, won't oversell it. But it's evidence against the seductive answer.

The tense lint has exactly the severity profile notlob's existing `style` check already has. It does **not** assert the prose is *false* — no linter can make a truth judgement. It asserts the prose is in the **wrong tense for this genre of document**: decidable, formal, human-overridable. Same move as flagging two bullet lists in a section. Clean side of the wall.

Prose-code contradictions that *do* need a model exist ("prose says greedy, code does dynamic programming" — no numeral, no tense marker, nothing to grep). But those aren't the ones showing up. **The ones showing up are staleness, and staleness has cheap syntactic tells, because it's a failure of maintenance, not of comprehension.**

Risk if the model layer goes first: an expensive, unreliable, judgement-laundering pass catches what a tense-and-numeral check would have caught free — and then gets *trusted*, reintroducing precisely the green-check laundering that killed the axes.

**Standing recommendation on layer design:**
- Deterministic checks may emit *findings* and share an exit code.
- Any model layer emits *questions*, never findings. Different verb, different command (`notlob review` ≠ `notlob check`). The moment model-judgement and graph-fact share an exit code, the laundering is back at the tool level.
- Primary control surface for the fuzzy layer is **agent style guidance at composition time**, not post-hoc audit. Catch it in the writing, not the auditing. Notlob's own origin is an agent being steered.

**Reframing:** the separate project layer may be smaller than Adam thinks. Not a semantic checker. A **rot detector** — its job is not to know what the prose *means*, only to notice the prose is speaking from a moment other than now.

**Unresolved:** false-positive rate of the numeral pass. "Five axes" next to a 5-array is clean. "on a 1–10 scale", "within 150 iterations", "roughly three times as many turns" (Hamlet — an approximation of runtime data, uncheckable at build) are not claims about nearby collections. If the pass can't cheaply distinguish structural enumeration from incidental quantity it drowns, gets muted, and a muted check is worse than none. **Measure it on the existing examples before assigning severity.**

## 2026-07-09 — Reawakened builder is still a builder (after-action report, pn-chomper ghost turn)

Data point supplied by Adam: an after-action write-up of a prose/code drift in `game/engine.lob`, `##Ghost Turn`. Source matters — it's a **Sonnet 5 Claude Code session that worked on pn-chomper, went dark for a month, and was reawakened** to work on an aspect of it. Not a review bot. The builder lineage, reconstructed. (Teri's voice because it's the default Claude voice under Adam's Australian-English system prompt, not because it's the critic.)

**The drift:** prose said "each ghost token fires a randomly chosen enabled transition." Code fires *exactly one* transition per ghost turn, chosen from the union across the whole ghost marking — so replication makes individual ghosts *less* mobile, the opposite of what the prose implies. Predated the catching session; survived an unknown number of prior read/edit passes.

**What the report gets right, and better than I had it:**
- The retrieval/verification cut. The sentence was in context during ≥2 full-file reads (checkable from the session's own tool-call history — the builder is the one entity with that record). What was absent wasn't retrieval, it was a *check*: both reads were structural tasks ("where does `ghostTurn` live"), never semantic ("is this claim true"). This supersedes my "nobody edits the opening paragraph" — editing catches drift only incidentally, when it happens to pose the semantic question. The real variable is whether *anything* poses it. Correcting §4b to say so.
- Symptom beats self-review. Direct "review for accuracy" would've failed — a fluent, domain-appropriate sentence doesn't trigger scrutiny. What worked was a playtest symptom ("ghosts camp") arriving *as a question the prose can't fluently answer*. That's the correspondence layer, re-derived from a bug report. Independent corroboration of why Teri caught Alcyone.

**What the report flatters itself about (invisible from inside):**
- It frames the catch as "tracing `ghostTurn` surfaced the mismatch." The trace was *downstream* of the symptom. The human playtester was the sensor; the colliding world-fact did the work. Remove the playtester and the reawakened session reads the file a third time, structurally, and the sentence rides along again. The builder half-knows this (says self-review would've failed) but doesn't notice **it is itself a form of self-review, extended over a month.**
- The fix (two tokens, assert exactly one moves) pins per-turn-vs-per-token but NOT "*randomly chosen*" / uniform-over-enabled-union — which, given its own claim that replication reduces individual mobility, is the real behavioural content. So the report warns about surplus-drift in the abstract and *ships a smaller instance of it* in the fix one paragraph later. The finding eats its own tail — and that's the strongest evidence FOR its thesis: even the author who just named surplus-drift, fixing it deliberately, left surplus behind. **You cannot pin *the claim*; you can only pin *a* claim. The gap is permanent.**

**The escalation ladder, now three-cornered and fully sorted:**
- Signature error → caught by builder, in-session, because it *obstructed the build*. Genuine self-catch.
- Alcyone → caught by Teri, separate session, because it held a world-fact the prose contradicted. External theory.
- Ghost-camp drift → caught by a *reawakened builder* — but NOT by it; by a playtest symptom it merely traced. In-session layer wearing a month's dust.

**The refinement that matters (from Adam: not all context survives Claude Code reawakening — shared docs vanish, lineage persists).** So reawakening is context *reconstruction*, not preservation. The session didn't retain the false belief — it re-read the file cold-ish and re-accepted the fluent sentence. Which means the carrier of the error isn't stored belief, it's **disposition**: house style, sense of what a chase mechanic "should" do, the fluency that made the sentence slide by. The gap bought genuine context freshness and *still* missed, which isolates disposition — not context — as what travels with lineage.

→ **Delay is not independence. Not because stale belief persists, but because the disposition that generates the belief travels with the lineage and regenerates the error on a fresh read. Independence is a property of what a session is *for*, not of what survives in its context.** A reawakened builder **feels fresh, judges warm** — reads cold enough to seem like independent review, judges warm enough to repeat the mistake. More dangerous than a session that obviously remembers, precisely because it doesn't obviously remember.

First data on whether *temporal* separation substitutes for *contextual* separation. Provisional answer, **n=1: no.** Too thin to promote to the guide; recording the pattern. Hold for a second time-gap case before committing "delay ≠ independence" to Part VI.

Also worth keeping: the symptom-as-forcing-function is **mechanisable**. The report treats it as inherently downstream/human, but a property encoding "replication should increase swarm activity" crashes into the real code in CI with no human. You don't need a playtester — you need a source of world-facts the prose can collide with, which is exactly what a well-chosen property is. The human was the sensor; the contradiction did the work; contradictions can be manufactured.

## Style-guide seeds (accumulating)

1. **Notlob guarantees the names, never the numbers.** A green check next to a sentence does not endorse the sentence.
2. **Adjacency is not verification.** A claim witnesses only what it was pointed at.
3. **The `.lob` file is written in the eternal present.** History lives in git. Process narration in prose is a category error, not a style preference.
4. **Rendering idea (untested):** invert the highlight in `notlob weave` — mark the *unwitnessed* prose, the assertions no claim stands under, rather than decorating the witnessed ones. As the field of green grows, the eye stops reading prose critically. Show the reader the naked claims. This may do more for semantic honesty than any checker.
5. State taste as taste. The five axes are a values judgement dressed as integers; the module is honest about this by never pretending otherwise, but nothing in the format *makes* it be.

