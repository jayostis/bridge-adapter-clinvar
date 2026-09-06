# cascade-bridge-adapter-clinvar — Agent Context

## Repository purpose

The **Cascade Bridge Adapter** for ClinVar VCV XML: a package of data
(manifest, pinned schema, fixtures with provenance, and later the mapping) that
a **Cascade Bridge** runs to turn ClinVar records into Cascade RDF. It is the
pilot adapter that the Cascade Bridge RFC,
[the-cascade-protocol/spec#43](https://github.com/the-cascade-protocol/spec/issues/43),
proposes in its section 16. Import-only.

Phase 1 (this layout, version 0.1.0) has no mapping and nothing that executes.
The phases and every decision behind the layout are in
[issue #1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1); the
README summarises them. Do not re-derive them.

## The rules

- **No code.** Markdown, XML, XSD, XSLT, YAML, JSON, JSON-LD, CSV, Turtle,
  SPARQL. No Java, JavaScript, TypeScript, Python, shell scripts, or Makefile,
  in any directory. An adapter is data; the thing that runs it is a Bridge
  (RFC section 6, "a universal adapter contains no code"). If something seems
  to need code, it is a finding for the Bridge specification, not a file here.
- **The manifest is the crate.** `ro-crate-metadata.json` at the root is
  both the adapter's manifest (its root entity is a `bridge:Adapter`: format
  id, envelopes, unit, detect XPath, profile, vocabulary pin, test manifest)
  and the provenance record for every file and remote dataset. There is no
  `adapter.yaml`; the RFC's section 6 sketch names one, and the departure is
  a finding for spec#43, not a reason to add one back.
- **No tests here.** The adapter declares its fixtures and how to judge them
  as data, in `fixtures/manifest.ttl`, a W3C-style test manifest (`mf:` plus
  the test terms of the `bridge:` vocabulary). A Bridge's harness executes
  them: version 1 in `bridge-engine-java`, version 2 in
  `bridge-engine-browser`. Neither repository exists yet. Do not add a test
  runner, CI workflow that runs fixtures, or scripts directory to this
  repository.
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
  The `bridge:` vocabulary under `schema/manifest/` is a Bridge-spec
  namespace, the seed of the specification's own, and moves out unchanged
  when a `bridge-spec` repository exists.

## Sibling checkouts this repository cites

Expected beside this repository, as sister directories:

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
schema/manifest/                 the bridge: vocabulary (adapter and test terms) and its SHACL shapes
fixtures/manifest.ttl            the test manifest: the cases and how each is judged (W3C mf: plus bridge:)
fixtures/in|expected|findings/   inputs, expected graphs, expected findings
docs/format.md                   ClinVar VCV XML as the adapter sees it
docs/stages.md                   RFC engine stages in Enterprise Integration Patterns terms
in/xslt/, in/sparql/, tables/, vocab/   phase 2 and 3; not yet created
```

## What can be checked without a Bridge

There is no test suite by design. Before claiming a change works, check what
is checkable with generic tools, and say which of these ran:

- Every JSON and Turtle file parses. The crate (`ro-crate-metadata.json`,
  parsed as JSON-LD with its own location as base) and `fixtures/manifest.ttl`
  (parsed with its location as base) load as one RDF graph and validate,
  conforming, against `schema/manifest/bridge.shapes.ttl` with a SHACL engine
  that supports SHACL-SPARQL (pySHACL or Jena). The shapes carry the links
  between the two: every `bridge:envelope` in a test action is one the root
  entity lists; the manifest's `bridge:adapter` is the root entity and the
  root's `bridge:testManifest` is the manifest; every `bridge:dataset` is a
  crate `Dataset` with a `contentUrl`.
- In that graph, the shapes already require every IRI that names a committed
  file to be a crate `File` entity: `bridge:input`, `bridge:graph`,
  `bridge:findings`, `bridge:sourceSchema` and each `bridge:documentSchema`
  (`sh:class schema:MediaObject`, what the RO-Crate context expands `File`
  to). A typo that names no entity fails the SHACL run, so there is no script
  to write for it. SHACL cannot see the filesystem, so check by hand that each
  of those crate `File`s exists on disk, and say that you did.
- The crate validates with the RO-Crate validator
  (`rocrate-validator validate . --profile-identifier ro-crate-1.2`, from the
  root); where that cannot run, a JSON-LD parse is the floor and the commit
  says so.
- Every input under `fixtures/in/` validates against
  `schema/ClinVarResult-Set.xsd` (any XSD 1.0 validator: `xmllint --schema`,
  lxml, the Red Hat XML extension in VS Code), and the crate's
  `bridge:detectXPath` is true for each of them (an XPath 3.1 evaluator such
  as Saxon; XPath 1.0's
  `boolean(/ClinVarResult-Set/VariationArchive | /ClinVarVariationRelease/VariationArchive)`
  is an acceptable stand-in, and the note says so).
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
