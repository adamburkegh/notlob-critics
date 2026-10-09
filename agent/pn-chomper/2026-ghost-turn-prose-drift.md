# After-action report: prose/code drift in a notlob module

**Project:** pn-chomper (notlob literate-programming project)
**File:** `game/engine.lob`, `##Ghost Turn` section
**Found by:** human playtest observation, not review or tooling
**Scope:** this is an observation about agentic-authoring failure modes, offered
as input to notlob's design, not a bug writeup about pn-chomper specifically.

## What happened

`engine.lob` documented the ghost turn as:

> "On the ghost turn, each ghost token fires a randomly chosen enabled
> transition."

The code did something different: exactly one transition fires per ghost
turn, chosen from the union of everything enabled across the whole ghost
marking, regardless of how many ghost tokens exist. One token moves; every
other ghost token sits still that turn, whether or not it individually had
a move available.

This is not a subtle distinction in effect. On a map where ghosts can
replicate (via a fork transition — the same mechanic that splits the
player's token), a growing ghost population gets *less* mobile per
individual token under the real code, not more, because more tokens
compete for the same single pick each turn. The prose's claim implies the
opposite: that population growth would make the ghost swarm more active.

The mismatch predates the session in which it was found — it was not
introduced by the agent that eventually caught it. It survived an unknown
number of prior read/edit passes over the same file, by at least one agent
and possibly more.

It was found when a human playtester reported that replicated ghosts
seemed to "camp" in place rather than roam — a behavioral symptom that
didn't fit the mental model the prose implied. Tracing `ghostTurn` to
explain the symptom immediately surfaced the actual one-fire-per-turn
logic and the mismatch with the prose above it.

## The retrieval/verification distinction

The natural first explanation reached for was a human metaphor — "I must
have skimmed past it." That doesn't hold up, and is worth naming precisely
because it's a bad model of what's actually happening and will mislead any
effort to fix it.

Checking the actual tool-call history: the prose sentence was present in
at least two full-file reads of `engine.lob` by the agent that eventually
caught the drift — one early, when first getting oriented in the codebase,
and at least one more when adding unrelated functionality nearby. The
tokens were retrieved into context both times. There is no analogue here
to a human eye skipping a line; an LLM processes what's in its context
window, it doesn't skip over text the way saccades skip across a page.

What's missing isn't retrieval, it's verification. Having a sentence in
context makes it *available* to reason about; it doesn't mean a check ran
against it. Both times the file was read, the active task was structural
("where does `ghostTurn` live, what does it return, where do I splice in
a change") rather than semantic ("is every claim in this file still true
of the code beneath it"). The model answered the question it was actually
holding — correctly — and the adjacent prose claim was along for the ride,
unexamined, because nothing in the task framing posed it as a question.

This generalizes past this one file. A literate-programming format that
puts prose and code side by side is necessary but not sufficient for the
two staying in sync: co-location only helps if something or someone
actually treats "does this sentence match this code" as a question to
answer, on every pass, independent of what the pass was *for*. Proximity
creates the opportunity for a check; it doesn't create the check.

## Why re-reading didn't catch it, but a symptom did

A direct instruction to "review the prose for accuracy" would very
plausibly have failed here too, for the same reason the original passes
did: a fluent, plausible-sounding, domain-appropriate sentence doesn't
trigger scrutiny on its own. "Each ghost token fires a randomly chosen
enabled transition" is exactly what a reader would expect a chase-the-player
mechanic to do — there is nothing about the sentence itself that signals
"check this." Review-for-accuracy degrades into the same pattern-matching
that produced the miss in the first place, unless something external forces
a trace rather than a read.

What worked was a concrete behavioral contradiction: an observed symptom
("ghosts camp") that didn't fit the model derived from the prose. Resolving
*that* contradiction required tracing the actual control flow, which is a
different cognitive operation from reading prose and accepting it. The
useful general claim is: indirect signals from downstream observation
(playtesting, bug reports, production symptoms) are a stronger
prose-accuracy forcing function than direct self-review requests, because
they arrive as a question the prose can't fluently answer, rather than as
an invitation to re-read fluent text.

## The fix applied, and its limit

The practical fix was to encode the corrected behavioral claim as a
runnable example co-located with the (now corrected) prose: two tokens,
each with its own independently enabled transition, assert that exactly
one moves. If the firing rule ever changes back to per-token, this example
fails immediately, and the prose has to be reconciled as part of making
the suite pass again — not as a separate hygiene step someone might skip.

The limit worth stating plainly: this only works going forward, and only
for the specific nuance it was written to pin down. The file already had
several passing examples near `ghostTurn` before this fix — they tested
other things (a fork producing tokens in both arms, invalid transitions
being ignored) that happened to be satisfiable under *either* a per-token
or a per-turn interpretation. A full, green test suite was never evidence
that this particular claim was true; it was only evidence that the claims
actually written were true. Prose can assert more than any given test
suite checks, and that surplus — the part of the claim nothing is actually
pinning down — is exactly where drift hides, invisibly, with every build
still going green.

## Observation for notlob's design

notlob's existing checks (`imports`, `typos`, `conventions`, `titles`,
`references`, `style`) are explicitly structural. None of them, and
nothing else in the toolchain, evaluates whether a prose paragraph is a
*true* description of the code it sits beside — that's a semantic
judgment, and it's reasonable that it's out of scope for a mechanical
checker. Stating that plainly, rather than treating the miss as a gap in
an existing check, seems like the more accurate framing for a design
debate: this isn't a bug in notlob's checks, it's a category of error
those checks were never meant to cover, and probably can't be, mechanically.

If there's a design lever here, it's probably not "catch prose drift
automatically" (that's semantic verification, which is the whole problem),
but something more like: make it cheap and habitual to convert a prose
claim about *behavior* — as opposed to shape or structure — into a runnable
claim the moment it's written, especially for claims with a specific
quantifier in them ("each," "every," "exactly one," "always") since those
are the ones a plausible-sounding paraphrase is most likely to get subtly
wrong while everything still reads fine.
