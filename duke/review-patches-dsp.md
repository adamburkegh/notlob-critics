# The Theorist Checks His Own Work

### A review of `patches-dsp` (notlob Rust binding, v0.3.0)

*Duke Fox, July 2026. Five modules, roughly 900 lines of literate Rust, fourteen properties that compile to real proptest harnesses — and one of them found a stability bug in the shipping DSP library this port was carved from.*

---

## I. A note on the author, because it changes the reading

This port was written by Dominic Fox. The same Fox whose observation about theory-traces — that a language model can follow the traces of a theory through a codebase without ever holding the theory that would let it extend the thing without aberration — is the third name in notlob's own lineage, cited in the project's `DESIGN.md` a few lines below Knuth and Naur.

So this is not a user kicking the tyres. It is the theorist behind the tool's central justification taking that tool and porting a slice of his own audio-DSP library into it. If notlob is a machine for holding theory in a form both people and models can share, then `patches-dsp` is the closest thing the ecosystem has to the designer of the engine driving his own car around the track. One expects either an unusually convincing demonstration or an unusually revealing failure. It is mostly the former, which is the more surprising outcome and the one worth explaining.

One historical point frames everything below. This port targets **notlob 0.3.0, from May** — a version with no `check` command. The entire semantic-consistency layer that every other review in this collection leans on did not yet exist. Whatever this project gets right about consistency, it got right *by hand*, or by the discipline of the format alone, without a linter watching. Whatever it gets wrong, the tool of the day could not have told anyone.

---

## II. The blockquote that justifies the whole enterprise

In the middle of `svf.lob`, in the section deriving the stability bound for a Chamberlin state-variable filter, there is this:

> *The `patches-dsp` source this module is derived from solves `f² + fd = 4` here, a looser bound that lets `f` cross the real stability edge — at `q = 0` a DC input near 8.8 kHz diverges. The `clamp-is-stable` property below caught the discrepancy; the formula above is the corrected bound.*

I did not take this on trust, because the entire point of this collection is that a good-looking claim proves nothing until it runs. So I went to the upstream library — the full `patches` monorepo, supplied alongside — and read `patches-dsp/src/svf/mod.rs`. Line 67:

```rust
let f_max = 0.5 * (-d + (d * d + 16.0).sqrt()) - 0.05;
```

That is the loose bound: it solves `f² + fd = 4`. The notlob module uses `(-d + (d*d + 4.0).sqrt()) - 0.05`, which solves `f² + 2·f·d = 4` — and the Jury stability test on the Chamberlin characteristic polynomial gives `f² + 2fd < 4` as the actual condition. I worked the arithmetic. At `q = 0`, damping `d = 2`: the upstream clamp permits a coefficient whose `f² + 2fd` reaches **6.15**, well outside the stable region of 4; the corrected bound holds it at 3.72, inside. The unstable coefficient maps back to a cutoff up in the 9 kHz region. The correction note is not a flourish. It is **true**, and the filter it describes is a real product that really diverges.

Sit with what happened here. A property test — `clamp-is-stable`, a one-line universally-quantified assertion that the clamped coefficient always lands in the stable region — was written as part of *documenting* the filter in literate form. In the writing, it failed. The failure was a genuine defect in the production Rust the port was derived from. The literate rewrite was more correct than its own source, and it knows exactly why, and it says so in the margin with a citation to the property that did the work.

This is the notlob thesis discharged in full. Not "claims make prose checkable" as a slogan, but: *the act of expressing code as a theory, with its invariants promoted to executable claims, surfaced a bug that ordinary unit tests in the original had missed.* Every other review in this collection has been an autopsy of the gap between what prose asserts and what code does. This is the first one where closing that gap paid out in a caught defect. It is the strongest single artefact I have reviewed, and it is not close.

---

## III. The properties are real, and they are the right properties

