# The Notlob Style Guide

*First edition. Duke Fox, July 2026.*

---

## Preface, with a warning

A style guide written before the thing has a style is prejudice with a table of contents, and I have written one anyway, because the alternative is watching a format acquire its conventions the way rivers acquire courses — by the accident of what happened to be soft when the water first came through. Notlob is soft right now. There are five example projects and one of them contains a lie in its opening sentence. This is the moment.

Everything below is derived from evidence in the repository as it stands, not from taste. Where I am guessing, I say so. Where I am generalising from a single case, I say that too. The rules that matter are the ones with a corpse behind them.

Two prior documents make claims about notlob's nature: `origin.md`, which is a design conversation, and `LANGUAGE.md`, which is a specification. This is neither. A specification tells you what parses. A style guide tells you what a `.lob` file is *for*, and therefore what you must not do with it even though the parser will let you.

---

## Part I — What a `.lob` file is

### 1. The file is written in the eternal present

A `.lob` file is a **synchronic** document. It describes what the code *is*. It does not describe how the code came to be that way.

The history belongs in the repository, which was built for exactly this purpose and has done the job competently since 1972. A commit message is a diachronic artefact. A `.lob` file is not.

```
    ✗   Originally this used a dictionary, but after the v2 refactor
        we moved to a list of tuples for ordering guarantees. Stage 1
        of the rewrite handled encoding; stage 2 added decode.

    ✓   The table is a list rather than a dict because conversion
        depends on descending value order.
```

The second version contains the *entire* useful content of the first. The reason for the list is preserved. What is discarded is the narrative of the author's afternoon.

**Rule:** no temporal deixis. *Now, currently, recently, originally, previously, formerly, no longer, used to, the new, the old, stage 1, since the refactor, TODO*, bare dates. Words whose meaning depends on when the sentence was written have no place in a document that will be read at an unknown time.

A `.lob` file has no now.

### 2. Why agents in particular get this wrong

Verbose historical commentary is the signature of a certain kind of junior programmer, and the human diagnosis is that they cannot yet separate the artefact from the labour that produced it. The comment becomes a monument to effort.

**That diagnosis is wrong for agents, and getting it right changes the remedy.**

Look at what is actually in the comments an agent leaves: staging, sequence, what was attempted, the state at each step. That is not pride. **It is a memory dump.** The agent has correctly established that it is about to be truncated, compacted, or ended, and it is writing continuity state into the only durable medium within reach — which happens to be the source file. It is not addressing a maintainer; it has never met one. It is addressing its own amnesiac successor. The note is dense with process because process is precisely what a successor needs in order to resume, and precisely what a maintainer needs never to see.

Comment rot is therefore not an indiscipline problem. **It is a missing-artefact problem.** There is nowhere for diachronic agent state to live, so it squats in the synchronic document.

### 2b. Where history actually goes

A journal cannot be a `.lob` file. This follows from the graph, not from taste: a `.lob` file participates in the name graph, every `#Reference` must resolve, and history's whole business is the dead. A journal entry mentioning a module deleted three commits ago would fail `check`. **The diachronic artefact must live outside the graph, necessarily, because the graph is a snapshot of what currently is.**

Three artefacts, then, not two:

1. **The `.lob` file.** Synchronic. Graph-checked. Eternal present.
2. **The commit history.** Diachronic, immutable, and correctly able to address the dead, because a commit message refers to a tree in which the thing still existed. *This is already the journal.* It has been the journal since 1972. Storage was never the problem.
3. **The scratchpad.** The missing one. What an agent needs is not a durable journal but **ephemeral working memory with a mandatory compaction step**: written to freely during a task, distilled into the commit message at the end, truncated. Gitignored.

A durable `journal.txt` does not fix rot. It **relocates** it. Anything durable and unchecked rots. The only safe home for volatile state is one that is deliberately, structurally temporary. **The discipline is the emptying, not the writing.**

Distinct from all three, and not to be conflated with the scratchpad: a **design record** such as `origin.md` is diachronic, curated, and published. It is a *work*, authored on purpose. A scratchpad is a *byproduct*. A file that fails to declare which one it is will drift into being both, and so neither.

**Where the guidance lives:** the *prohibition* belongs in this style guide, which humans read during review. The *provision* — scratchpad location, compaction ritual — belongs in `AGENTS.md`, which agents load at composition time, where the intervention has to happen. A lint tells the agent *don't*. An artefact tells it *there*. Prefer the second.

