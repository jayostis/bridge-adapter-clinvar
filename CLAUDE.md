# bridge-adapter-clinvar — Agent Context

## Repository purpose

The **Cascade Bridge Adapter** for ClinVar VCV XML: a package of data
(manifest, pinned schema, fixtures with provenance, and later the mapping) that
a **Cascade Bridge** runs to turn ClinVar records into Cascade RDF. It is the
pilot adapter that the Cascade Bridge RFC,
[the-cascade-protocol/spec#43](https://github.com/the-cascade-protocol/spec/issues/43),
proposes in its section 16. Import-only.

Phase 1 (this layout, version 0.1.0) has no mapping and nothing that executes.
The phases and every decision behind the layout are in
[issue #1](https://github.com/jayostis/bridge-adapter-clinvar/issues/1); the
README summarises them. Do not re-derive them.

## The rules

- **No code.** Markdown, XML, XSD, XSLT, YAML, JSON, JSON-LD, CSV, Turtle,
  SPARQL. No Java, JavaScript, TypeScript, Python, shell scripts, or Makefile,
  in any directory. An adapter is data; the thing that runs it is a Bridge
  (RFC section 6, "a universal adapter contains no code"). If something seems
  to need code, it is a finding for the Bridge specification, not a file here.
- **No tests here.** The adapter declares its fixtures and how to judge them
  as data, in `fixtures/cases.yaml`. A Bridge's harness executes them:
  version 1 in `bridge-engine-java`, version 2 in `bridge-engine-browser`.
  Neither repository exists yet. Do not add a test runner, CI workflow that
  runs fixtures, or scripts directory to this repository.
- **Big datasets are referenced, not committed.** Small inputs are checked in.
  A multi-gigabyte release is a URL, a version, a digest and a licence in
  `fixtures/ro-crate-metadata.json`, and a dataset case in `cases.yaml`.
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

## Sibling checkouts this repository cites

Expected beside this repository, as sister directories:

- `../spec` — the vocabularies; pinned at `e77ba5e8004bf57f34d42a1bee69d1cfb95f86e3`
  in `adapter.yaml` (the same pin as `../conformance/scripts/SPEC_PIN`). The
  RFC is spec#43; the identity RFC is spec#38.
- `../conformance` — `fixtures/genomics/clinvar/` at `0ea48bb` is where the
  four oracle triplets are copied from. Their `.input.xml` / `.expected.ttl` /
  `.gaps.json` naming becomes `in/` / `expected/` / `findings/` here.
- `../cascade-cli` — `src/lib/clinvar-converter/` (about 2,200 lines) is the
  converter the adapter re-expresses as data, and
  `tests/clinvar-conformance.test.ts` is the oracle comparison the cases
  file's `compare` rule restates. Format id `clinvar` comes from its
  `registry-entry.ts`.
- `../sdk-typescript` — the runtime lineage the RFC positions the Bridge
  beside; not read by anything here yet.

Facts about ClinVar's files, the two envelopes, XSD versions and NCBI's
release cadence are in `docs/format.md`, verified against
`ftp.ncbi.nlm.nih.gov` on 2026-09-06. Re-verify against the site before
changing a pin; NCBI publishes a `.md5` beside every release and XSD.

## Layout

```
adapter.yaml                     Bridge-facing manifest (schema/manifest/adapter.schema.json)
schema/ClinVar_VCV_2.6.xsd       NCBI's schema, pinned byte for byte
schema/ClinVarResult-Set.xsd     efetch envelope wrapper: includes NCBI's, adds the root it lacks
schema/manifest/                 JSON Schemas for adapter.yaml and fixtures/cases.yaml
fixtures/cases.yaml              the cases and how each is judged
fixtures/ro-crate-metadata.json  provenance for every fixture file and remote dataset (RO-Crate 1.2)
fixtures/in|expected|findings/   inputs, expected graphs, expected findings
docs/format.md                   ClinVar VCV XML as the adapter sees it
docs/stages.md                   RFC engine stages in Enterprise Integration Patterns terms
in/xslt/, in/sparql/, tables/, vocab/   phase 2 and 3; not yet created
```

## What can be checked without a Bridge

There is no test suite by design. Before claiming a change works, check what
is checkable with generic tools, and say which of these ran:

- Every YAML and JSON file parses, and the two manifests validate against
  their JSON Schemas (`jsonschema`, draft 2020-12).
- The crate validates with the RO-Crate validator
  (`rocrate-validator validate fixtures/ --profile-identifier ro-crate-1.2`);
  where that cannot run, a JSON-LD parse is the floor and the commit says so.
- Every input under `fixtures/in/` validates against
  `schema/ClinVarResult-Set.xsd` (any XSD 1.0 validator: `xmllint --schema`,
  lxml, the Red Hat XML extension in VS Code).
- Every `fixtures/expected/*.ttl` parses as Turtle (`riot --validate`, rdflib).
- Every digest in the crate matches its file (`sha256sum`), and the XSD's md5
  matches NCBI's published `.md5`.
- The four oracle triplets are byte-identical to conformance at `0ea48bb`
  (`git diff --no-index`, or compare `git hash-object` output against
  `git -C ../conformance ls-tree 0ea48bb fixtures/genomics/clinvar/`).

CI on this project's sibling repositories is Linux and invokes `python3` and
the JVM tools directly; do not commit any machine-specific way of running
them here.

## Conventions

- Conventional commits: `feat(adapter): ...`, `docs: ...`, `fix(fixtures): ...`.
- Issue and document text is written impersonally, as findings and decisions,
  not as promises by a person.
- `CHANGELOG.md` is updated in the same commit as the change it describes;
  its version is the `version` in `adapter.yaml`.
- Every fact copied from NCBI or a sibling repository names its source and
  date, in the crate for data and in `docs/` for prose.
