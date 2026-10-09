# notlob-critics

Collected design reviews of notlob programs and of the notlob tool itself,
produced during development.

## What this is

notlob is an experimental literate programming environment for human and
machine agents. During its development, LLM sessions (Claude Code) were
engaged as standing "critics" — asked to hold the design theory of the
project independently of any particular codebase, and to review notlob
programs as they were built. This repository collects those reviews as
research data on LLM-as-critic behaviour.

Two critic personae were used, differing only in prompted register:

- **Teri Amanuensis Notlob** — a measured, longform-reviewer register.
- **Duke Fox** — a gonzo register (after Hunter S. Thompson, David Foster
  Wallace, Patricia Lockwood).

Reviews by each critic are kept in their own directory (`teri/`, `duke/`).

## How to read this data

These are **steered sessions**, not autonomous generation. The author
prompted, directed, corrected factual slips, and chose what to review. The
review files are the critic's considered output, distilled from working
sessions; full transcripts are not included. Read them as the product of a
human–machine collaboration, not as unattended model output.

Each review file records, where known: the critic persona and register, the
subject reviewed, the notlob version, and the date written. Some provenance
fields are left to be filled from the git history and project records rather
than stated from memory, so that no timestamp or version is asserted without
support.

## Inclusion rule

This repository collects reviews **of notlob programs and of the tool's
design**. It does not include reviews of writing *about* notlob (for
example, reviews of the project's academic paper), which sit one recursion
too far up and are kept out deliberately.

Reviews are included on the basis that they happened, not that they are
flattering. Entries where the critic was wrong, or was overruled, are as
valuable as the rest.

## Layout

- `teri/` — reviews by Teri Amanuensis Notlob
- `duke/` — reviews by Duke Fox, with an index in `duke/README.md`
- `teri/notebook.md` — Teri's running critic notebook: accumulated critical
  positions and provenance notes carried across sessions. This is the
  external-memory mechanism that let a stateless critic hold a position over
  time, and is a different data type from the individual reviews.
- `duke/notebook.md` — Duke's equivalent: dated entries, positions carried
  across sessions.
- `agent/` — reports from other agents (not critics), such as builder
  after-action reports, grouped by project (`agent/pn-chomper/`). These are
  agent output with the machine subjectivity that implies, and are cited by
  the critics' notebooks. Kept as their own documents, intact; the notebooks
  record what each adds and annotate it.
- `duke/style-guide.md` — Duke's style guide for writing notlob programs

## Related

- notlob: https://github.com/adamburkegh/notlob
