# RIF — Research Interchange Format (anonymous review artifact)

Supplementary artifact for the EACL 2027 submission *"RIF: A Research Interchange Format for Scientific Research in the Age of AI-Assisted Science"*.

## Contents

| Path | Content |
|---|---|
| `SPEC.md` | RIF v0.2 format specification (compact) |
| `paper.rif.yaml` | **Complete working example**: *Bag of Tricks for Efficient Text Classification* (fastText, EACL 2017) encoded as a third-party RIF repository — claim graph, provenance-graded evidence, derived insights, style corpus, figures, reader profiles |
| `views/paper-en.pdf` | **Generated venue-paper view**: a complete, readable IMRaD paper (abstract, related work, model, experiments, discussion) generated from the repository in the authors' lexicon, with every sentence anchored to a claim node (superscripts). `views/paper-en.tex` is the renderer output source. |
| `claims.yaml` · `experiments.yaml` · `insights.yaml` · `style.yaml` · `figures.yaml` · `references.yaml` · `datasets.yaml` · `profiles.yaml` | The full RIF repository, one file per section (`paper.rif.yaml` is the manifest). References and datasets follow a *resolve, don't host* principle: resolvable identities (DOI, arXiv, Hugging Face, `rif://`) instead of hosted copies; the bibliography of a view is derived from the anchors it uses. |
| `demo/index.html` | Interactive demo: the same repository rendered for four profiles (reviewer / practitioner / student / full venue-paper view), with claim-level traceability and the full `.rif` source browsable in-page. Self-contained: download and open in any browser. |

## The idea in one line

```
RIF repo (claims · evidence · insights · style · figures)
        │
        ▼   view = render(graph, profile, query)
venue paper │ practitioner note │ student view │ agent answer
```

Three rules govern rendering: **anchoring** (a sentence that cannot be anchored to a
node cannot be generated), **watchpoint invariance** (author-declared watchpoints
survive in every view), and **floors** (claims below the reader's expertise are
replaced by their author-written paraphrase).

This is a third-party encoding (`encoding.by: third-party`): insights carry
`provenance: derived` and do not constitute the original authors' voice.