I could not execute them — the Rust runner shells out to `cargo`, and there is no `cargo` in my sandbox. But two things stand in for execution. First, the runner is honest about absence: read its source and you find that when `cargo` is missing, every claim returns an ERROR, not a pass and not a silent skip. The lesson of the TypeScript `~property` hole — that a silent no-op is worse than an honest failure — was already learned here, in an earlier version, by a different author. Second, the properties are simple enough to verify by reading, and one of them I verified by reimplementing its arithmetic outright.

What lifts this project above competent is *which* properties were chosen. Run them against the catalogue of property shapes and it is a near-complete house:

- **`closed-under-flush`** (denormal) — idempotence plus range restriction. `flush_denormal` applied to its own output is a fixed point, and the output set is `{0} ∪ normals`. This is the *closure* property, and it is the correct thing to say about a flushing primitive: not "it works on these four values" but "its image is closed under reapplication."
- **`clamp-is-stable`** (svf) — the invariant-preservation property that found the bug. For all coefficients and resonances, the clamp lands inside the stable region.
- **`damping-monotone`** (svf) — a *monotonicity* relation: more resonance knob, less damping, everywhere.
- **`poly-lanes-track-mono`** (svf) — a **metamorphic** property, the hardest and most valuable shape in the set. Sixteen SIMD voices fed identical input must reproduce the scalar kernel lane-for-lane, within a relative tolerance. This is precisely the property that catches structure-of-arrays refactor bugs, the class of error most likely to slip through example-based testing because the scalar and vector paths look obviously equivalent right up until one isn't.
- **`advance-reaches-target`** (ramp) — a *convergence* property: a ramp begun toward any target over any interval arrives, to tolerance, after exactly that many steps.
- **`bounded-input-bounded-output`** (svf), **`silence-has-no-denormals`** (blocker), **`voice-independence`** (ramp) — stability and non-interference invariants over long sample runs.

Nobody handed Fox a menu. He wrote the menu, in effect, before the style guide that tried to reverse-engineer it existed. The `~property` catalogue this collection has been groping toward — roundtrip, preserves, monotone, idempotent, metamorphic — is here, populated, in a real project, correctly matched to the mathematics of each kernel. If I needed one artefact to teach a new author what a property is *for*, as distinct from an example, it would be this one.

And the examples know their place. They are handholds — `q_to_damp(0.0) == 2.0`, a single settling run — chosen to make a sentence palpable, three or four to a section, never straining to cover the space. The `#Tests` appendices do the exhaustive, tonally-flat sweep. The division of labour the format prescribes is simply *observed* here, without anyone appearing to try. That is what a well-shaped format buys when a strong author uses it: the conventions stop being rules and become the path of least resistance.

---

## IV. Where it drifts — and the one place the format could not help

For all that, this project is not immune to the disease the rest of the collection documents. It is only a milder case, and the drift sits in exactly the predicted location.

`binding.lob` opens:

> *Three modules are included: a denormal-flushing primitive, time-domain utilities, and a DC-blocking highpass filter…*

The directory holds **five** `.lob` modules. Beyond the three named, there is `svf.lob` (412 lines, the largest and best in the project) and `coef/ramp.lob` (250 lines, the shared primitive the SVF depends on). The binding-overview prose describes an earlier, smaller state of the port. The SVF and the ramp were added; the sentence that counts the modules was not revisited.

This is the eight-versus-five signature, for the fourth time across four projects, and once again it lands in **the one paragraph nobody re-edits** — the module-opening overview, written once at composition and thereafter structurally excluded from every subsequent operation. The rule from the earlier reviews holds without a wobble: prose survives if and only if it is edited in the same operation as the code it describes, and a binding-overview paragraph is never in the same operation as the addition of a whole new module in a different file.

Three things make this instance more interesting than a simple repeat.

First, **the tool of the day was blind to it.** Version 0.3.0 had no `check`, hence no `references` advisory, no coverage line reporting module counts, nothing. Even the weak deterministic detection this collection has been proposing — a numeral in prose beside a countable graph fact — did not exist. Fox could not have been warned. This is the cleanest possible demonstration that the failure is *structural to the medium*, not attributable to a careless author or a missing lint: give the format's own designer-adjacent theorist the format without the guardrail, and the guardrail's absence shows up in precisely the place the guardrail was later built to watch.

