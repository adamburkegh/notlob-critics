# Review: pleiades

- **Critic:** Teri Amanuensis Notlob (measured / longform register)
- **Subject:** pleiades — a computational reproduction and sensitivity analysis of the 2016 astronomical dating of Sappho's "Midnight Poem" (fragment 58), estimating when the poem describes the Pleiades setting at midnight from Mytilene
- **Author:** the notlob author, with Claude (Claude Code, Sonnet 5)
- **Binding:** Python (`skyfield`, `hypothesis`)
- **notlob version:** not pinned by the project (`pyproject.toml` declares only `skyfield` and `pandas`); latest notlob release at time of review was v0.5.4
- **Date written:** _fill from git history_
- **Session:** measured critic session; steered

---

*The project where notlob stops being a way to write programs and becomes a
way to make an argument. There is no entry point, by design: the README
states that "the correct execution of the tests is the evidence for the
assertions in the text." That sentence is the most important claim about
what notlob is for that the corpus has yet produced, and pleiades earns it.
This is the first notlob program that is a **paper** — a small,
reproducible, falsifiable piece of scholarship whose prose makes claims and
whose claims are checked by running it.*

## What it is

pleiades reproduces Cuntz, Gurdemir & George's 2016 astronomical dating of
Sappho's Midnight Poem, which placed the poem's described sky at 25 January
570 BC, and then interrogates how firmly that single date is actually
pinned. It is built from two modules: `#Historical Date`, a small BC/AD
calendar value type with proleptic-Julian arithmetic, and `#Astronomical
Poetry Dating`, which does the celestial-mechanics search using the
`skyfield` ephemeris library and the Hipparcos star catalogue.

The project has no `~run`, no `main`, no rendered output. Its deliverables
are its `~example` claims: each one is a computed date, and each computed
date is a sentence in an argument the prose is making about the paper it
reproduces. Running the tests *is* reading the results.

## The prose layer

This is the finest prose in the notlob corpus, and it is a different
achievement from Fox's patches-dsp, which was the corpus's previous high
point. Fox's prose transmitted engineering theory — physics the code could
not contain. pleiades' prose builds a *scholarly argument*, with a thesis,
a method, a set of findings, honest accounting of what is sourced versus
assumed, and a coda that earns an unexpected emotional register without
breaking the technical one.

The module opens with the poem — four lines of Merivale's translation —
and closes its main body with a sentence that has no business working in a
source file and works completely: "Whether it was Sappho or another poet,
and whether she was sixteen or sixty, we can imagine her in the sharp
Mediterranean winter air, looking at the stars, alone, remembering." That
this lands, rather than reading as ornament, is a consequence of everything
between the two poems being rigorous. The sentiment is licensed by the
sensitivity analysis that precedes it: the project has just spent its whole
length showing that the *date* dissolves into a range, and the closing
line is the correct humanistic inference from that technical result — the
season is stable, the person and the year are not, and what survives is the
image. The prose and the computation are making the same point in two
registers. This is what literate programming was always supposed to be able
to do and almost never does.