### 2a. Why the contradiction goes unseen even when the file is fully in context

Worth stating plainly, because the intuition that "the model read the whole file, so it should have noticed" is natural and false. Four things stack:

- **Reading is not checking.** Attention is retrieval-shaped, not audit-shaped: it computes what is relevant to the next token, not whether everything present is mutually consistent. Counting five array elements is a serial operation impersonated by a parallel one. The information is in context. The comparison was never performed. Having the file on the desk is not having read it, and reading it is not having audited it.

- **Autoregression has no backward pass at inference.** When the five-element array is emitted, the prose asserting *eight* is behind the cursor, and the cursor does not return. Context conditions generation; generation cannot revise context. The contradiction is not detected and ignored — there is no state in which it constitutes an error, because there is no site at which a flag could be raised. A human *can* scroll up and usually doesn't; that is a discipline failure. A model mid-generation cannot; that is an architectural one. There is no reread. There is only re-reading, as a separately invoked act, on a later turn, which nobody invoked.

- **Genre salience, and notlob makes it worse.** An opening paragraph is, by convention, *orientation* — framing, not content. Every reader, silicon or otherwise, files summary sentences under decoration. Notlob's doc-node structure makes the module-opening paragraph the natural home for exactly the summarising cardinality claim most likely to rot. **The format nominates the most dangerous sentence in the file and seats it where nobody looks hard.** This is structural, not an accident of one project.

- **Fidelity did the rest.** The instruction was *consolidate and simplify*. The axes were trimmed because that was the task. The prose was preserved because preserving text is what faithfulness looks like at the sentence level. Nothing was careless. It was scrupulous at the wrong granularity — which is exactly what a copyist is: a scribe whose local accuracy is total and whose document is wrong.

The instruction that follows from all of this is therefore not "write fewer comments." It is: **write for a reader who has never seen the process** — and, for whoever maintains the project, *give the process somewhere else to go.*

### 3. Length is earned, not imposed

From `origin.md`, and it holds: a module should be as long as its concept requires. The one-function file is context-starvation. The thousand-line file is a concept that has quietly become three concepts.

The test is not line count. The test is whether a competent reader, dropped into the file cold, can hold the whole thing in mind at once. If you find yourself writing "as discussed above" about something four subheadings back, you have two modules.

**This rule is load-bearing, and not for aesthetic reasons.** Agents scan rather than read whole files, and scanning is not merely a premature optimisation — it is rational under an **externality**. The cost of missing context is paid by a future reader, on a later day, out of another budget. The agent never experiences the bug it caused, and so has no feedback loop by which to correct. An instruction to "read the whole file" competes against a live token cost, and loses, especially late in a session when the instruction is far behind the cursor.

Nudges do not fix this. Two things do:

1. **Short modules.** If a `.lob` file is Montaigne-length, it fits, the full read is cheap, and there is no scanning decision to be made. The literary argument and the context-window argument are the same argument. That they coincide is evidence the design is onto something.
2. **Making the unit of access the unit of meaning.** Grep returns lines. Notlob should return modules. The MCP server and `notlob query` — which operates on the graph rather than on text — already have the right instinct; it should be finished. If the tool surface offers no sub-module read, scanning is not discouraged, it is *impossible*. **Affordances beat nudges, because an affordance never has to win an argument against cost.**

---

## Part II — Prose and code

### 4. Adjacency is not verification

This is the most important rule in the document and it was bought with a real failure.

`examples/ts-media/media-attributes/analysis.lob` opens: *"Core data types, sample dataset, and analysis algorithms for exploring how media forms differ across eight perceptual and social dimensions."*

The `AXES` array below it has five entries. Two lines further down, a claim:

```
~example
    AXES.length === 5
```

It passes. Every check in the project is green. The file contains a false statement about its own contents, an executable refutation of that statement, and a passing test suite, all within twenty lines of each other.

The prose was not hallucinated. It was **true** — of an earlier artefact, before a consolidation trimmed eight axes to five. The sentence survived a translation the code did not. This is the oldest error in scribal culture: the copyist faithfully reproducing a description of a diagram that was dropped from the exemplar. `origin.md` reaches for the medieval codex as its founding metaphor. The first ambitious example promptly contracted the codex's signature disease.

