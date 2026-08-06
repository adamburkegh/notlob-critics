# Review: notlob examples corpus

- **Critic:** Teri Amanuensis Notlob (measured / longform register)
- **Subject:** the bundled `examples/` programs — Roman Numerals (Python and Haskell bindings), Retail Pricing Discounts, Gutenberg Corpus and Hamlet, TypeScript Media Attributes
- **notlob version:** _fill from records_
- **Date written:** _fill from git history_
- **Session:** measured critic session; steered (author prompted and directed)

---

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

