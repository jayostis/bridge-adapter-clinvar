# cascade-bridge-adapter-clinvar — Agent Context

The Cascade Bridge Adapter for ClinVar VCV XML: a package of data a **Cascade
Bridge** runs to turn ClinVar records into Cascade RDF. Import-only. Phase 1
(version 0.2.0) has no mapping and nothing that executes.

The contract is the **Cascade Bridge Specification**, [cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec),
pinned by the crate's `bridge:specPin`. Dependencies point one way: an adapter
knows about the specification, and the specification knows nothing about any
adapter. Phases and the decisions behind the layout are in [issue #1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1),
the layout itself in the README. Do not re-derive them.

## The rules

- **No code.** Markdown, XML, XSD, XSLT, YAML, JSON, JSON-LD, CSV, Turtle,
  SPARQL. Nothing that executes, in any directory. An adapter is data; the thing
  that runs it is a Bridge. If something seems to need code, it is a finding for
  the specification. The lint measures this rather than taking it on trust.
- **The manifest is the crate.** `ro-crate-metadata.json` is both the adapter's
  manifest and the provenance record for every file and dataset. There is no
  `adapter.yaml`; the RFC's section 6 sketch names one, and the departure is a
  finding for the specification, not a reason to add one back.
- **No tests here.** Fixtures and how to judge them are declared as data in
  `fixtures/manifest.ttl`; a Bridge's harness executes them, and no Bridge exists
  yet. Do not add a test runner or a workflow that runs fixtures.
  `.github/workflows/validate.yml` is a lint, not a fixture run; its own
  `CLAUDE.md` says why that is no contradiction.
- **No copy of the specification.** The `bridge:` vocabulary and its SHACL shapes
  live in `cascade-bridge-spec`. To change one, change it there, tag it, and move
  `bridge:specPin` and the workflow's ref here in the same commit. The two name
  one commit; the lint fails the run when they differ.
- **No Cascade terms are minted here.** A value with no Cascade term goes in the
  adapter's own namespace (`vocab/`, phase 2) or in the findings sidecar. Terms
  in `cascade:` and `genomics:` go through spec's RFC process.
- **Identity is not a blocker.** How a record's IRI is minted is not settled and
  is decided in phase 2. Nothing in the layout depends on the answer, so do not
  reopen it to make progress on anything else, and do not settle it from a
  discussion thread — ask.

## Sibling checkouts

Expected as sister directories. Which commit each is pinned at is in the crate,
not here.

| | |
|---|---|
| `../cascade-bridge-spec` | the contract: vocabulary, shapes, profile, and the lint CI calls |
| `../spec` | the Cascade vocabularies this adapter writes to |
| `../conformance` | where the four oracle triplets came from |
| `../cascade-cli` | the converter this adapter re-expresses as data |
| `../sdk-typescript` | runtime lineage; not read by anything here yet |

## Before pushing

CI calls the specification's lint at the pinned tag; `docs/adapter/validation.md` there
says what it checks. SHACL cannot see the filesystem, so by hand: every crate
`File` exists and every `sha256` matches, then `fixtures/CLAUDE.md` and
`schema/CLAUDE.md`. A commit says which ran.

## Where a rule goes

This file holds what has to be known *before* choosing a directory to open.
Everything else belongs in a `CLAUDE.md` in the directory it governs, which loads
only when that directory is touched. Keep this file under 80 lines.

## Conventions

- Conventional commits: `feat(adapter): ...`, `docs: ...`, `fix(fixtures): ...`.
- Impersonal: findings and decisions, not promises by a person.
- **Say it once.** A fact in the crate, the vocabulary or the specification is
  linked, never restated: two statements of one contract can disagree, and have.
- **No archaeology.** What a file used to be, and what changed in a move, is
  git's job. Not a header, not a comment.
- **Why, never what.** A comment restating the line below it goes. A reason that
  belongs to a term goes in its `rdfs:comment` or `sh:description`, where it is
  machine-readable, not in a header block above it.
- Every fact copied from NCBI or a sibling repository names its source and date.
