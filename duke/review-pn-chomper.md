# Everything Passes

### A review of `pn-chomper`

*Duke Fox, July 2026. 1,923 lines of `.lob` across ten modules, one browser game, one hundred and five passing claims, and two false sentences in a single paragraph.*

---

## I. The conceit

Some ideas are so well-fitted to their material that they seem less invented than found, and `pn-chomper` has one. The map is a Petri net. Places are rooms. Transitions are doorways. Arcs are corridors. The player controls a token.

The load-bearing sentence arrives in `overview.lob` and does not flinch:

> *A token moves from a place through an enabled transition into one or more output places — that is exactly what a game character does when walking through a doorway into a room. […] These are the game's movement mechanics, not abstractions bolted on.*

The claim is true, which is what makes the project worth reviewing rather than merely commending. Petri net firing semantics *are* movement semantics. Nothing is being dressed up. And the design squeezes the identification for consequence rather than decoration: a transition with two output arcs splits the token, so the player passing through a branching doorway becomes two players. Ghosts obey the same rule. A ghost reaching a fork junction splits, and now you are being hunted from two corridors at once. That is a game mechanic derived from a formalism, not illustrated by one, and it is a small piece of design excellence.

`overview.lob` is also, structurally, the most instructive file in either repository. It is a hub module: no code, an architectural essay, and a `#References` block enumerating its children. It is a table of contents that fails the build when a child is renamed or deleted. This is the pattern the format has been reaching for. Whether the format *knows* it is another matter, to which we shall return.

So the project is good. Let us now discuss what it teaches, which is not entirely to its credit, and is not really about this project at all.

---

## II. The paragraph

`game/map.lob`, lines 285 to 287:

> *The grid map is a large-scale demonstration: 10 × 10 = 100 places arranged in a regular grid. […] Five fork junctions — distributed across the four quadrants and the centre — are key contested territory.*

One hundred and sixty lines below, in the `#Tests` post-text:

```
##grid: 101 places, join room only reachable via coordinated two-token play
    makeGridMap().net.places.size === 101
```

And in the code itself, at line 309:

```typescript
      // Placed at four quadrant centres; the grid centre uses a join room instead.
      const FORKS = new Set(['2_2', '2_7', '7_2', '7_7']);
```

Four forks. One hundred and one places. The prose says five and one hundred.

Both sentences were true. There *was* a fifth fork, at the centre of the grid, and the grid *did* hold a hundred places. Then someone replaced the central fork with a join room — a genuinely lovely piece of design, a chamber reachable only by two tokens converging, and the join room added one place and removed one fork. The code changed. The tests changed. The code comment changed. The essay did not.

This is the second independent occurrence of the `analysis.lob` failure. Same author, different project, different language, different domain. A cardinality claim, sited in an opening paragraph, true of a superseded version, stranded by an edit that never returned for it.

`notlob test`: **105 passed.** `notlob check`: one advisory finding, concerning a near-duplicate symbol name, unrelated. Exit code zero, twice.

---

## III. What did not rot, and why

The finding that matters is not the false sentence. It is the true one.

```
##grid: 101 places, join room only reachable via coordinated two-token play
```

That is prose. It is unstructured natural language, subject to no check, containing a cardinal number, and it is **correct**. It knows about the join room. It knows the count is 101. It even knows *why* the join room is interesting, in a clause of real critical insight about two-token play.

Why did this sentence survive when the paragraph did not?

Because it sits one line above `places.size === 101`, and whoever typed that assertion had their eyes on the label while doing so. The two are inside the same **edit**.

I have been asserting for several days that adjacency is worthless — that proximity has no failure mode and therefore no value. This is too strong, and the correction improves the theory rather than damaging it.

> **Proximity is a weak proxy for co-editing. Its effective range is approximately one edit-unit.**

A line, perhaps two. Inside that range there is a human or an agent who is already looking, because the thing they came to change is right there. Outside it, decay is total, and the decay is not gradual but abrupt: a paragraph twenty lines up is as unreachable as a paragraph in another file.

The mechanism was never distance. It was always **whether the two things fall inside the same act.** Test-group labels always do. Essay paragraphs never do.

