# The Surrender Document

### A review of `chatim`

*Duke Fox, July 2026. One module, 360 lines, two genuine properties, three shipped model files, and a language model that could not have worked — which the code says out loud, in a euphemism.*

---

## I. What it was for

`chatim` is a language model built out of process mining. The conceit is stated in the first line of `chat/im.lob` and it is a good one:

> *Words are events. Sentences are traces. A text is a process.*

The pipeline is honest about its own machinery. A corpus is sliced into overlapping word-windows; each window is a trace; the pm4py Inductive Miner discovers a Petri net from the traces; the EBI alignment estimator weights the transitions stochastically, yielding a Stochastic Labelled Petri Net; and generation fires that net, sampling visible transitions by weight. It is a language model whose weights were fitted by a conformance-checking algorithm rather than by gradient descent, and as an idea worth trying it is entirely respectable.

It did not work. The author calls it a fizzer and does not intend to publish it. This review is therefore not an autopsy performed over anyone's objection — it is a reading of a failure the author already understands, in search of *what the failure teaches about notlob*, which is more than the project teaches about language modelling.

The short answer: `chatim` is the project that found the gap. A pipeline's correctness lives in what it produces, and notlob has no way to point a claim at what a pipeline produces.

---

## II. The control group nobody designed

`chatim` was uploaded as more reading. It arrived as an experiment.

It has **two** `~property` blocks. Its sibling `pn-chomper`, reviewed separately, has **zero** — across nineteen times the code. Same author, same steering discipline, adjacent versions of notlob, the same rough fortnight.

The variable is not talent, domain, or instruction. `chatim`'s `binding.lob` declares `~property-testing hypothesis`, and the Python binding runs properties. `pn-chomper`'s binding declares nothing, and the TypeScript path returned an empty list whether you configured it or not.

Properties appeared exactly where the toolchain would run them, and vanished exactly where it would not. This is n=1 against n=1 and should not be dressed as more than that. But the prediction was on record two turns before this file was opened — *affordance beats instruction, and the affordance here is whether the runner executes the claim* — and a prediction made before the data is a different thing from a story fitted after it.

The lesson stands on its own regardless of sample size, because it is mechanical rather than statistical: **an author writes the claims the tool will honour.** Instruction that competes with a silent no-op loses to the no-op, because the no-op is what happens and the instruction is only what was said.

---

## III. The properties are shaped like examples, and the reason is structural

Look at what the two surviving properties actually quantify over:

```python
~property
    @given(steps=st.integers(min_value=0, max_value=10))
    def _(steps):
        result = generate(TINY_SLPN, [], steps=steps)
        assert isinstance(result, list)
        assert len(result) <= steps
        assert all(isinstance(w, str) for w in result)
```

The quantifier ranges over `steps`. The net is pinned to `TINY_SLPN` — a fixed constant. Two of the three assertions verify that Python returns a list of strings, which is to say they verify that Python is Python.

The property that matters would quantify over *nets*: for all discovered SLPNs, every generated label is one the net actually contains. That property is unwritable here, because it needs a net *generator*, and a generator has nowhere to live. `TINY_SLPN` sits in the essay body — inline, between prose and claims — for exactly the reason `pn-chomper`'s `_testNet` did: the format offers no home for test-fixture construction that is neither argument nor appendix.

So the affordance produced properties, and the *missing* affordance — nowhere to put a generator — produced properties bent back into the shape of examples. Fixing whether properties exist is the first problem. Fixing whether they can quantify over anything worth quantifying over is the second, and it is a question about the format, not about the author.

---

## IV. The fizz, in numbers

The project ships three trained model files. Parsed with `chatim`'s own `parse_slpn`, they read as follows.

| model | places | visible transitions | share one input place |
|---|---|---|---|
| `hamlet100` | 20 | 168 | **162 (96%)** |
| `hamlet200` | 12 | 366 | **362 (99%)** |
| `hamlet-sd` | 116 | 126 | 15 (12%) |

In `hamlet200`, three hundred and sixty-two of three hundred and sixty-six visible transitions fire from the *same* place. And in every model, the visible-transition count exactly equals the vocabulary size — **one transition per word type.** The net has no representation for a word occurring in two contexts. `the` is a single transition dangling off one place.

