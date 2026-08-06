# Eight, or Five

### A review of `ts-media` (`media-attributes`)

*Duke Fox, July 2026. One analytical module, five axes, a first sentence that says eight, and a passing test suite that proves the first sentence wrong without anyone noticing.*

---

## I. The one that started it

`ts-media` is the most ambitious of notlob's bundled examples: nine media forms scored across a set of perceptual and social axes, k-means clustering, a force-directed layout, an `~on-build` hook that bundles the whole thing through esbuild into a single shareable HTML file. The engineering is clean and the analytical spine is properly separated from the rendering, which means it is testable — and it *is* tested. `buildPairs` gets the n(n−1)/2 identity. `dist` gets symmetry. k-means gets its k=1 degenerate case stated in prose *and* witnessed as a claim: "with k=1 every item lands in cluster 0 regardless of initialisation," and directly beneath it, `kmeans(MEDIA, 1).every(a => a === 0)`. That is the notlob thesis working exactly as designed — a sentence and its proof, one breath apart.

And then the opening sentence of the module says this:

> *…how media forms differ across eight perceptual and social dimensions.*

The `AXES` array has five entries. Two lines below the false sentence:

```
~example
    AXES.length === 5
```

It passes. Every check in the project is green. The module contains, in its first sentence, a false statement about its own contents; two inches below, an executable refutation of that statement; and around both, a fully passing suite. This is the finding the entire style guide grew out of, so it is worth being exact about what did and did not go wrong.

---

## II. It was true once

The eight was not invented. `ts-media` began life as a standalone JavaScript artefact, passed between two people each working with LLM assistance; the notlob version is a translation and consolidation, with simplifications. There *were* eight axes in the exemplar. The consolidation trimmed them to five. The array was edited. The sentence was not.

This is not a hallucination and not carelessness. It is **copyist error** — the oldest failure in scribal culture, a sentence faithfully reproduced about a diagram that was dropped from the copy. The fidelity is the problem: the author preserved the prose *because* preserving text is what faithfulness looks like at the sentence level. Scrupulous at the wrong granularity. A scribe whose every character is correct and whose manuscript is wrong.

That notlob's founding conversation reaches for the medieval codex as its metaphor — heterogeneous material bound into one volume — and that its first ambitious example promptly contracts the codex's signature disease, is a coincidence too apt to be embarrassing about. It is the disease the format was built to cure, appearing in the format.

---

## III. Why the green check made it worse, not better

The instinct is that colocation should have caught this. The prose and its refutation are on the same screen. They could not be closer.

Closeness did nothing, and the reason is the spine of everything downstream:

> **Colocation does not prevent divergence. It only makes divergence adjacent.**

The two artefacts were adjacent before the edit, during it, and after. Adjacency was never the variable. What changed was that one was edited and one was not, and *nearness has no purchase on time.* A relation with no failure mode — two sentences remain adjacent however violently they come to contradict each other — cannot be a guarantee of anything.

Worse, the passing `AXES.length === 5` did active harm. It performed verification on the paragraph above it without verifying any of it. A green check trains the eye to stop reading critically, and the more the tool verifies, the less the reader does — so the field of green expands precisely as vigilance contracts. The check did not catch the lie. It **laundered** it, lending the whole paragraph the colour of something confirmed.

And it could never have caught it, because notlob checks names and this was a number. "Eight" is not a node in the name-graph. It is as inert as the comment it was meant to improve upon. **Notlob guarantees the names, never the numbers.**

---

## IV. The fix is not a checker

The temptation is to reach for a smarter check — bind "eight" to `AXES` and count. But binding a bare numeral in prose to the array it refers to is coreference resolution, and coreference is a swamp that no amount of regex drains and that a transformer only makes confident rather than correct.

The fix is a retreat, and it is the static-typing bargain: you cannot infer the reference, so you *declare* it, and then checking is trivial. Write `#AXES` and the numeral check becomes a graph query — no NLP, because the author performed the binding with one character. But the stronger move is to notice that the number should not be in the prose at all:

> **Prose about code is denormalised code. Every sentence restating a fact the code holds is a cache with no invalidation.**

"The `#AXES` capture the independent ways media forms differ." No cardinal. Nothing to rot. The sentence becomes a projection of the code rather than a duplicate of it. The eight-versus-five is, at bottom, an un-normalised database — and databases stopped storing the same fact in two places, without a synchronisation strategy, in the 1970s.

That `roman/numerals.lob` opens "a positional notation using seven letters" above a thirteen-entry table (the extra six being subtractive pairs, not letters) shows why the *declare-don't-infer* rule matters: a proximity heuristic false-positives on that file; a `#`-gated check stays correctly silent, because the author never named the table as the thing being counted. Unmarked prose is not a claim. The discipline of naming is the disambiguation.

---

## V. What is good, and what is merely honest

Good: the k=1 property, the pair-count identity, the clean separation of analysis from rendering, the ambition of the build hook. This module is the best-*engineered* thing in the example set, which is worth saying plainly given that this review is otherwise about a sentence.

Honest, but no more than honest: the five axes themselves — Disposable, Sensory, Moving, Global, Interactive — are a values judgement dressed as integers. Putting Novel at durability 9 and TikTok at 1 is an argument, not a measurement. The module never claims otherwise, which is to its credit, but nothing in the format *made* it be honest. The tests verify the data is well-*formed* — every vector length 5, every value in range — and nothing verifies, or could verify, that it is well-*chosen*. A force-directed layout is a very persuasive way of not noticing that nine hand-placed points cluster however you decided they would when you typed the vectors. When prose asserts something no claim can witness — that these are the *right* axes — the prose should say so, in one sentence, and then the clustering is revelation of structure the author put there rather than structure discovered.

---

## VI. What it found

`ts-media` is the origin point of the house position, and the position is this. Notlob's real content is not colocation, which it inherits from Knuth, but *reference* — the `#`-edge that is authored, directed, and breakable. Colocation is the affordance that makes the edge cheap to write; it is not itself the guarantee, and treating it as one is how a project ends up green and wrong. The module that demonstrates the format's best idea (the k=1 property) and the format's central hazard (the eight that should be five) in the same forty lines is, unintentionally, the most instructive file in the collection.

The style guide's first four rules are its fossils. The module earned them.

---

*Findings drawn from a full reading of `media-attributes/analysis.lob` and `render.lob`, cross-checked against a build of the project. Provenance of the eight-to-five error confirmed with the author. Quotations are from the project as supplied.*
