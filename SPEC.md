# RIF — Research Interchange Format v0.2 (compact specification)

*A git-native format in which the repository is the research and the paper is a view.*

## 1. Design principles

1. **Claims first.** The atomic unit is a falsifiable claim with a stable, citable ID
   (`rif://<repo>/C4`), not a section.
2. **Executable evidence.** Every empirical claim points to a re-runnable experiment
   (command, environment, data); every formal claim to a definition/proof. Every value
   carries a provenance grade: `verified` (re-executed) / `as-reported` (stated in the
   source) / `claimed` (unverified estimate).
3. **Narrative is data.** Intuitions and motivating examples are author-written blocks
   tagged by claim and audience; the renderer selects and expands them, never invents them.
4. **Rendering is a function.** `view = render(graph, profile, query)`. The reader
   profile is an object of the format.
5. **Honesty by construction.** Non-claims are explicit; watchpoints survive in every view.
6. **Non-generable author content.** Insights, style, and original figures may be
   compressed or translated by the renderer, never fabricated.

## 2. Repository layout

```
paper.rif.yaml        # manifest: identity, encoding origin, artifacts, profiles
claims.yaml           # typed claim graph (nodes + typed edges)
formal.yaml           # definitions, equations, theorems
experiments.yaml      # protocols + results + replication commands
insights.yaml         # author voice: contributions, reading, watchpoints, outlook
style.yaml            # style corpus (author's prior papers) + register + lexicon
figures/              # original figures (verbatim assets) + figures.yaml manifest
references.yaml       # related work as resolvable nodes (rif:// | doi | arxiv | bibtex)
datasets.yaml         # first-class dataset nodes, resolved to existing repositories
narrative/*.md        # prose blocks tagged by claim and audience
profiles/*.yaml       # predefined reader profiles
```

(Small repositories may inline all sections in `paper.rif.yaml`, as the working
example here does.)

## 3. Claims

```yaml
- id: C4                      # stable, citable: rif://<repo>/C4
  type: empirical             # empirical | formal | methodological | positioning
  statement: "..."            # one falsifiable sentence
  audience_floor: student     # below this level: replaced by narrative paraphrase
  evidence: [E1]              # -> experiments.yaml / formal.yaml
  depends_on: [C2]            # typed edges: depends_on | extends | contrasts | uses
  non_claims: ["..."]         # what this claim does NOT assert
```

Edges may point outside the repository (DOI or another RIF repo's claim): related
work becomes a typed graph merge between repositories.

## 4. Evidence

```yaml
- id: E1
  kind: experiment            # experiment | proof | user-study | external
  protocol: "..."
  datasets: [{name, uri, license}]
  command: "..."              # one-shot replication
  environment: "docker://..."
  results:
    - { metric: accuracy, value: 92.5, provenance: as-reported }
```

## 5. Insights (the author's voice; non-generable)

```yaml
insights:
  contributions: [{ id: K1, statement: "...", claims: [C2] }]
  reading:       [{ id: R1, about: [E1], text: "..." }]
  watchpoints:   [{ id: W1, about: [C4], severity: caution, text: "..." }]
                 # severity: caution | limitation | open-problem
  outlook: "..."
```

**Watchpoint invariance**: a rendered view that omits the watchpoints (at most
paraphrased to the reader's level) is invalid.

## 6. Encoding origin

```yaml
encoding:
  by: author | third-party
  source: "doi/arXiv of the text this derives from"   # for third-party
```

With `by: third-party`, insights carry `provenance: derived`: they are inferred from
the published text, not authorial voice, and an author-produced encoding replaces them.

## 7. Style

```yaml
style:
  corpus: [{ title: "...", doi: "...", weight: 1.0 }]  # author's prior papers
  register: { person: we, stance: "..." }
  lexicon: { prefer: [...], avoid: [...] }
```

Style modulates form only; the claim graph remains the content perimeter.

## 8. Figures

```yaml
figures:
  - { id: G1, kind: original, uri: figures/fig1.pdf,   # author-made: used VERBATIM
      anchors: [C2], audience: [student, expert],
      captions: { expert: "...", student: "..." } }    # per-audience author captions
  - { id: G2, kind: generable, from: E1 }              # renderer may draw it, must label it
```

If an `original` figure anchors a claim present in the view, it takes precedence over
any generated visual; generated visuals are always labelled as renderer-generated.

## 9. References: `references.yaml` (resolve, don't host)

Related work is a set of typed, resolvable nodes - not a prose section:

```yaml
references:
  - id: R2
    resolve: { arxiv: "1606.01781" }   # levels: rif:// (claim-level, best) |
    label: "Conneau et al. (2017), VDCNN, EACL"   #   doi/arxiv/openalex | bibtex (fallback)
    role: baseline        # baseline | method-source | dataset-source | positioning
    anchored_by: [C4, E1]
```

The `rif://repo/C3` level anchors a *claim* of another RIF repository: related work
becomes a typed graph merge. **The bibliography of a view is a derived artifact**:
the renderer generates it from the reference nodes actually anchored in that view,
in the citation style of the target venue.

## 10. Datasets: `datasets.yaml` (resolve, don't host)

Datasets are first-class nodes that evidence items point to:

```yaml
datasets:
  - id: D3
    name: "DBpedia ontology classification"
    resolve: { hf: "dbpedia_14" }      # hf | doi (DataCite/Zenodo) | openml | uri
    version: "..."                     # revision/commit
    checksum: "sha256:..."
    license: "CC-BY-SA"
    access: open                       # open | gated | private
    role: eval                         # train | eval | source
```

Version + checksum are the stable identity that re-execution needs to promote
`as-reported` results to `verified`; without them provenance grades cannot be
audited.

## 11. Reader profiles and rendering rules

```yaml
profile:
  expertise: layperson | student | practitioner | expert
  goal: evaluate | reproduce | apply | learn | cite | survey
  time_budget: 3min | 15min | 60min
  language: en | ...
  formalism: full | light | none
```

- `expertise` sets the claim floor; `goal` sets selection and ordering;
  `time_budget` sets expansion depth; `language`/`formalism` set surface form.
- **Anchoring rule**: every generated sentence must be anchored to a node ID.
  A sentence that cannot be anchored cannot be generated.