Second, **it is a cardinality restatement — a denormalisation.** "Three modules" is a fact the filesystem already holds; writing it into prose created a cache with no invalidation, and it went stale the moment a fourth module landed. The style-guide rule *do not restate in prose what the code already holds* would have prevented it outright: the sentence wants to be "the modules under `patches/`", named by reference, not counted by hand.

Third, and to the author's considerable credit, **the cross-module references themselves did not rot.** `blocker.lob` references `#Patches Denormal` and uses its `flush_denormal`; `svf.lob` references `#Patches Coef Ramp` and builds on its `CoefRamp<2>`. I checked both against the graph. They resolve. The *load-bearing* references — the `#`-edges that are authored, directed, and breakable, the ones the whole binding argument rests on — are intact, because they are exercised by the code and would break the build if wrong. Only the *decorative* count in the overview drifted, because nothing executes an overview. This is the distinction the collection keeps arriving at, illustrated once more: the reference held, the number did not, and the difference is whether the claim has a failure mode.

There is a smaller note. `svf.lob` at 412 lines is at the very top of the Montaigne band, arguably past it — it carries coefficient math, a scalar kernel, and a full polyphonic SIMD kernel. A case could be made to split the polyphonic kernel into its own module, as the ramp primitive was split. But the counter-case is real: the poly kernel's defining property is that it *equals* the mono kernel, and `poly-lanes-track-mono` needs both in scope to say so. Splitting them would put the metamorphic property in an awkward position relative to the two things it relates. This is length earned by a genuine conceptual unit, and I would leave it. The test is whether a competent reader can hold the whole file at once, and a filter engineer can.

---

## V. What it settles, and what it opens

`patches-dsp` is the counter-example the collection needed. Every prior review has been, in one way or another, a catalogue of the format's failure modes: prose that lies, claims that decorate, properties that skip, artefacts that can't be reached, pipelines that can't be checked. A reader of those reviews alone might reasonably conclude that notlob is a well-argued aesthetic whose central promise — theory made checkable, held across people and machines and time — does not survive contact with real code.

This project is contact with real code, by a strong author, and the central promise survives. More than survives: it *pays out*, in the form of a stability bug caught in the writing, in a shipping library, by a property that existed only because the literate form asked for one. That is the demonstration the arXiv paper needs and does not yet have — an existence proof that the claim layer does work the format cannot do without it, on a problem hard enough that ordinary testing missed it.

Two things it opens, for the notebook rather than the verdict.

The drift in `binding.lob` proves the prose-rot failure is a property of the *medium*, not of any author or any tool generation, because here it appears under the most favourable possible conditions — expert author, strong discipline — and in the absence of any tool that could have caught it. That strengthens the case for the deterministic guardrails considerably. The failure is not a skill issue. It is a physics issue.

And the quality of the properties here raises a question the style guide should sit with. The catalogue was built by generalising from what projects *lacked*. This project lacked nothing — it is the positive template the catalogue was trying to describe. It may be that the fastest way to teach the property discipline is not a menu of shapes in a document nobody reads, but *this file*, in the graph, queryable: `show me a metamorphic property` returning `poly-lanes-track-mono` from a real kernel. Docs are read once. A corpus is queried at need. The best example set in the ecosystem is now sitting in it, and the tooling should make it reachable.

---

*Verified by: full reading of all five DSP modules; extraction of the upstream `patches-dsp/src/svf/mod.rs` and line-level comparison of the two stability bounds; independent reimplementation of the Jury stability arithmetic confirming the upstream bound admits an unstable pole at q=0 near 9 kHz; graph queries confirming cross-module reference resolution; and inspection of the Rust runner's cargo-absent error path. Properties were not executed — no `cargo` in the review environment — but the runner compiles them to real proptest harnesses and errors honestly on toolchain absence. The module-count discrepancy in `binding.lob` was confirmed against the directory. Quotations are from the project as supplied.*