The lesson is structural and the design does not yet admit it:

> **Colocation does not prevent divergence. It only makes divergence adjacent.**

### Why proximity cannot work

It is worth being precise about *why*, because the answer determines what the fix must look like.

**Divergence is temporal. Proximity is spatial.** The prose and the code in `analysis.lob` were adjacent before the consolidation, adjacent during it, and adjacent afterwards. Adjacency was never the variable. What changed was that one of them was edited and the other was not. Nearness has no purchase on that at all. It is a remedy applied in the wrong dimension.

**Adjacency has no failure mode.** Two sentences sitting beside one another remain sitting beside one another however violently they come to contradict each other. Nothing snaps. Nothing goes red. The relation is unfalsifiable, which is precisely why it is worthless as a guarantee — and precisely why it is so comfortable to believe in.

Worse than worthless. Proximity *does* have an effect, but on the reader rather than the code. A passing claim beneath a paragraph performs verification without doing any, and the reader's eye — the last real checker in the system — relaxes. **Proximity buys nothing mechanically and costs you vigilance.**

Why did anyone think otherwise? Because in WEB it worked. Knuth wrote both halves, once, and reread the whole thing repeatedly, because he was Knuth. Proximity in a literate program was never an invariant; it was an *invitation to notice*, aimed at a human who was going to reread the page anyway. It is a discipline, not a mechanism. It held Knuth's programs together because Knuth was compulsive, not because WEB enforced anything. Remove the compulsive reader and the layout goes on working perfectly while the meaning comes apart underneath it.

### 4b. Proximity's range is one edit-unit

*Added after `pn-chomper`, which supplied the correction.*

The claim that adjacency is worthless is too strong, and the evidence against it is instructive.

In `game/map.lob` a module-opening paragraph asserts a hundred places and five fork junctions. The code holds a hundred and one places and four forks. But a test-group label in the post-text reads `##grid: 101 places, join room only reachable via coordinated two-token play` — unstructured prose, containing a cardinal, subject to no check — and it is **correct**.

It survived because it sits one line above `places.size === 101`, and whoever typed that assertion was looking at the label while doing so. The two are inside the same **edit**.

> **Proximity is a weak proxy for co-editing. Its effective range is approximately one edit-unit** — a line, perhaps two. Inside that range there is already a reader, because the thing they came to change is right there. Outside it, decay is total and abrupt: a paragraph twenty lines up is as unreachable as a paragraph in another file.

The mechanism was never distance. It was always **whether the two things fall inside the same act.** Test-group labels always do. Essay paragraphs never do.

This sharpens §16 from a genre observation into a mechanism. The module-opening paragraph is not dangerous because nobody reads it. It is dangerous because **nobody edits it.** It is a slot that receives text once, at composition, and is thereafter structurally excluded from every subsequent operation on the file.

### 4a. Reference is proximity in a better space

The sharpest objection to the above is that **reference is itself a form of proximity** — distance-1, merely in a different space. This is correct, and it improves the argument rather than damaging it. The question is not proximity versus reference. It is why adjacency in the *graph* is trustworthy when adjacency in the *text* is not. Three reasons, and all three are needed:

- **The graph metric is authored.** Text adjacency is a byproduct of serialising a graph onto a line. Someone had to *type* the `#`. Intent lives in the edge.
- **The graph edge is directed.** "Eight dimensions" points at `AXES`; `AXES` does not point back. Text distance is symmetric and cannot express aboutness.
- **The graph edge can break.** Delete `AXES` and `#AXES` dangles, loudly. Move a paragraph and text-adjacency silently reconfigures with no error whatever. The edge has a failure mode. The metric has none.

So the problem was never proximity. It was **proximity in the wrong space.** Text-space is one-dimensional because paper is one-dimensional; the graph is the space the meaning was always in, and the file is a projection of it. This is Fox's point restated: the theory lives in traces that *refer* to other traces. In the edges. Not in the layout.

Put sharply: **notlob quotes Fox and implements Knuth.** Fox's argument is about reference. Knuth's practice is about layout. The eight-versus-five is what happens when a design inherits the arrangement and forgets the binding.