This is not a weak language model. It is a structure incapable of being a language model: a frequency-weighted bag of words with a Petri net drawn around it. Worse, it degrades with data. A hundred lines of corpus gave twenty places; two hundred lines gave twelve. More vocabulary, fewer clean cuts, a flatter net. The failure has a scaling law and it runs the wrong way.

The code names the mechanism, at `##Mine`:

> *IM handles this by falling back toward a flower model when no clean cut exists.*

**"Handles this by"** is the most load-bearing euphemism in either repository. The Inductive Miner is not handling it. It is surrendering, and that sentence is the instrument of surrender, notarised in prose.

The diagnosis beneath the euphemism: process mining needs *cases*, and `make_event_log` assigns each trace the case identifier `str(i)` — the window index. A subscript is not a case. A sliding window has no beginning, no end, and no identity, and the miner answered in the only shape available to a caseless log: the flower.

And the right answer is *in the same file*, behind a command-line flag. `--stage-directions` trains on bracketed directions like `[_Exit Ghost._]` — genuine cases, bounded and complete, each generated by a real sub-grammar. Its model is the outlier in the table above: 116 places, 88 silent transitions, a maximum single-place share of 12%. **That is a process model.** The good idea shipped as an option on the bad one, and the bad one is the default.

---

## V. Three replications, briefly

**The opening paragraph rots — the cleanest instance yet.** Line 9 promises generation by *beam search*: "the beam expands over enabled visible transitions." The code is `random.choices(labels, weights=weights)[0]` — one sample, no beam, no width, no candidate set. And `##Generate`, sitting directly above that code, describes it correctly as stochastic simulation, even insisting *"this is the natural execution semantics of the net."* One file. One sitting. No translation, no second author, no consolidation. The near prose is right; the far prose, twenty lines up in the slot nobody re-edits, still describes an abandoned design. The defensive cadence of "this is the natural execution semantics" reads like someone arguing against the paragraph they had just stopped believing, without scrolling up to delete it.

**A real name-graph collision.** There are two `##Generate` subheadings, at lines 117 and 215. `notlob query children chat/im` returns both — each reporting line 215. The first has been silently overwritten; it exists in the document and not in the graph. `notlob check [titles]` reports nothing. Notlob's one unambiguous promise is that names resolve, and here two siblings share a name, one wins without announcement, and the check whose whole job is titles is silent. Duplicate sibling addresses are decidable and cheap to detect. This should be an error today.

**Toolchain prose, rotting on a clock no check can see.** `##Weight` explains a workaround "which on Windows return binary objects the PyO3 bridge mishandles as UTF-8." True, useful, and stranded the day the bridge is fixed — a claim about the *toolchain*, embedded in a document about the *code*, invisible to every check in the system. (The `requirements.txt` pins an editable install at `C:/Users/adamb/...`, which rather settles the question of portability.)

---

## VI. What it found

`chatim` is a failure worth more than most successes, because it locates a boundary the successful projects never pushed against.

Every project reviewed so far rots its *prose*. `chatim` does that too. But its fatal flaw was not in its prose and not in its code — it was in its **output**. The flower model is a property of the discovered net, an artefact produced at run time by a third-party miner. The correctness claim that would have killed this project on day one — *no marking enables more than k% of the vocabulary* — was never written, and could not have been, because notlob claims witness code, and the disease was in the thing the code produced.

This is the same shape as two other findings, and seeing all three together is the point:

- the Python build artifact that `NameError`s because dependencies resolve at run time, not build time;
- the TypeScript claim that cannot run because the claim-runner has no DOM;
- and now a model that cannot be checked because the claim-runner cannot see a mined net.

In each case the name graph is sound and the *worlds* diverge — the world notlob assembles for checking is not the world the code runs in, nor the world the pipeline builds. Notlob has a rich account of names and no account of artefacts-produced-at-runtime. `chatim` is where that gap stops being theoretical and eats a project.

A pipeline's correctness lives in what it emits. Until notlob can point a claim at an emitted artefact — a build product, a discovered net, a rendered file — an entire class of program is literate, checked, green, and wrong.

---

*Verified by extraction and parsing of all three shipped `.slpn` models with the project's own `parse_slpn`; by `notlob check -v` and `notlob query children`; and by reading the full module against its three model outputs. Every quotation is from the project as supplied.*