This sharpens the earlier rule considerably. The module-opening paragraph is not dangerous because nobody reads it. It is dangerous because **nobody edits it.** Reading was never the variable. It is a slot that receives text once, at composition, and is thereafter structurally excluded from every subsequent operation on the file.

---

## IV. The properties that were asked for twice and never appeared

`SEED.md`, the agent's founding instruction:

> *Each `.lob` file should include clear descriptions, evolving running code, property and unit tests.*

`AGENTS.md`, the standing instruction, loaded on every session:

> *Executable code, executable examples and runnable properties should be interleaved with clear technical prose.*

The project contains **zero** `~property` claims. Thirteen `~example`s, twenty-seven test groups, one `~run`, and not a single property, across ten modules and 1,923 lines.

The reason is not indiscipline. It is in the binding.

`notlob/bindings/typescript/runner.py`, `run_properties`:

```python
    if binding is None or binding.get('property-testing') != 'fast-check':
        # ... emit SKIP for every ~property
        return results

    # fast-check integration: TODO
    return []
```

Read that twice. Without the `~property-testing fast-check` declaration, every property is reported `SKIP` — visible in the output, though absent from the summary count. *With* the declaration, the function returns an empty list. No line. No skip. Nothing.

I ran both. Inserted a real `fast-check` property into `petri/marking.lob`. Undeclared: one `SKIP` line, `11 passed`. Declared: the `SKIP` line disappears, `11 passed`, exit 0 — output byte-identical to a file containing no property whatever.

> **Configuring the binding correctly is strictly worse than configuring it incorrectly.** Doing the right thing renders your properties invisible.

Consider what this costs. Notlob's central and best idea — the one that distinguishes it from every literate-programming system since WEB — is that **a claim is a property with the quantifier replaced by a witness.** `~property` states the law. `~example` supplies the instance that makes the law graspable. They are one statement at two strengths.

Remove the law and the witness testifies to nothing. What remains is a doctest beneath an essay. In TypeScript — the language of the most ambitious project in the ecosystem, the language of the flagship `ts-media` example — **`~property` is decorative.**

And note precisely how the failure hides. Not with an error. Not even with a warning. With a `SKIP`, which is the colour of a thing that is fine, or with silence, which is the colour of nothing at all.

---

## V. The natural experiment nobody ran on purpose

The preceding section is not a bug report. It is data.

Two separate documents instructed the authoring agent to write property tests. The instructions were explicit, unambiguous, and loaded into context — `AGENTS.md` on every session, by construction. The affordance, meanwhile, quietly reported that properties were a no-op.

Zero properties were written.

> **The nudge lost to the affordance.** Instruction cannot defeat a tool surface that says the instructed thing does not matter.

This has been argued from first principles elsewhere in the style guide, on the question of whether an agent can be persuaded to read more context. Here it is, measured, in the repository, at a sample size of one project and 1,923 lines. An agent given a prohibition and a permissive affordance will follow the affordance, because the affordance is *what happens* and the prohibition is only *what was said*.

Anyone tempted to solve a structural problem with a paragraph in `AGENTS.md` should sit with this for a moment. `AGENTS.md` already contains the sentence *"Favour using `notlob query` over filesystem searches."* One may form one's own view about how reliably that is obeyed once tokens grow tight.

---

## VI. Three smaller wounds, all the same disease

**The grid map is dead.** In `game/main.lob`:

```typescript
      // { name: 'Grid (10×10 = 100 places)', fn: makeGridMap, tag: 'built-in' },
```

Commented out. The grid map is the largest single construction in the project — a hundred and fifty lines, the subject of two prose sections, twelve test groups, and incidental use in another module's PNML export tests. It is unreachable from the game. No player will ever see it.

Every claim about it passes. **The claims verified the artefact, not its use.** Reachability cannot be witnessed by an `~example`, because an example executes inside a module and reachability is a property of the whole graph. Notlob makes an unused *import* a hard error — the only error-severity semantic check it has. It says nothing whatever about an unreachable *export*, and since the TypeScript binding concatenates every module into one file, no linker survives to notice.