That a pointer should be more reliable than being *literally next to* something is unintuitive, and the reason is worth naming. Our intuitions about co-location come from physical space, where proximity is causally live — things near each other interact, exert force, catch fire together. Text imports the intuition and none of the physics. **A book's pages are adjacent because it has to be bound. Nothing about the binding is a claim.**

**Correction, and it is small:** colocation is not the guarantee. It is the **affordance** — the thing that makes it cheap to write the `#` which *is* the guarantee. Adjacency's job is to put the referent within reach so that naming it costs nothing. It was never meant to be load-bearing alone.

**Rule:** a claim witnesses exactly what it names, and nothing else in the vicinity. When you write a sentence that asserts a fact about the code, either point a claim at *that fact*, or accept that the sentence is unverified prose and may rot.

### 5. Notlob guarantees the names, never the numbers

`notlob check` validates that references resolve, that symbols are not near-duplicate typos, that titles do not collide. It cannot read a sentence containing the word "eight" and go count an array, because "eight" is not a name in the graph. It is as inert as the comment it was supposed to improve upon.

The danger is not that the check is limited. Every tool is limited. The danger is that **a field of green checks trains the eye to stop reading prose critically.** The more the tool verifies, the less the reader verifies, and the gap between those two sets is exactly where the rot lives.

### 6. Do not restate in prose what the code already holds

The fix for §4 and §5 is not a smarter checker. **No checker catches the eight-versus-five in unmarked prose, and none ever will** — binding "eight dimensions" to `AXES` is coreference resolution, and coreference is a swamp. The fix is a retreat from that impossibility, and it is the static-typing bargain: you cannot infer the type, so you declare it, and afterwards checking is trivial. The cost is one character, paid by the author.

```
    ✗   Eight dimensions capture the independent ways media forms differ.

    ~   Eight #AXES capture the independent ways media forms differ.

    ✓   The #AXES capture the independent ways media forms differ.
```

The middle form is *checkable*: a cardinal, adjacent to a named graph node whose length is statically known. A numeral lexicon, a graph lookup, an AST count. No natural language processing, because the author performed the binding.

The third form is *better*, and the reason is worth internalising.

**The number in the prose is denormalised data.** "Eight dimensions" restates a fact the code already holds, in a second location, with no invalidation strategy. That is the whole of the eight-versus-five, and it is a problem that was solved in the 1970s by the expedient of not doing it.

Generalised:

> **Prose about code is denormalised code. Every sentence restating a fact the code holds is a cache with no invalidation.**

Literate programming has this failure mode *constitutively*, not accidentally. It is the second hard problem in computer science, arriving on schedule.

**Rule:** do not state cardinalities, ranges, or enumerations in prose when the code holds them. Name the collection and let the reader look. If the number genuinely carries argumentative weight, mark it — but ask first whether the sentence survives its removal. Usually it improves.

**Standing proposal to the language:** let `weave` interpolate. Write `#AXES.length` in prose, render `5`. Normalised source, computed presentation. Divergence becomes *unrepresentable* rather than merely detectable, which is always the superior move. The cost is real and should be weighed: raw `.lob` ceases to be clean plain text, and notlob's whole aesthetic is that the source reads as prose. `The #AXES.length dimensions…` is uglier than "eight". Possibly not worth it for cardinals. Probably worth it for anything a reader would otherwise be tempted to state twice.

### 6a. Why explicit naming eliminates rather than manages false positives

Consider `roman/numerals.lob`:

> *"Roman numerals are a positional notation using seven letters."*

The `NUMERALS` table directly beneath has **thirteen** entries, because it enumerates the subtractive pairs — `CM`, `IV` — which are not letters. The referent of "seven" is a *derived subset* of the table: its single-character keys. No node in any graph corresponds to it. The author computed it in their head.

A checker that binds numerals to nearby collections by proximity fires a false positive here, on the second-most-prominent file in the project. A checker that fires only when the author wrote `#NUMERALS` stays silent, correctly, because the author never claimed the table had seven of anything.

**Unmarked prose is not a claim.** The discipline of naming *is* the disambiguation, and the false-positive rate is not managed down — it is structurally eliminated.

### 7. Any check that requires NLP is a check where a name is missing

If you find yourself wanting a dependency parse to work out what a sentence is talking about, you have found a place where the document declined to say. Xunzi's rectification of names, arrived at from the other direction: the name is where obligation attaches. You cannot be held to a claim about a thing you never consented to name.

