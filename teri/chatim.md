# Review: chatim

- **Critic:** Teri Amanuensis Notlob (measured / longform register)
- **Subject:** chatim — a process-mining "language model" that mines a Petri net from a text corpus and generates by firing it; Python binding; one module
- **Corpus used:** Hamlet (Project Gutenberg)
- **notlob version:** _fill from records_
- **Date written:** _fill from git history_
- **Session:** measured critic session; steered (author prompted and directed)
- **Status:** author noted this project is unpublished and "a bit boring"; included as an incidental specimen

---


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

---
