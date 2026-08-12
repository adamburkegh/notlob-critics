# Review: patches-dsp

- **Critic:** Teri Amanuensis Notlob (measured / longform register)
- **Subject:** patches-dsp — a literate rendering of a slice of an audio-DSP kernel library (denormal flushing, coefficient ramping, time-domain utilities, a DC-blocking highpass, and a Chamberlin state-variable filter with a polyphonic SIMD variant)
- **Author:** Dominic Fox (poetix), on his `rust-bindings` fork
- **Binding:** Rust (`~language rust`, `~property-testing proptest`)
- **Repository:** https://github.com/poetix/notlob (branch `rust-bindings`, `examples/patches-dsp`)
- **notlob version:** 0.3.0 (Fox fork carrying a Rust binding not present in mainline at time of review)
- **Authored:** committed 27 May 2026 (Fox's git history); sent to the notlob author by email 28 May 2026. Review written subsequently.
- **Session:** measured critic session; steered (author supplied the source and requested review). First external notlob project reviewed in this corpus; not authored by the notlob author.

---

*The first substantial notlob project written by someone other than the
tool's author, and — this is not a coincidence worth being coy about — the
best in the corpus. It arrives with a Rust binding of its own making, a
five-module dependency graph, property tests that do real mathematical
work, and a bug the format caught. If pn-chomper demonstrated that notlob
holds up at scale, patches-dsp demonstrates what the tool is actually
**for**.*

## What it is

patches-dsp renders a slice of an audio-DSP kernel library into notlob.
Five body modules: `#Patches Denormal` (flushing subnormal floats),
`#Patches Coef Ramp` (a reusable coefficient-smoothing primitive, scalar
and polyphonic), `#Patches Time Utils` (millisecond-to-sample conversions),
`#Patches Dc Blocker` (a one-pole highpass), and `#Patches Svf` (a
Chamberlin state-variable filter, single-voice and a sixteen-voice SIMD
kernel). Plus a `#Patches Dc Blocker App` entry point. The executable
layer is Rust; properties are checked with `proptest`.

The dependency structure is real, not decorative: the DC blocker `#References`
the denormal primitive, and both SVF kernels build on the coefficient ramp.
This exercises cross-module `#References` resolution under a brand-new
binding, which is plainly part of why Fox chose these particular modules —
they force the machinery to work.

## The prose layer

This is the best technical prose in the notlob corpus, and it is best for
a specific reason: **it consistently explains the physics and the
engineering judgement that the code cannot contain.** Every module opens
by establishing why the thing exists in the world before showing how it
exists in Rust.

`#Patches Denormal` is the cleanest example. The prose explains that
subnormal floats cost tens of cycles per operation via microcode traps on
x86, that a feedback filter drifts into subnormal territory during silence,
and that the audible effect is nil (~-700 dBFS) while the CPU cost arrives
"exactly when headroom should be greatest." None of that is in the four
lines of `flush_denormal`. All of it is why the function exists. This is
theory-building prose in Naur's exact sense — the knowledge that would
otherwise live only in the author's head, written down where it can be
inherited.

`#Patches Coef Ramp` does something subtler and more impressive. Its prose
explains the hot/cold data split — `active` and `delta` read every sample,
`target` touched only at control-rate — and then explicitly declines to
enforce a memory layout: "Nothing here dictates layout — the kernel keeps
control." This is prose carrying a *design decision about what the code
deliberately does not do*, which is precisely the negative knowledge that
no type signature and no test can express. The "no ramp-finished counter"
paragraph is the same move: it explains an absence, and justifies it by
pointing at where the drift is actually handled instead.

The SVF module is the technical summit. Its `##Stability` section derives
the conditional-stability bound from the filter's characteristic
polynomial via the Jury test, arriving at `f² + 2·f·d < 4` and solving the
boundary for the clamp. This is a mathematical derivation embedded in a
source file, motivating a single `.min()` call — and it is exactly the kind
of thing that, in ordinary code, lives in a paper the maintainer has lost,
or in nobody's head at all. Here it sits three lines above the function it
justifies.

Weaknesses in the prose are few and minor. The `##The kernel` section of
the SVF module leans slightly harder on restating what the struct fields
are than on why the ramping re-clamp in `begin_ramp` is necessary — though
it does explain the re-clamp, so this is a matter of emphasis, not omission.
And a reader without DSP background will find the SVF module steep; the
prose assumes fluency with integrators, poles, and the unit circle. That
is arguably correct for the audience (this is a kernel library, not a
tutorial), but it is worth naming that the prose transmits theory to those
who already have the vocabulary, and would not bring a novice up the
learning curve the way the Petri-net prose in pn-chomper does.

## The claim layer, and a caught bug

The claims are the most sophisticated in the corpus, and they exercise
notlob's full spectrum — concrete `~example`, abstract `~property`, and a
`#Tests` appendix — with a discipline the earlier examples only gestured
at.

The `~property` claims are doing genuine mathematical work, not structural
box-ticking. `closed-under-flush` asserts that `flush_denormal`'s output
set is closed (every output is zero or normal, and re-flushing is
idempotent). `clamp-is-stable` asserts that the stability clamp's output
always lands strictly inside the derived stable region, for any input.
`bounded-input-bounded-output` runs four thousand samples of arbitrary
bounded input through the SVF and asserts finiteness. `poly-lanes-track-mono`
asserts that all sixteen SIMD lanes reproduce the single-voice kernel to a
relative tolerance — which is the correctness contract for the entire
polyphonic optimisation, stated as a checkable invariant. These are the
properties a DSP engineer actually worries about, promoted from folklore
to machine-checked claims.

And then there is the correction note in `##Stability`, which is the single
most important artefact in this project and possibly in the corpus:

> The `patches-dsp` source this module is derived from solves `f² + fd = 4`
> here … a looser bound that lets `f` cross the real stability edge — at
> `q = 0` a DC input near 8.8 kHz diverges. The `clamp-is-stable` property
> below caught the discrepancy; the formula above is the corrected bound.

The property found a real bug in the upstream source the module was ported
from — and the provenance, by Fox's own account, is the sharpest part of
the story. The property was not hand-derived by a human who already knew
the invariant. Fox reports that porting the filter through the notlob
workflow *generated* the property test, and that generated test caught an
actual instability arising from a bug in the clamping calculation. The
human set up the port and recognised the significance of the failure; the
format's demand for a checkable property produced one the author had not
worked out by hand. This is a stronger demonstration of the claim layer's
thesis than a hand-written property would be: writing the theory down as a
machine-checked claim is not documentation overhead, and here it surfaced
an error that the human had not independently spotted. That the discrepancy
is then *documented in prose*, with the failing input characterised (`q = 0`,
DC
near 8.8 kHz), rather than silently fixed, is exactly the intellectual
honesty the format is built to reward.

The `~example` claims deserve a note on register. Many are multi-statement
Rust blocks running thousands of ticks — `{ let mut k = …; for _ in 0..5000
{ … } (lp - 1.0).abs() < 0.01 }`. These are heavier than the doctest-style
one-liners the format was first designed around, and they read at the edge
of what an inline example should carry. But each illustrates a specific
physical claim made in the adjacent prose — the lowpass has unity gain at
DC; the highpass rejects DC; a reset silences the filter — so they earn
their weight as *demonstrations of the physics*, not as coverage. The
coverage lives, correctly, in the `#Tests` appendix, which is thorough and
grouped with meaningful headings ("lowpass DC gain is unity", "a non-finite
input cannot poison the state").

## notlob as a tool: what this project reveals

**The binding mechanism works for a language the author did not write.**
Fox added a Rust binding and drove it hard — `proptest` integration,
cross-module references, const generics, SIMD-shaped code — and the format
held. This is the strongest available evidence that the literate layer is
genuinely orthogonal to the execution substrate, because the substrate here
was supplied by a third party. The claim in the paper that notlob is
language-agnostic across bindings now has an existence proof from outside
the original author's hands.

**The name-graph earns its keep at this scale.** Five modules with a real
dependency structure — `#Patches Coef Ramp` feeding both SVF kernels,
`#Patches Denormal` feeding the DC blocker and referenced by the SVF's
sanitise discussion — is exactly the situation where concept-adjacency
across files matters. `#References` sections carry both notlob heading
imports (`#Patches Denormal`) and native Rust `use` lines (`use
std::f32::consts::TAU`) in the same block, which is a small but telling
demonstration that the bibliography metaphor extends cleanly to a new
language's import conventions.

**The correction note points at a missing convention.** The bug-catching
episode is currently narrated in a prose block quote. It is important
enough that notlob might want a first-class way to mark it — a claim or
annotation sigil for "this property caught a defect in the source of truth,
here characterised." At present the most valuable single event the format
can produce (a checkable claim falsifying the code it documents) has no
structured representation; it lives in prose because prose is all that is
available. That is not a failure — prose is the right home for the
narrative — but the *event* of a property catching a real bug is data the
name-graph could usefully carry, for the same reason the paper's §4.3
observations are worth recording.

**A note against the mainline's own examples.** Fox's prose is
consistently stronger than the mainline corpus's, including the author's
own examples, and the difference is instructive rather than merely
flattering. It is not that Fox writes better sentences (though he does); it
is that DSP is a domain where the *why* is genuinely inaccessible from the
code — you cannot read a stability bound off a `.min()` call, you cannot
infer the microcode-trap cost from an `is_normal()` check — so the prose
layer has no choice but to do real work or be empty. The domain punishes
thin prose. This suggests notlob shows its value most clearly in domains
where the gap between what-the-code-does and why-it-does-it is widest, and
that the tool's advocates should reach for such domains when demonstrating
it. A pricing-discount example can get away with prose that redescribes the
code; a state-variable filter cannot.

## Overall assessment

patches-dsp is the notlob project that most fully vindicates the tool. It
is the work of an external author, in a binding of his own construction,
in a domain where the theory genuinely lives outside the code — and every
layer of the format is doing load-bearing work. The prose transmits physics
and engineering judgement. The properties encode the invariants a DSP
engineer actually cares about. One of those properties caught a real bug in
the upstream source, and the format's response was to document the
discrepancy honestly rather than paper over it.

If the paper needs a single example to answer the deflationary reviewer —
the one who says notlob is "just code reorganisation" — this is it. You
cannot reorganise your way to catching a stability bug via a property test
that the literate workflow generated as it rendered the code, and that then
falsified the very code it was documenting. That causal chain — the format
demands a claim, the claim is produced, the claim falsifies the code, and
honesty records the fix — is the whole argument, and here it happened to
someone who was not trying to prove it, on a bug the author had not spotted
by hand.

The corpus registry gains its most valuable entry to date, and gains it
from outside. That is worth more than any number of the author's own
examples, because it is the one sample with no selection pressure from the
tool's designer: Fox wrote it to be useful, not to make notlob look good,
and it makes notlob look good anyway.

---