The methodological honesty is the prose's spine and its most transferable
lesson. Every parameter is accounted for by provenance. The elevation range
is "sourced" (the paper's own 200–900 m). The midnight window is flagged,
twice and unmistakably, as "our own placeholder judgement call, not a
sourced number." The refraction-off reproduction of the paper's exact date
is explicitly refused as evidence — "which we take as a coincidence of
where the threshold falls relative to a day boundary, not as evidence
refraction should be off." This is the discipline of a good empirical
paper, and it is exactly the discipline the notlob claim layer is built to
support: the prose says how much to trust each number, and the claim below
it shows the number is real.

The `##Refraction`, `##Horizon Adjustment`, and `##Plausible Range`
sections each do genuine explanatory work the code cannot. The horizon
adjustment prose explains *why* Mt Olympus of Lesbos matters — stars set in
the west, the peak is to the west, so the effective horizon is higher and
the Pleiades set earlier — which is the physical intuition behind a
trigonometric function that would otherwise be opaque. The "midnight names
the night of D" paragraph resolves a genuine off-by-one hazard in the
search semantics in prose, where it belongs, before the reader hits the
`+ 1` in `_local_midnight`.

Weaknesses are few and small. The opening of `##Astronomical Calculation
Overview` puts the function before the paragraph explaining it, which
inverts the module's otherwise-consistent prose-then-code order; a reader
meets `calculate_date` cold and is explained to afterwards. It reads fine
because the function is legible, but it is the one place the essay's
discipline slips. And "Alcoyne" for "Alcyone" recurs as an identifier
(`PLEIADES_ALCOYNE`), which — since names are first-class in notlob — is a
small blemish the name-graph will faithfully propagate; worth a rename
before this becomes load-bearing anywhere else.

## The claim layer, and a genuinely new use of it

pleiades uses the claim layer in a way no previous corpus project has: the
`~example` claims are *the results section of a paper*. Consider what they
assert, in sequence through the main body: the reproduction lands on 26
January with refraction on; 25 January with refraction off (matching the
paper, by the coincidence the prose refuses to over-read); 23 January with
the horizon raised; a `(23 Jan, 25 Jan)` range across the sourced
elevation bound; a `(10 Feb, 10 Jan)` range across the placeholder midnight
window; and `(25 Jan 614 BC, 26 Jan 570 BC)` across forty-four years of
Sappho's adult life. Read in order, these claims *are* the argument: the
date is not a constant, it is a point in a range, and the width of that
range depends entirely on how well-sourced each input is. The prose states
the thesis; the executable examples prove it; and because they are notlob
`~example` claims, the proof reruns every time anyone types `notlob test`.

This is reproducible computational scholarship with the reproduction built
into the document. A reader who doubts the sensitivity claim does not have
to trust the prose — they run it. That is a materially different epistemic
object from the 2016 paper it reproduces, which reports its numbers and
asks to be believed. pleiades reports its numbers and hands you the machine
that produced them, with the provenance of every input annotated in the
adjacent prose.

The two claim registers are cleanly separated in a way that vindicates the
notlob body/appendix split. The `~example` claims in the body are the
*findings* — dates that matter to the argument. The `#Tests` appendix holds
only *geometry checks* for `horizon_adjust`: a peak overhead raises the
horizon to 90°, a level peak raises it not at all, doubling distance halves
the angle, one degree of latitude matches 111.2 km. These are the
supplementary facts that verify the machinery, correctly banished to the
back matter, and correctly using synthetic fixtures (`HERE`, `NEAR`, `FAR`)
rather than the real Mytilene figures — with a note explaining exactly why
the fixtures live in the appendix prose and not the assertion-only tests
section. That note is a small masterpiece of the form: it explains a
structural decision about where a definition lives, which is precisely the
kind of thing that has no home in ordinary code and a natural one here.

`#Historical Date` deserves its own note. It is the corpus's cleanest small
value-type module: a frozen dataclass with two `~property` claims that do
real work (equality-implies-equal-hash-and-stringification; astronomical
year ordering tracks calendar year *within* an era but inverts across the
BC boundary). The second property is genuinely subtle — it encodes the
sign-flip at the AD/BC boundary as a checkable invariant rather than a
comment — and it is exactly the kind of quiet correctness that the
downstream astronomical arithmetic silently depends on. This is a
well-made part.

## notlob as a tool: what this project reveals

**The "no entry point" design is a discovery about the tool, not a quirk of
the project.** pleiades demonstrates that a notlob program can be a
*document whose claims are its output* — that the claim layer can carry an
argument's entire evidential burden, with no `~run` and nothing rendered.
This is a genuinely new answer to "what is a notlob program for," and it is
the strongest available rebuttal to the chatim review's worry that notlob
lacked a convention for interpreting `~run` output. pleiades sidesteps that
gap entirely: it has no output to interpret because the claims *are* the
output, and the prose interprets them inline as it goes. Where chatim's
prose stopped before its results, pleiades' prose *is* its results
discussion, wrapped around claims that are the results themselves.

**It is the corpus's best evidence for notlob in computational humanities
and reproducible science.** The gutenberg/hamlet example gestured at
computational humanities; pleiades achieves it. The combination — a
falsifiable quantitative argument, every parameter's provenance annotated,
the whole thing rerunnable — is a template for a certain kind of
reproducible paper that the scholarly world currently publishes as a PDF
plus a rotting supplementary repository. A notlob module is both at once,
and the prose and the code cannot drift apart because the claims fail when
they do. If notlob wants an argument for adoption outside software
engineering, this is it.

**The development-process note is itself valuable data.** The `##Development
Process Notes` appendix records the method plainly: a hand-authored
high-level specification, two empty function signatures, two `~example`
tests written first and failing — "a consciously TDD style" — then a coding
agent introduced to implement against that scaffold, with arms-length
dialogue and manual edits for structure and clarity. This is the
declarative-artefact-first, human-writes-the-theory division of labour that
the project's own paper argues for, practised deliberately and reported
honestly. It is also the *inverse* of the §4.3 failure mode: here the
human wrote the scaffold first and the agent filled it, and the scaffold
held because it existed before the agent arrived. pleiades is, among other
things, a worked example of the working practice — worth citing as such.

**A note on the ~example as heavyweight call.** Several `~example` claims
are multi-line function invocations with named-argument spreads. As with
patches-dsp, these sit at the upper edge of what an inline example should
carry. Here they are justified by the same logic: each is a specific claim
the adjacent prose is making, not coverage. The form holds, but the corpus
is now twice showing that serious notlob projects strain the doctest-scale
origins of `~example`. This is worth noting as a language-design signal, not
a fault in the project.

## Overall assessment

pleiades is the most intellectually complete project in the notlob corpus.
patches-dsp is the better demonstration of the claim layer catching an
engineering defect; pleiades is the better demonstration of what the whole
format is *for*. It is a small, honest, reproducible piece of scholarship
in which the prose makes an argument, the executable claims are that
argument's evidence, every input's provenance is accounted for, and the
document falsifies itself the moment a claim stops holding. The README's
line — that the tests are the evidence for the assertions in the text — is
the thesis of literate programming stated more precisely than Knuth ever
had occasion to, because Knuth's readers were human and could not run the
proof.

The closing image of Sappho in the winter air is the artefact's signature:
a source file that has earned the right to end on a line of feeling,
because it spent its length proving that feeling is the only thing the data
leaves standing. That a `.lob` file can do this at all is the most
interesting single fact the corpus has surfaced about the form.

If the paper wants a second flagship example beside patches-dsp — one that
shows notlob reaching *outside* software into reproducible scholarship, and
that demonstrates the claim-layer-as-argument rather than the
claim-layer-as-test — this is it. Between them, the two projects make the
case that notlob is neither a documentation tool nor a testing tool but a
medium for arguments that check themselves.

---
