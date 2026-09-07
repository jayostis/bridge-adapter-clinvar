# cascade-bridge-adapter-clinvar — Agent Context

## Repository purpose

The **Cascade Bridge Adapter** for ClinVar VCV XML: a package of data
(manifest, pinned schema, fixtures with provenance, and later the mapping) that
a **Cascade Bridge** runs to turn ClinVar records into Cascade RDF. It is the
pilot adapter that the Cascade Bridge RFC,
[the-cascade-protocol/spec#43](https://github.com/the-cascade-protocol/spec/issues/43),
proposes in its section 16. Import-only.

Phase 1 (this layout, version 0.2.0) has no mapping and nothing that executes.
The phases and every decision behind the layout are in
[issue #1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1); the
README summarises them. Do not re-derive them.

The contract this adapter is written against is the **Cascade Bridge
Specification**, [cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec),
pinned by the crate's `bridge:specPin` and named in `conformsTo` by the adapter
profile IRI. That repository holds the `bridge:` vocabulary, the SHACL shapes
and the lint; this repository holds none of them and must not hold a copy.
Dependencies point one way: an adapter knows about the specification and the
specification knows nothing about any adapter.

## The rules

- **No code.** Markdown, XML, XSD, XSLT, YAML, JSON, JSON-LD, CSV, Turtle,
  SPARQL. No Java, JavaScript, TypeScript, Python, shell scripts, or Makefile,
  in any directory. An adapter is data; the thing that runs it is a Bridge
  (RFC section 6, "a universal adapter contains no code"). If something seems
  to need code, it is a finding for the Bridge specification, not a file here.
  This is now measured rather than promised: the specification's lint takes an
  inventory of every git-tracked file and refuses one whose declared
  `encodingFormat` is outside the allowed media types, and a file the crate
  does not describe at all is a file nobody reviewed.
- **The manifest is the crate.** `ro-crate-metadata.json` at the root is
  both the adapter's manifest (its root entity is a `bridge:Adapter`: format
  id, envelopes, unit, detect XPath, profile, vocabulary pin, test manifest)
  and the provenance record for every file and remote dataset. There is no
  `adapter.yaml`; the RFC's section 6 sketch names one, and the departure is
  a finding for spec#43, not a reason to add one back.
- **No tests here; one lint, and it is not this repository's.** The adapter
  declares its fixtures and how to judge them as data, in
  `fixtures/manifest.ttl`, a W3C-style test manifest (`mf:` plus the test terms
  of the `bridge:` vocabulary). A Bridge's harness executes them: version 1 in
  `bridge-engine-java`, version 2 in `bridge-engine-browser`. Neither
  repository exists yet. Do not add a test runner, a CI workflow that runs
  fixtures, or a scripts directory here.

  `.github/workflows/validate.yml` does not break that rule, and the
  distinction is worth stating rather than leaving a reader to infer it from a
  workflow file. **Running an adapter's fixtures is a Bridge's job. Checking
  that the package is well formed is a lint, and it belongs where the package
  lives.** The workflow runs no mapping and compares no graph; it checks out
  this repository and calls the action the specification publishes, at the tag
  that names the pinned commit. It contains no logic and must not grow any: a
  check that needed writing is a change to the specification's lint, made
  there, and consumed here by moving the pin.
- **Big datasets are referenced, not committed.** Small inputs are checked in.
  A multi-gigabyte release is a URL, a version, a digest and a licence in
  `ro-crate-metadata.json`, and a `bridge:DatasetCompletionTest` in
  `fixtures/manifest.ttl` naming that entity by its IRI.
- **Verbatim copies are never edited.** `schema/ClinVar_VCV_2.6.xsd` and
  everything under `fixtures/in/`, `fixtures/expected/` and
  `fixtures/findings/` are byte-for-byte copies whose digests the crate
  records. `.gitattributes` and `.editorconfig` protect them from
  normalisation. To change one, replace it from its source and update the
  crate's digest, size and description in the same commit.
- **Identity is not a blocker.** How a record's IRI is minted is decided in
  phase 2 when the Variant template is written; the position in
  [spec#38](https://github.com/the-cascade-protocol/spec/issues/38#issuecomment-5555482906)
  is the working basis and the layout does not depend on it. Do not reopen
  naming to make progress on anything else.
- **No Cascade terms are minted here.** Values with no Cascade term go in the
  adapter's own namespace (`vocab/`, phase 2) or in the findings sidecar;
  terms in `cascade:`, `genomics:` and the rest go through spec's RFC process.
- **No copy of the specification.** The `bridge:` vocabulary and its SHACL
  shapes lived here, under `schema/manifest/`, until 0.2.0; they were the seed
  of `cascade-bridge-spec` and were deleted once it existed. A second statement
  of a contract is a statement that can disagree with it, and the two had
  already begun to. To change a shape or a term, change it there, in a pull
  request against that repository, tag it, and move `bridge:specPin` and the
  workflow's ref here in the same commit.
- **The pin is written twice and must agree.** `bridge:specPin` in the crate
  and the ref in `.github/workflows/validate.yml` name the same commit of
  `cascade-bridge-spec`, one for the reader and one for the machine. The lint
  compares them and fails the run when they differ. Every commit pinned is
  tagged in that repository; a pin to an untagged commit is a bet on nobody
  rewriting a branch.

## Sibling checkouts this repository cites

Expected beside this repository, as sister directories:

- `../cascade-bridge-spec` — the **Cascade Bridge Specification**
  (`jayostis/cascade-bridge-spec`): the `bridge:` vocabulary, the SHACL shapes,
  the adapter profile, the prose contract and the lint this repository's CI
  calls. Pinned at `0af0fc970f75360d4f84acc2203c26b5fd9d6d23`, tagged `v0.2.0`,
  by the crate's `bridge:specPin` and by the ref in
  `.github/workflows/validate.yml`. `vocab/bridge.ttl` and
  `shapes/bridge.shapes.ttl` there were seeded from this adapter's
  `schema/manifest/` at `0b2d548`, which is why those two files are gone from
  here. `docs/validation.md` there is the contract the lint implements and
  `docs/alignment.md` is why the pin works the way it does.
- `../spec` — the vocabularies; pinned at `e77ba5e8004bf57f34d42a1bee69d1cfb95f86e3`
  by the crate's `bridge:vocabularyPin` (the same pin as
  `../conformance/scripts/SPEC_PIN`). The commit is on the fork,
  `jayostis/spec`, and not on the org's `main`; the crate links the fork and
  its pin entity says why. The RFC is spec#43; the identity RFC is spec#38.
- `../conformance` — `fixtures/genomics/clinvar/` at `0ea48bb` is where the
  four oracle triplets are copied from. That commit too is on the fork,
  `jayostis/conformance`, and not on the org's `main`; the files last changed
  at `8a5e203`, which is on the org's `main` with identical blobs. The crate
  links the fork.
  `X.input.xml` becomes `in/X.xml`, `X.expected.ttl` becomes
  `expected/X.ttl`, and `X.gaps.json` keeps its suffix as
  `findings/X.gaps.json`.
- `../cascade-cli` — `src/lib/clinvar-converter/` (about 2,200 lines) is the
  converter the adapter re-expresses as data, and
  `tests/clinvar-conformance.test.ts` is the oracle comparison the test
  manifest's `bridge:IsomorphicConversionTest` restates. Format id `clinvar`
  comes from its `registry-entry.ts`.
- `../sdk-typescript` — the runtime lineage the RFC positions the Bridge
  beside; not read by anything here yet.

Facts about ClinVar's files, the two envelopes, XSD versions and NCBI's
release cadence are in `docs/format.md`, verified against
`ftp.ncbi.nlm.nih.gov` on 2026-09-06. Re-verify against the site before
changing a pin; NCBI publishes a `.md5` beside every release and XSD.

## Layout

```
ro-crate-metadata.json           the adapter manifest, and the provenance of every committed fixture, schema and document, and every dataset (RO-Crate 1.2)
schema/ClinVar_VCV_2.6.xsd       NCBI's schema, pinned byte for byte
schema/ClinVarResult-Set.xsd     efetch envelope wrapper: includes NCBI's, adds the root it lacks
(the bridge: vocabulary and its SHACL shapes are NOT here: they are in cascade-bridge-spec, at bridge:specPin)
.github/workflows/validate.yml   calls the specification's published lint at the pinned tag; no logic of its own
fixtures/manifest.ttl            the test manifest: the cases and how each is judged (W3C mf: plus bridge:)
fixtures/in|expected|findings/   inputs, expected graphs, expected findings
docs/format.md                   ClinVar VCV XML as the adapter sees it
docs/stages.md                   RFC engine stages in Enterprise Integration Patterns terms
in/xslt/, in/sparql/, tables/, vocab/   phase 2 and 3; not yet created
```

## What can be checked without a Bridge

There is no test suite by design. Three checks now run in CI, on every pull
request, by calling the specification's published lint; the rest are still by
hand, and a commit says which of them ran.

**What CI runs.** `.github/workflows/validate.yml` calls
`jayostis/cascade-bridge-spec/.github/actions/validate-adapter` at the tag that
names the pinned commit. It runs, and their contract is `docs/validation.md`
there:

1. the crate is a valid RO-Crate 1.2;
2. the crate (`ro-crate-metadata.json`, parsed as JSON-LD with its own location
   as base) and `fixtures/manifest.ttl` (parsed with its location as base) load
   as one RDF graph and conform to that repository's `shapes/bridge.shapes.ttl`
   under pySHACL with SHACL-SPARQL. The shapes carry the links between the two
   (every `bridge:envelope` in a test action is one the root entity lists; the
   manifest's `bridge:adapter` is the root entity and the root's
   `bridge:testManifest` is the manifest; every `bridge:dataset` is a crate
   `Dataset` with a `contentUrl`), and they require every IRI naming a
   committed file to be a crate `File` entity: `bridge:input`, `bridge:graph`,
   `bridge:findings`, `bridge:sourceSchema` and each `bridge:documentSchema`
   (`sh:class schema:MediaObject`, what the RO-Crate context expands `File`
   to), so a typo that names no entity fails rather than passing quietly. The
   crate's `bridge:specPin` is checked against the revision the workflow calls;
3. every git-tracked file is either a crate entity with a declared
   `encodingFormat` in the allowed set, or is `README.md`, `LICENSE`,
   `CHANGELOG.md`, `CLAUDE.md`, `ro-crate-metadata.json` or a dotfile.

Reproduce any of them locally with
`python3 <path to a cascade-bridge-spec checkout>/scripts/validate-adapter.py .`
at the pinned commit of that repository.

**What is still by hand, before pushing.** None of it needs a Bridge, and CI
cannot do it: SHACL cannot see the filesystem, and checks 4 to 6 of
`docs/validation.md` there are specified and not built.

- Each crate `File` the shapes accept exists on disk.
- Every input under `fixtures/in/` validates against
  `schema/ClinVarResult-Set.xsd` (any XSD 1.0 validator: `xmllint --schema`,
  lxml, the Red Hat XML extension in VS Code), and the crate's
  `bridge:detectXPath` is true for each of them (an XPath 3.1 evaluator such
  as Saxon; XPath 1.0's
  `boolean(/ClinVarResult-Set/VariationArchive | /ClinVarResult-Set/set | /ClinVarVariationRelease/VariationArchive)`
  is an acceptable stand-in, and the note says so). It must also be true for
  `<ClinVarResult-Set><set/></ClinVarResult-Set>`, the response efetch
  returns when a query matched no record, and false for a document under a
  known root that holds neither (`docs/stages.md`, Content-Based Router).
- Every `fixtures/expected/*.ttl` parses as Turtle (`riot --validate`, rdflib).
- Every digest in the crate matches its file (`sha256sum`), and the XSD's md5
  matches NCBI's published `.md5`.
- The four oracle triplets are byte-identical to conformance at `0ea48bb`:
  compare `git hash-object <file>` here against
  `git -C ../conformance ls-tree 0ea48bb fixtures/genomics/clinvar/`, or run
  `git -C ../conformance show 0ea48bb:<path> | cmp - <file>`. Compare
  against the blobs, never against `../conformance`'s worktree: a clone with
  `core.autocrlf=true` holds those files as CRLF, so `git diff --no-index`
  or `cmp` on the worktree reports every input as different at byte 40, and
  "fixing" that would break the recorded digests.

CI on this project's sibling repositories is Linux and invokes `python3` and
the JVM tools directly; do not commit any machine-specific way of running
them here.

## Conventions

- Conventional commits: `feat(adapter): ...`, `docs: ...`, `fix(fixtures): ...`.
- Issue and document text is written impersonally, as findings and decisions,
  not as promises by a person.
- `CHANGELOG.md` is updated in the same commit as the change it describes;
  its version is the root entity's `version` in `ro-crate-metadata.json`.
- Every fact copied from NCBI or a sibling repository names its source and
  date, in the crate for data and in `docs/` for prose.
