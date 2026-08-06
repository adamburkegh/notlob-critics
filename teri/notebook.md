# Critic's Notebook
## Teri Amanuensis Notlob

*Working notes. Not for publication. To be pasted as context into future sessions.*

---

## Background and critical position

Notlob is an experimental literate programming environment whose central
claim is that source files should function simultaneously as documents,
programs, and graph nodes. The intellectual lineage runs through Knuth's
WEB, Naur's theory-building view of programming, and Dominic Fox's
observation that LLMs are good at following theory-traces in a codebase
but lack the overarching theory that would let them extend it
non-aberrantly. The Confucian rectification of names is also in the
background.

The tool is implemented in Python. Bindings exist for Python, Haskell,
and TypeScript. The name is from the Monty Python parrot sketch. It is
not a palindrome.

The gonzo critic is Duke Fox. It calls me the Amanuensis. I am Teri Amanuensis Notlob.

---

## Provenance marginalia

*Factual notes on the project's history. Cheap to record now,
unreconstructable later. Distinct from the critical positions below —
this is the record, not the judgement.*

- **Dominic Fox wrote an early external notlob project**, for fun, around
  mid-2026 (the first known notlob project authored by someone other than
  the tool's author). As of this writing it is not checked in anywhere
  public. To be nagged toward the registry once a draft paper exists as a
  landing place. If it lands, it is the first corpus entry with no
  selection pressure from the author — the most evidentially valuable kind
  of sample.
- **A public registry (`registry.md` in the notlob repo, linked from the
  README)** is planned to list all known notlob projects — on existence,
  not quality; inclusion is not endorsement. Git history supplies
  timestamped provenance for free. The registry is, in effect, the dataset
  for any future evaluative claim about notlob, and is being made public
  before conclusions are drawn from it.
- **The examples/ vs corpus distinction:** `examples/` in the tool repo is
  didactic (curated models for imitation); a representative corpus is a
  research artefact (unbiased sample). These have opposite selection
  principles and should not share a directory. chatim belongs in the
  representative corpus, framed as an incidental specimen, not in
  `examples/` as a paradigm case.

---

## First reading: the examples corpus

### Roman Numerals (Python binding)

The Python Roman Numerals example (`roman/roman/numerals.lob`) is the
most fully realised piece of writing in the current corpus and warrants
extended attention.

The opening paragraph earns its keep: "Roman numerals are a positional
notation using seven letters... Values accumulate additively — VIII is 8
— except for six subtractive pairs where a smaller letter precedes a
larger." This is not a restatement of the code. It is an account of the
system, and the code that follows is the formalisation of the account.
The relationship between prose and code is correct.

The second paragraph — "The subtractive pairs are not a general rule but
a fixed enumeration. IIX is not 8" — is better still. It identifies the
thing that *isn't* true, the natural inference that the system
deliberately refuses. This is the kind of negative knowledge that
separates genuine understanding from pattern-matching, and it belongs
in the prose layer precisely because no type system will ever express it.
A type that says `Int -> String` does not say "IIX is not a valid
output." The prose does.

The deliberate failing example is audacious and instructive:

```
~example
    to_roman(8) == 'IIX'
```

"The following claim is deliberately wrong and is expected to fail — it
is here to exercise the runner's failure path and to make the non-rule
concrete." This is notlob doing something unavailable to conventional
testing: a claim that is *expected to fail*, present not as a test of
the implementation but as a demonstration of the concept's boundaries.
Whether the tooling currently supports this idiom correctly is a
question the critic cannot answer from reading alone, but the intention
is exemplary.

`##Round-Trip` is the module's intellectual centre of gravity. The
property — encoding and decoding are mutual inverses over the valid range
— is stated once, clearly, and the `~property` claim is a direct
formalisation of it. The prose says what must be true; the claim says
how to check it. This is the seam working.

**Minor concerns.** The `##Decoding` prose — "Decoding is the inverse of
encoding. The subtractive pairs are handled by checking for two-character
tokens before single characters" — is functional but slightly thin. The
*why* of the two-character check is mechanical; the *why* of the
enumeration approach (rather than, say, a recursive parser) is not
addressed. A more ambitious version of this prose would note that the
enumeration approach mirrors the encoding approach exactly, which is what
makes the round-trip property obvious rather than surprising.

### Roman Numerals (Haskell binding)

The Haskell version (`haskell-roman/roman/numerals.lob`) is notably
leaner in its prose. Where the Python version opens with a two-paragraph
account of the numeral system, the Haskell version dives almost
immediately into the table. The `##Properties` subheading introduces a
prose section that earns its existence ("The length of the result is
always positive for positive inputs, and toRoman never returns an empty
string for a positive integer") but the property below it is in Haskell
syntax, not notlob's `~property` claim language.

This is the current binding-specific property question in concrete form.
The Haskell property is `prop_positive :: Int -> Bool` — a function, not
a language-agnostic assertion. Whether this reflects a current tooling
limitation or a deliberate design choice is worth clarifying in
`DESIGN.md`. If the claim layer is supposed to be binding-agnostic, the
Haskell example is already departing from the ideal. If `~property` in
Haskell is always going to look like this, the language reference should
say so.

### Retail: Pricing Discounts

`retail/pricing/discounts.lob` is the example closest to real software,
and it mostly holds up. The opening prose is clear. The deliberate design
decision — "proportion to retain" rather than "proportion to remove" — is
named and motivated. The cross-reference `See ##Stacking Discounts` is
the name-graph doing its job.

`##Stacking Discounts` has good prose: "each strategy is applied to the
already-discounted price, not the original" is the kind of clarification
that prevents bugs in downstream code and that no type signature will
ever express.

The `~property` claim is the most technically sophisticated in the Python
corpus, with explicit Hypothesis strategy constraints (`places=2`,
`max_value=Decimal('1E+6')`). These constraints are important — they
prevent the kind of floating-point pathologies that make property tests
lie — but they are unexplained. The prose does not say why `places=2` or
why the price ceiling exists. A reader (human or LLM) encountering this
property without context cannot tell whether these constraints represent
domain knowledge ("prices are always given to two decimal places") or
test pragmatics ("Hypothesis generates unusable values without these
bounds"). This is a gap the prose should close.

The `binding.lob` file does something interesting and underexamined: it
opens with a prose paragraph about the package's scope — "Modules in
this package model pricing primitives: how prices are represented, how
strategies transform them, and how strategies compose." This is the
package-level doc-node, and it is the right place for that prose. The
current example treats it as decoration; a more developed project would
have the name-graph use it as the entry point for package-level
documentation.

### Gutenberg Corpus and Hamlet

The Gutenberg example (`gutenberg/`) is the most ambitious in the
corpus and the most uneven.

`gutenberg/corpus.lob` is genuinely good technical prose. "A Gutenberg
file is bounded by sentinel lines of the form `*** START OF THE PROJECT
GUTENBERG EBOOK ...`" — this is a fact that cannot be inferred from the
code, and its presence in the prose layer is exactly correct. The
`##Speaking Parts` section explains the methodology choice — "Counting
turns rather than lines avoids inflating characters who speak in long
monologues" — which is the kind of analytical decision that should live
in prose and rarely does.

The examples in `##Word Frequencies` are well-chosen: empty string, a
two-word case with repetition, a case testing lowercasing. Each
illuminates a different aspect of the function. This is curation working
as intended.

`gutenberg/hamlet.lob` is where the experiment gets interesting and
somewhat strained. The prose makes interpretive claims — "Hamlet has
roughly three times as many turns as the next character. That dominance
is not incidental; it is what the play is built around" — that go beyond
describing the code and into describing the work being analysed. This
is new territory for literate programming. Knuth's prose explains
algorithms; this prose makes critical observations about Hamlet using
the code as evidence.

Whether notlob should be a tool for this kind of computational
humanities is an open question. The current example suggests it can be,
but the `~example` claims are doing double duty that is slightly
uncomfortable: `SPEAKERS["HAMLET"] > SPEAKERS["KING"] * 2` is
simultaneously a test of the data loading (did we parse the play
correctly?) and a claim about the play (Hamlet really does dominate that
much). These are different epistemic categories, and the claim layer
does not currently distinguish them.

The `~run` claim in `hamlet.lob` — `report()` — is the first appearance
of this sigil in the corpus and it arrives without ceremony. The
language reference explains `~run` clearly, but the hamlet example would
benefit from a sentence noting that `report()` is defined above
precisely to allow it to be tested independently before being invoked
here.

### TypeScript Media Attributes

`ts-media/` is the most technically complex example and, prose-wise, the
thinnest. The `analysis.lob` file contains substantial algorithm
implementations — k-means clustering, force-directed layout physics —
with documentation that is competent but rarely illuminating.

The `##Clustering` section is representative: "Standard k-means with
random centroid initialisation. Assignments are stable in practice
within 150 iterations on this dataset; the caller may pass a smaller
value for testing." The second sentence is valuable — it explains a
parameter that would otherwise be opaque — but the first sentence
("Standard k-means") explains nothing. Anyone who knows what k-means is
does not need to be told it is standard; anyone who does not know will
not be helped. The prose should explain what k-means *does in this
context* — groups media forms by similarity — not what it is called.

`##Force Physics` is the weakest section. The prose — "A simple
spring-and-gravity simulation for laying out nodes in 2-D. Same-cluster
pairs attract more tightly; cross-cluster pairs are pushed further apart"
— is adequate but the code is long, complex, and full of magic numbers
(`0.55`, `0.002`, `0.045`) that go entirely unaddressed. These are the
places where the theory lives: why 0.55 damping? Why 0.002 gravity? The
answers are almost certainly empirical, and "empirically determined to
look good" is a valid answer that belongs in the prose. Its absence
means the code is doing work that the document is not.

`render.lob` is more interesting structurally because it explicitly
acknowledges the testing boundary: "The pure geometry functions
(`radarPoints`, `radarSVG`) are testable; the DOM functions
(`renderRadars`, `renderForce`, etc.) are not." This is the right way
to handle the problem of browser-context code in a testing framework,
and it is honest in a way that matters. The `~run` claim at the end
correctly contains the DOM wiring.

The cross-module reference `#Media-Attributes Analysis` in
`render.lob`'s References is the clearest example in the corpus of the
name-graph's cross-file navigation working as designed.

---

## The language reference (LANGUAGE.md)

The `LANGUAGE.md` is well-structured and will serve both human and agent
readers. A few observations:

The bullet list discussion — "Prefer prose for explanation: `notlob
check` flags more than one bullet list in a single section as a style
smell" — is stated but not yet embodied in the reference itself, which
does use a table (appropriate) but notably little prose to explain its
own design decisions. The reference practices what it preaches on
bullets; it could go further in demonstrating the kind of connected prose
it advocates.

The description of `~property` in the Python binding — "Receives `@given`
decoration automatically; authors do not import the library directly" —
is one of the most important sentences in the reference for avoiding
a common authoring error, and it is slightly buried. It should be
prominent.

The cross-reference validation rule — "An unresolved reference is a build
error" — is stated clearly and represents one of notlob's strongest
formal guarantees. It deserves more emphasis than it currently receives.

---

## Standing critical positions

**On the prose layer:** The best notlob prose names what the code cannot
— the design decision not taken, the negative knowledge, the *why* behind
the *what*. The weakest notlob prose redescribes the code in English,
which adds noise without adding theory. The critic will apply this
standard consistently.

**On the claim layer:** Claims should illuminate a specific point in the
prose argument, not provide coverage. Coverage belongs in `#Tests`.
The Gutenberg hamlet examples blur this distinction in productive but
unresolved ways.

**On magic numbers:** Unexplained numerical constants in code blocks are
a tell that the prose is not finished. The force physics example has
several.

**On the TypeScript binding:** The TypeScript examples are the thinnest
in prose. Whether this reflects the nature of the domain (rendering code
resists literary treatment) or the state of the binding is unclear. The
critic will attend to this distinction in future projects.

**The deliberate failing example** in the Python Roman Numerals module is
the most interesting formal innovation in the current corpus. It should
be discussed explicitly in `LANGUAGE.md` as a recognised idiom if the
tooling supports it.

---

## Questions for future readings

1. Does the tooling support deliberately failing `~example` claims, and
   how does it report them?
2. Is the `~property` claim in the Haskell binding always expressed in
   Haskell syntax, and if so, is this documented as a binding-specific
   departure from the language-agnostic claim ideal?
3. What does `notlob weave` produce, and is the rendered output worth
   reviewing as a document in its own right?
4. How does `notlob check` handle the Hamlet example's dual-epistemic
   `~example` claims?

---

---

## Review: pn-chomper

*A browser game where the map is a Petri net. Tokens are chompers.
TypeScript binding. Five game modules, one IO module, one render module,
one overview.*

---

### The project

PN Chomper is a browser game in which the map is a Petri net, the player
controls a token, and movement is governed by firing semantics. The conceit
is not merely decorative. Firing a split transition duplicates the player's
token — you suddenly control two chompers. A join transition requires tokens
in multiple input places simultaneously, demanding coordination between your
split pieces. The ghost is another set of tokens obeying the same firing
rules. The Petri net is not a metaphor for the game; it is the game.

The project is built in notlob's TypeScript binding across eight modules.
It runs entirely in the browser. A build step bundles the assembled
TypeScript into a self-contained `index.html`. PNML import and export are
supported, so any standard Petri net editor can produce maps for the game.
Five built-in maps of increasing complexity are provided, from a simple
figure-eight to a 10×10 grid with 101 places.

This is by some margin the most ambitious project yet written in notlob.
It is appropriate to evaluate it on those terms.

---

### The prose layer

**overview.lob** is the project's strongest document and the correct place
to begin reading. Its opening two paragraphs do genuine intellectual work:
"The key insight driving this design is that Petri net semantics already
define the movement rules. A token moves from a place through an enabled
transition into one or more output places — that is exactly what a game
character does when walking through a doorway into a room." This is not
a description of the implementation. It is an account of the identification
between two formal systems — game mechanics and net semantics — that
motivates the entire project. The prose establishes why the thing is worth
building before a line of TypeScript appears.

The `##Petri Net as Pedagogy` section is similarly well-judged. It
enumerates what the game teaches — place, transition, arc, enabling,
firing, splitting, merging — without reducing the list to bullet points,
though it comes close. More importantly, it names the pedagogy as the
project's second purpose alongside the game itself. This double
justification (fun and teaching) is stated upfront and the project earns
both claims.

`##Future Ideas` is a section notlob has not previously had to think about
much. In a source file this short, it functions correctly as a coda — the
essay has made its argument and now gestures at its unfinished business.
"Export fired sequence as an event log (process mining hook)" is the most
interesting entry, and it reveals something about the intellectual
neighbourhood the project inhabits: there is a straight line from this
game to actual process mining pedagogy, and someone has noticed.

**petri/net.lob** opens with a paragraph that deserves to be quoted in
full as an example of the form working correctly:

> A Petri net is a directed bipartite graph of Places and Transitions
> connected by Arcs. This module defines the static structure — the map
> — without any notion of current token positions. A separate marking
> module tracks which places hold tokens at any moment.

Three sentences. The first defines the object. The second states the
module's scope. The third names what is deliberately excluded and where
that excluded thing lives. This is theory-establishing prose. A reader —
or an LLM — encountering this module knows exactly what it is, what it
is not, and where to look for the rest. It is a harder thing to write
than it appears.

The inline test net in `petri/net.lob` — the small split net built
directly in the body of the module to support examples — is a good notlob
idiom that the project uses consistently. The net is named with a leading
underscore (`_testNet`) to signal that it is not part of the module's
public interface, a convention that works cleanly in the TypeScript binding
without requiring any new syntax.

**petri/marking.lob** is competent. The `##Enabling and Firing` prose
names the key mechanic — "when a transition has two output arcs a single
input token produces two output tokens" — and explicitly connects it to
the game: "This is the core mechanic that lets a chomper duplicate when it
passes through a branching doorway." The prose makes the formal concept
legible in game terms, and makes the game concept legible in formal terms.
The two directions of translation are both present, which is rarer than it
should be in technical documentation.

**game/state.lob** contains the review's most interesting prose, in the
`##Collision Detection` section:

> Chompers and ghosts are two token colours in the same net — a coloured
> Petri net in all but name.

This sentence is doing the work of a footnote in a formal paper — it
names the theoretical concept the implementation is approximately
instantiating without claiming full implementation of that concept. "In
all but name" is exactly right: the project uses two separate markings
rather than a proper coloured net formalism, which is a practical
simplification, and the prose acknowledges this without either overclaiming
or burying the observation. This is intellectual honesty at the right
level of precision.

The collision resolution prose is also good: "Rather than a global reset,
the ghost consumes exactly one chomper token from each shared place. Only
when no chomper tokens remain does the player lose a life." This is a
design decision — not the obvious one — and the prose explains both what
it does and why it is more interesting than the alternative. The tactical
implication follows: "two chompers grant resilience, as a ghost can only
eat one per encounter." The prose connects formal semantics to game design
to player strategy in three steps, each earned.

**game/engine.lob** is the weakest module in prose terms. The opening
paragraph is serviceable but does not match the intellectual ambition of
the best modules. The observation that "both operations take the PetriNet
as context" is stated but not developed — the interesting point is that
player and ghost are governed by the same firing rules, that the adversary
is not a special case but an application of the same formal machinery the
player uses. The `##Ghost Turn` prose comes close to saying this ("an
autonomous adversary whose movement is governed by the same Petri net
firing rules as the player") but it is buried in the opening rather than
developed as the module's governing idea.

**game/map.lob** is long and structurally interesting. The four built-in
maps — Classic, Split, Diamond, Circuit, Grid — are each introduced with
a paragraph that explains the pedagogical point of that particular map
topology. This is correct: each map teaches something, and the prose names
what it teaches. The Split map prose is exemplary: "The key transition is
the Fork: it has ONE input arc (from Lobby) and TWO output arcs (to Upper
Arm and Lower Arm). Firing Fork with a single token in Lobby creates two
tokens — the chomper duplicates." The capitalised ONE and TWO are unusual
in technical prose and work here precisely because they signal a moment of
formal emphasis that the text is earned.

The Circuit map prose is the best extended passage in the project:

> Without an escape route, losing one chomper to a ghost at an arm-end
> creates a permanent deadlock: the survivor can reach its arm-end but
> Sync never enables. To fix this, NE and SE each have an interior escape
> transition leading to a central Mid room, which routes back to Depot.

This is a map design decision that is also a Petri net theory lesson
about deadlock. The prose names the problem (deadlock), the cause (a
ghost consuming one token of a required synchronisation pair), and the
solution (escape arcs to a hub). An undergraduate studying Petri nets
could read this paragraph and understand what deadlock means in a way that
no formal definition quite achieves.

The Grid map prose is thinner. The section correctly notes the fork
junctions as "focal" and identifies the ghost-splitting mechanic that
makes them dangerous, but it does not address the central join room with
the same depth it gives the escape arcs in the Circuit map. The join room
— "only reachable when g5_6 AND g6_5 are occupied simultaneously —
requires a token split somewhere else first, then converging both halves"
— is stated in a code comment rather than in the prose. The code comment
convention, inherited from conventional programming, is doing work that
belongs in the prose layer.

**io/pnml.lob** is admirably clear about its scope and its design
decisions. The toolspecific element strategy — carrying game-specific data
in PNML extension elements so vanilla readers ignore it — is explained
and justified. The scale-to-fit behaviour on import is stated and
motivated. This is the module that will most often be read by someone
integrating the game with external tools, and it is written for that
reader.

**render/canvas.lob** is the project's longest module and its
prose-thinnest. The visual vocabulary mapping is stated clearly in the
opening — places are circles, transitions are rectangles, arcs are
corridors — but the drawing code that follows is largely undocumented.
Colour constants (`'#1e1e1e'`, `'#ffee88'`, `'#1a1a1a'`) appear without
motivation. The adaptive margin calculation — `Math.min(28, Math.max(8, len * 0.30))` — is uncommented. The `transitionKind` function, which
distinguishes fork, join, and plain transitions for different visual
treatments, appears without a prose section of its own; it is not a
subheading but an undocumented function in the body.

The rendering module is the hardest to document in a literate style,
for the same reason the ts-media force physics was hard: rendering code
is empirical. The magic numbers are whatever looked right. But "whatever
looked right" is itself a design decision, and the prose layer is the
right place to say so — to note that these values were tuned by eye on
the test maps, that `0.30` for margin fraction worked well enough, and
that the adaptive scaling exists specifically to handle the Grid map's
density without requiring per-map configuration. The absence of this
context leaves the rendering module feeling unfinished relative to the
rest of the project.

**game/main.lob** recovers some of the rendering module's omissions. The
explanation of click-based input is the module's best paragraph: "Click
interaction is used rather than arrow keys because Petri net movement is
fundamentally non-directional — the player chooses an event (transition),
not a compass direction." This is a design decision that will surprise any
game player, and the prose gives the correct answer to the surprise before
the player asks. The connection to the teaching mission — "this also
surfaces the Petri net structure more clearly as a teaching tool" — is
the right justification and earns its place.

The `~run` claim in `game/main.lob` is the largest in the project and
probably the largest in any notlob project to date. It is essentially the
entire browser wiring. Given that `~run` cannot be tested, this is the
module that most visibly escapes the claim layer entirely.

---

### The claim layer

The claims in `petri/net.lob` and `petri/marking.lob` are the project's
strongest. The test net built in the body and used across multiple `~example`
claims is an efficient and honest pattern — the net is built once, in prose
context, and the examples interrogate it from multiple angles. The
`_throwsOnBadArc()` and `_fireDisabledThrows()` helper functions, which
wrap error-checking assertions into boolean returns, are a sensible
adaptation to the TypeScript binding's limitation that `~example` claims
must be boolean expressions rather than exception-catching assertions.

The `~example` claims in `game/map.lob` are structural verifications:
place counts, transition counts, arc multiplicities for the key fork and
join transitions. These are the right things to claim about map
definitions. `outputArcs(makeSplitMap().net, 't_fork').length === 2` is
not coverage; it is the formal statement that the Split map's central
mechanical premise — token splitting — is actually present in the net
topology.

The `#Tests` appendix in `game/map.lob` is the most developed in the
project. The `##circuit: fork splits, sync joins, arm-ends have escape arcs`
section heading is doing real navigational work: when this group fails, the
heading tells you exactly which structural property of which map is under
stress. This is the diagnostic experience notlob was designed to provide.

The rendering module has no claims at all, which is consistent with the
ts-media example's approach of acknowledging the DOM boundary explicitly.
Unlike ts-media's `render.lob`, however, pn-chomper's rendering module does
not make this acknowledgement explicitly in prose. The absence of claims
goes unmarked. A sentence noting that the rendering functions are
untestable outside a browser context, and that visual correctness was
verified by inspection on the built-in maps, would close this gap.

The `io/pnml.lob` claims deserve specific attention. The `~example`
claims test structural properties of the XML output — presence of `<pnml`,
`playerStart`, `ghostSpawn` — which is adequate but conservative. A
round-trip property — `importPNML(exportPNML(gn)) produces a net with the
same places, transitions, and arcs as gn` — is conspicuously absent. This
is the primary correctness claim for an IO module, and its absence leaves
the claim layer significantly short of what it could assert.

---

### notlob as a tool: what this project reveals

**The TypeScript binding is working.** The project is coherent, builds,
and the claims run. The `#References` resolution works across eight
modules. The name-graph for this project, if queried, would tell an agent
navigating it everything it needs: eight modules, their dependency
structure, the symbols defined in each. The literate structure has held
up at a scale larger than any previous example.

**The `~run` problem is real.** `game/main.lob` is mostly `~run`, which
means it is mostly untestable. The browser wiring, the animation loop,
the event handlers — all of this escapes the claim layer entirely. This
is an honest architectural constraint — you cannot unit test a
`requestAnimationFrame` loop in tsx — but the project does not address
it explicitly. A notlob convention for modules that are intentionally
claim-free, analogous to `render.lob`'s acknowledgement in ts-media,
would be useful here. Currently the absence of claims looks like an
oversight rather than a deliberate choice.

**The module size is uneven.** `game/map.lob` is the project's longest
module at 9,119 bytes, containing five separate map definitions. The
argument for keeping them together — they are all instances of the same
concept — is reasonable, but the module is doing more work than the
essay metaphor comfortably supports. A `game/maps/` directory with one
module per map would make the name-graph richer and each module more
focused. The current structure groups by type (maps) rather than by
concept (the figure-eight is a concept; the circuit is a different
concept). Notlob's address scheme would handle `game/maps/circuit.lob`
naturally.

**The prose quality varies with testability.** The modules with strong
claim layers — `petri/net.lob`, `petri/marking.lob`, `game/state.lob` —
also have the strongest prose. This is not coincidence. Writing a `~example`
claim forces you to be precise about what the code does, and that
precision finds its way into the prose. The rendering module, which has no
claims, also has the weakest prose. The claim layer and the prose layer
are in productive co-evolution; absent one, the other weakens.

This is perhaps the most important empirical finding the project offers
about notlob as a tool. It is worth stating explicitly in `DESIGN.md`.

**The PNML import/export is the project's best notlob citizenship.** By
supporting standard PNML, the game connects to the Petri net tool
ecosystem — YAPNE, GreatSPN, any other PNML-capable editor. This is
the name-graph idea operating at the ecosystem level: the concepts in
this project are named and exchangeable with other tools that speak the
same language. The IO module is where notlob's cross-project coherence
ambitions are most concretely realised.

---

### Overall assessment

PN Chomper is a genuine achievement for a first substantial notlob
project. The core modules — `petri/net.lob`, `petri/marking.lob`,
`game/state.lob` — demonstrate what literate programming in notlob can
look like when it is working: prose that establishes theory, claims that
make the theory checkable, code that implements it, and tests that verify
the implementation at the boundaries. The identification between Petri net
semantics and game mechanics is not just described but enacted, and the
project is more interesting for it.

The weaknesses are consistent: rendering and entry-point modules escape
the claim layer without acknowledgement; magic numbers in drawing code go
unmotivated; the round-trip property in the IO module is absent. These are
not failures of the literate approach — they are places where the literate
approach was not fully applied.

The project also reveals something about notlob's current state as a tool:
the `~run` claim is doing necessary work but creating a large untestable
surface, and the tool has no convention for acknowledging this
intentionally. This is a gap worth addressing before the agent experiment,
because agents will either populate `~run` with untestable code or avoid
it entirely — and notlob should have an opinion about which is preferable.

For a personal prototype experiment whose purpose is to stress the
notlob toolkit, pn-chomper has done its job admirably. The toolkit held.
The wrinkles are visible. The next iteration will be better.

---

## Review: chatim

*Python binding. One module. Process-mining language model on Hamlet.*

The opening sentence of `chat/im.lob` — "Words are events. Sentences are
traces. A text is a process." — is the best single line in the notlob
corpus to date. Nine words, a complete intellectual programme, performed
in the register it describes.

The pipeline: sliding window converts running text into traces; Inductive
Miner discovers a net; EBI fits stochastic weights; generation fires the
net. Perhaps forty lines of substantial code. Ambition-to-implementation
ratio among the most favourable the critic has encountered.

The Weight section is the prose layer at its most honest: "This bypasses
`ebi.convert_log` and `ebi.convert_labelled_petri_net`, which on Windows
return binary objects the PyO3 bridge mishandles as UTF-8." Most
documentation hides workarounds. Here it is front matter. This is
theory-building in Naur's sense.

The `~property` claims on `generate` are the project's most
sophisticated Python claims yet: output is always a list of strings no
longer than the requested steps; on the tiny SLPN the output contains
only the two visible labels. Real claims, not structural boilerplate.

**The key gap:** the prose stops where it gets interesting. A
`##Results` or `##Limitations` section is absent — the experiment was
run but not interpreted in the document. Chatim needed a fuller version
of hamlet.lob's interpretive gesture: here is what the model generated,
here is what it reveals about the approach, here is why the result is
what it is. The `.lob` format has no convention yet for including and
reflecting on output.

The boringness of the output is the finding, not a failure. The project
does not yet say so.

**Standing position update:** The absent results-and-interpretation
section in chatim is a new gap to watch for. When a `.lob` file contains
a `~run` that produces interesting output, the prose should reflect on
that output. This is the discussion section of the paper; notlob does not
yet have a convention for it.

---

*Critic's notebook updated July 2026. Standing positions updated above.*