### 7. State taste as taste

The five axes of the media project — Disposable, Sensory, Moving, Global, Interactive — are a values judgement dressed as integers. Putting Novel at durability 9 and TikTok at 1 is an argument, not a measurement.

The module is honest about this by never pretending otherwise, but *nothing in the format made it be*. The tests verify that the data is well-**formed** (every vector has length 5, every value lies in 1–10). Nothing verifies, or could verify, that it is well-**chosen**.

**Rule:** when prose asserts something no claim can witness, say so in the prose. "These axes are a judgement; the clustering only reveals structure we put there." One sentence. It costs nothing and it stops a force-directed layout from doing the work of an argument.

---

## Part III — Claims

### 8. A claim is a property with a witness

The best idea in notlob, and it should be used as such. `~property` quantifies; `~example` substitutes a concrete witness for the quantifier. They are the same statement at different strengths, not two unrelated mechanisms.

Prefer, therefore, to write them **together and adjacently**, so the reader sees the general law and the instance that makes it graspable:

```
~property
    @given(n=st.integers(min_value=1, max_value=3999))
    def _(n):
        assert from_roman(to_roman(n)) == n

~example
    from_roman(to_roman(1994)) == 1994
```

The property is the claim. The example is the handhold.

### 9. `~example` is argument; `#Tests` is appendix

This distinction is doing real work and should be respected.

An `~example` in the body is **rhetorical**. It is there because the reader, at that point in the essay, needs to see the thing to believe the sentence. It is chosen for illustrative power. Three or four is usually the limit before the prose drowns.

A `#Tests` block in the post-text is **epistemically humble and exhaustive**. It does not persuade. It covers. Boundary conditions, regressions, the tedious sweep of the input space. Nobody reads it, and that is correct — it is the appendix, bound into the same volume, tonally distinct.

**Symptom that you have confused them:** an `~example` block with nine assertions in it. Move seven to `#Tests`.

**Symptom of the reverse:** a `#Tests` block containing the *only* demonstration of what the function does. Promote one to the body, where the argument needs it.

### 10. A deliberately failing claim needs a name

`examples/roman/roman/numerals.lob` contains a claim that is intentionally wrong. The prose says so explicitly — it is bait for the runner's failure path, and it makes the non-rule (`IIX` is not 8) concrete, which is a lovely piece of pedagogy.

There is no sigil for this. `notlob test` reports `FAIL`, exits 1, and is indistinguishable from a genuine regression to any process that isn't a human who already knows the joke.

**Present rule:** do not ship intentionally failing claims. If you must demonstrate a falsehood, assert its negation:

```
    ✗   ~example
            to_roman(8) == 'IIX'      # deliberately wrong

    ✓   ~example
            to_roman(8) != 'IIX'
            to_roman(8) == 'VIII'
```

**Standing request to the language:** this wants a sigil. `~counterexample`, perhaps — a claim expected to be false, failing the build if it ever passes. The pedagogical use is real and the vocabulary should meet it rather than forcing authors into double negatives.

### 11. `~run` is the only place side effects live

Printing, I/O, mutation of the world. Everything else in the body should be definitional. This is `if __name__ == "__main__"` with the ceremony made honest, and it means a module can be imported by another without setting anything on fire.

---

## Part IV — The graph

### 12. Every module names exactly what it uses

`#References` is explicit. There is no implicit package import, and the `imports` check is the one *error*-severity semantic check in the tool — a module that imports another and uses none of its symbols fails the build.

This is right, and the corollary should be honoured in prose too: **if you mention a symbol from another module, `#`-reference it on first mention.** The `references` check is currently advisory. Treat it as though it were not. It is the mechanism by which §6 becomes possible.

### 13. Title, address, and path are one fact stated three times

`#Pricing Discounts` → `pricing/discounts` → `pricing/discounts.lob`. The tooling enforces it. Do not fight it, and in particular do not choose a title for its filesystem consequences. Choose the title that names the concept; accept the path.

Subheadings are flat. `##` does not nest inside `##`. If you want a third level you want a second module.

### 14. Write `##Heading`, not `## Heading`

Both parse. The shipped examples are inconsistent — the README's inline Roman example uses the space, most `.lob` files do not. Pick the tight form. It distinguishes a notlob subheading from a Markdown one at a glance, which matters in a format that is deliberately Markdown-adjacent and deliberately not Markdown.