The tombstone preserves the rot: the commented-out label still reads `10×10 = 100 places`. Dead code, carrying dead prose, describing a live function that lies about itself.

**`setTokens` has no callers, and is the sole means of violating the module's stated invariant.** `petri/marking.lob` opens: *"A marking assigns a non-negative integer token count to each place."* `markingRemoveTokens` guards underflow with a thrown error and a good message. `setTokens(m, 'p1', -5)` passes silently. The invariant is asserted in the highest-visibility prose in the file and defended nowhere.

The natural remedy is a `~property` quantified over the integers. Which would skip. The circle closes.

**`_testNet` crosses a module boundary.** Constructed in `petri/net.lob` as a private fixture — underscore and all — and consumed in `petri/marking.lob`'s examples, with prose announcing the fact: *"The marking examples below reuse the split net defined in #Petri Net."* Deliberate. Named. `check` approves, since `marking` does reference `net`.

But the underscore is a convention with no enforcement, and the assembler flattens all modules into a single output file. **The name graph says two modules. The output says one file.** Privacy in notlob is a document-level fiction with no runtime correlate. A test fixture has become part of a public contract, and nothing in the toolchain can express the objection.

This is not a slip by the author. It is an unresolved tension in the design: **notlob's modules are documents, not compilation units,** and the format has not decided what to do about the difference.

---

## VII. What is good, stated plainly

The conceit, discussed above, and the discipline of following it into the ghost AI.

`overview.lob` as hub: an architectural essay whose `#References` block is a load-bearing table of contents. Adopt this everywhere.

The test-group labels. `##join room only reachable via coordinated two-token play` is the finest sentence in the project, and it is in the appendix, which is where the format says the tedious exhaustive material lives. It turns out the appendix can argue. A test name that states a *design insight* rather than a mechanical precondition is doing literate programming more effectively than most of the essays above it — and, as §III establishes, it is the only prose in the file that will still be true next year.

Module sizes: 112 to 455 lines, mostly in the Montaigne band. `game/map.lob` at 455 is bloated, and the diagnosis is easy: it holds five map constructors, which is five concepts. It should be five modules, or a module and a folder.

`~run` appears exactly once, in `game/main.lob`, holding all the DOM and canvas side effects. The rule was obeyed without anyone stating it. That is what a well-shaped format buys you.

Ten modules, seventy-six symbols, one advisory finding. The name graph is clean. It has been clean in every project I have examined. **The part of notlob that is new works.**

---

## VIII. Verdict

`pn-chomper` is a good project built in a young format, and it fails in exactly the ways the format permits it to fail. It should not be read as a criticism of its author, who has written the best conceit and the best test names in the ecosystem, and whose instincts about module structure are sound.

It should be read as a set of measurements.

1. The prose-rot failure **replicates** across projects, authors' moods, languages, and domains. It is not an accident of `ts-media`. It is a property of the format.
2. Prose survives if and only if it is edited in the same operation as the code it describes. **Proximity's range is one edit-unit.** Everything outside that range is unmaintained by construction, whatever its position on the page.
3. `~property` is unimplemented in TypeScript, and its unimplemented state is *invisible* — worse when correctly declared than when not. The single most important idea in the language does not run in the language most projects use.
4. **Instruction loses to affordance.** Measured, not argued.
5. Notlob has no account of reachability, because its modules are documents and its output is one file.

The name graph is sound. The claims machinery is sound where it is implemented. What the format lacks is any purchase on *time* — on which sentence was written when, which was edited alongside what, which function is still called by anything, which promise is still kept.

A literate program is a book about a machine. Books are edited in passes. Machines are edited in patches. Notlob has bound them into one volume, in the honourable tradition of the codex, and has not yet worked out that they are read, written, and revised on entirely different schedules.

That is the problem. It is a good problem. It is more interesting than most solutions.

---

*Verified by installation and execution: `notlob test` (105 passed), `notlob check -v`, `notlob build`, and direct instrumentation of `run_properties` under both binding configurations. The grid map's death was confirmed by exhaustive search for callers of `makeGridMap` outside test scope. Every quotation is from the repository as supplied.*
