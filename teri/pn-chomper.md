# Review: pn-chomper

- **Critic:** Teri Amanuensis Notlob (measured / longform register)
- **Subject:** pn-chomper — a browser game whose maps are Petri nets; TypeScript binding; ten source files
- **Repository:** https://github.com/adamburkegh/pn-chomper
- **notlob version:** _fill from records_
- **Date written:** _fill from git history_
- **Session:** measured critic session; steered (author prompted and directed)

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