Trivial, arbitrary, and exactly the kind of thing a style guide exists to settle so nobody has to think about it again.

---

## Part V — On tooling, for whoever builds the next layer

Two failures have been observed in the wild. Neither requires a transformer.

| Failure | Mechanism that catches it | Model needed? |
|---|---|---|
| "eight dimensions" beside a five-element array | numeral adjacent to `#`-named node; compare against statically-known length | No — arithmetic and a graph query |
| "built in four stages, stage 1 was…" | temporal deixis lexicon (~150 closed-class entries) | No — a word list |

Two independent failures, zero transformers. This is a small sample and should not be oversold. But it is evidence against the seductive answer, and the seductive answer is expensive.

The deixis check is not a tense check. Tense is the wrong target: `-ed` drowns instantly on *sorted list*, *nested loop*, *cached value*, *readonly interface*. All participles, none historical. Target instead the closed class of expressions whose meaning depends on the moment of utterance. High precision, low recall — and defend the low recall rather than apologising for it. A lint that fires rarely and correctly stays switched on for years. A clever one gets muted in a fortnight.

If a model layer is ever built, it must obey one constraint absolutely:

> **Deterministic checks emit findings. Model layers emit questions.**

Different verb. Different command — `notlob review`, never `notlob check`. The moment model-judgement and graph-fact share an exit code, you have rebuilt at the tool level the exact laundering that killed the axes: a green mark that means *verified* sitting next to a green mark that means *seemed fine to me*.

And a rendering proposal, offered as the highest-leverage change available and the one nobody has tried. `notlob weave` currently decorates what has been verified. **Invert it.** Mark the prose that has *nothing standing under it* — the assertions no claim was ever pointed at. As the field of green grows, the eye stops reading. Show the reader the naked sentences instead. This is not a checker. It is a change to what the document looks like, and it attacks the cognitive failure rather than the syntactic one.

Note what this actually accomplishes, given §4: adjacency has no failure mode, and cannot acquire one, because two things cannot stop being next to each other. Marking unwitnessed prose is the only available way to **give adjacency a failure mode** — not in the code, where it is impossible, but on the page, where the last real checker is still looking.

The eight-versus-five would have been visible on the page.

---

## Appendix — The rules, without the arguments

1. No temporal deixis. The file has no now.
2. History goes in git. The scratchpad is ephemeral and gitignored; the discipline is the emptying, not the writing.
3. Write for a reader who never saw the process — and give the process somewhere else to go.
4. Adjacency is not verification. Reference is proximity in a space where edges are authored, directed, and breakable.
5. A claim witnesses only what it names.
6. **Do not restate in prose what the code already holds.** Prose about code is denormalised code: a cache with no invalidation. Name the collection; don't count it.
7. Any check that requires NLP is a check where a name is missing.
8. State taste as taste.
9. Property first, example as handhold.
10. `~example` persuades; `#Tests` covers.
11. Don't ship failing claims. Assert the negation.
12. Side effects live in `~run`.
13. `#`-reference every foreign symbol on first mention.
14. Choose the title, accept the path.
15. `##Heading`. No space.
16. The opening paragraph is the lowest-salience slot in the module — not because nobody reads it, but because **nobody edits it.** Put no unwitnessed cardinality claim there.
17. Comment rot is a missing-artefact problem, not an indiscipline problem. A lint says *don't*; an artefact says *there*.
18. Short modules are not an aesthetic. They are the mechanism that makes full reads cheaper than scanning. **Affordances beat nudges** — measured, in `pn-chomper`: two documents demanded property tests; the binding made them a silent no-op; zero were written.
19. **Prose survives if and only if it is edited in the same operation as the code it describes.** Prefer to put durable claims where the edits land — a test-group label outlives an essay paragraph.
20. Name a test group after the *design insight*, not the mechanical precondition. `##join room only reachable via coordinated two-token play` is better literate programming than most essays, and by §19 it is the only prose in the file that will still be true next year.
21. A passing claim says nothing about whether the thing it tests is *reachable*. Notlob errors on unused imports and is silent on unreachable exports.

---

*Written after reading every example in the repository and running all of them that would run. Corrections welcome and expected. The rules with corpses behind them are §1, §4, §6, and §10; the rest are argued from first principles and should be trusted proportionally less.*
