# Changelog

All notable changes to this adapter are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
semantic versioning and are the root entity's `version` in
`ro-crate-metadata.json`.

## [Unreleased]

### Changed

- Agent context split by directory. `CLAUDE.md` at the root, 218 lines to 79,
  holds only what has to be known before choosing a directory to open; the rest
  moved to `fixtures/CLAUDE.md`, `schema/CLAUDE.md` and
  `.github/workflows/CLAUDE.md`, which load only when those directories are
  touched. The layout section went: the README already carried it. The pinned
  commits went: the crate already carries them.
- Three conventions added to both this repository and the specification: say it
  once, no archaeology, and why never what. They are what the split is measured
  against.
- `.github/workflows/validate.yml` keeps two lines of comment instead of
  seventeen; the reasoning is in the `CLAUDE.md` beside it, stated once.

### Added

- Crate entities for `fixtures/CLAUDE.md` and `schema/CLAUDE.md`. The lint's
  allowlist matches the root `CLAUDE.md` by exact path, so a nested one is a
  file the crate must describe or check 3 fails.

## [0.2.0] - 2026-09-06

[#4](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/4): the
adapter says which specification it is written against, validates itself against
that specification in its own CI, and stops carrying a copy of it. Closes
[#3](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/3).

Three states were correct while this adapter was the only thing that existed and
are wrong now that
[cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec) does:
nothing checked this repository, it carried its own copies of that
specification's vocabulary and shapes, and it did not say which specification it
was written against. No mapping yet; still nothing here executes.

### Added

- `bridge:specPin` on the crate's root entity, naming
  [`0af0fc9`](https://github.com/jayostis/cascade-bridge-spec/commit/0af0fc970f75360d4f84acc2203c26b5fd9d6d23)
  of `cascade-bridge-spec`, tagged `v0.2.0` there, as a `SoftwareSourceCode`
  entity with `codeRepository` and `version` — the shape `bridge:vocabularyPin`
  already used. That commit is tagged, which is the condition
  `docs/alignment.md` there sets for a pin: a pin to an untagged commit is a bet
  on nobody rewriting a branch.
- `.github/workflows/validate.yml`, which checks out this repository and calls
  `jayostis/cascade-bridge-spec/.github/actions/validate-adapter@v0.2.0`. It has
  no logic of its own and must not grow any. The ref and `bridge:specPin` name
  the same commit, one for the machine and one for the reader; the lint compares
  them and fails the run when they differ, so they move together.
- The adapter profile IRI,
  `https://ns.cascadeprotocol.org/bridge/v1-draft/adapter-profile/`, in the root
  entity's `conformsTo`, with a contextual entity describing it. **The RFC issue
  URL stays alongside it**, deliberately: the profile is the machine-checkable
  claim a validator tests, the RFC is the standard being implemented, RO-Crate
  permits several `conformsTo` values, and dropping the RFC would lose the only
  link from this crate to the document the whole design answers to. The profile
  IRI does not dereference yet, which that repository's README records.

### Removed

- `schema/manifest/bridge.ttl` and `schema/manifest/bridge.shapes.ttl`, and the
  `schema/manifest/` directory. They were the seed of `cascade-bridge-spec`'s
  `vocab/` and `shapes/`, and once that repository existed a copy here was a
  second statement of a contract, which is a statement that can disagree with
  it — and the two had already begun to. The lint fetches the specification at
  the pinned commit and validates against the shapes there.

  The two improvements made here during
  [#2](https://github.com/jayostis/cascade-bridge-adapter-clinvar/pull/2)'s
  review were carried up first, and were checked to be present at the pinned
  commit before anything was deleted: `sh:class schema:MediaObject` on the five
  file-valued property shapes, and the findings comparison as a multiset.
- The crate's `hasPart` entries and data entities for those two files, and every
  reference to `schema/manifest/` in `README.md`, `CLAUDE.md`,
  `fixtures/manifest.ttl`'s header, `.vscode/settings.json` and
  `.vscode/extensions.json`.

### Changed

- `README.md` and `CLAUDE.md`: the adapter pins a specification and is validated
  against it in CI; the vocabulary and shapes live in `cascade-bridge-spec`, not
  here; the verification list says which three checks CI runs and which a
  contributor should still run locally before pushing.
- `CLAUDE.md`'s rules state explicitly why a lint workflow does not break "no
  tests here", rather than leaving a reader to infer it from a workflow file:
  **running an adapter's fixtures is a Bridge's job; checking that the package
  is well formed is a lint, and it belongs where the package lives.** Nothing in
  the workflow runs a mapping or compares a graph.
- `CLAUDE.md` gains `../cascade-bridge-spec` as a sibling checkout and gains the
  rule that no copy of the specification is kept here.
- The root entity's `version` is `0.2.0`.

### Unchanged, and checked

- Nothing under `fixtures/in/`, `fixtures/expected/`, `fixtures/findings/` or
  `schema/ClinVar_VCV_2.6.xsd` changed by a byte, and every `sha256` in the
  crate still matches its file.

## [0.1.0] - 2026-09-06

Phase 1 of [#1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1):
the adapter laid out, with no mapping and nothing that executes.

### Added

- The layout of section 6 of the Cascade Bridge RFC
  ([spec#43](https://github.com/the-cascade-protocol/spec/issues/43)), with
  one departure: the manifest is an RO-Crate at the root, not an
  `adapter.yaml`. `schema/`, `fixtures/` with `in/`, `expected/` and
  `findings/`, and `docs/`. The phase 2 and 3 directories (`in/xslt/`,
  `in/sparql/`, `tables/`, `vocab/`) are named in the README and not yet
  created.
- `ro-crate-metadata.json` (RO-Crate 1.2), the adapter manifest and the
  provenance record in one graph. Its root entity is a `Dataset` and a
  `bridge:Adapter`: format id `clinvar`, name, version, licence (SPDX
  Apache-2.0, as an entity), the RFC it conforms to, the `xslt-3` profile
  requirement (no tier: an adapter declares what it needs, and its tier is
  measured by the catalog from which Bridges pass its test manifest, RFC
  section 11), the vocabulary pin (spec `e77ba5e`, an entity naming the
  fork and why the commit is there) and the two vocabularies it writes
  (`genomics v1-draft`, `core 3.13`, as entities), the source media type
  and schema, the two envelopes (`#envelope-efetch` with its wrapper
  schema, `#envelope-release`) as entities, the
  `VariationArchive` unit, the detect rule as one XPath 3.1 expression,
  and the test manifest. Its data entities carry provenance, digest,
  version, date and licence for every committed fixture file, the pinned
  XSD (a local `File` based on NCBI's URL entity), the wrapper schema, the
  vocabulary and shapes files, the docs and the monthly release. No weekly
  release is listed: on 2026-09-06 the weekly symlink resolved to the
  monthly file, and an entity with no content of its own is not something a
  Bridge can run; a dated weekly is added when one exists. It states that
  the four oracle inputs have no recorded fetch date or method, and links
  the conformance commit they were copied from on the fork that holds it.
- `schema/manifest/bridge.ttl`, the `bridge:` Cascade Bridge vocabulary
  (`https://ns.cascadeprotocol.org/bridge/v1-draft#`), with an
  `rdfs:comment` on every term: the adapter terms the crate's root entity
  and envelopes use (three classes, one individual, thirteen properties,
  two of them, `bridge:table` and `bridge:extensionVocabulary`, declared for
  phase 2 and not yet used) and, on top of W3C's `mf:` test-manifest
  vocabulary, the test terms the test manifest uses (three test types that
  carry the comparison rule and eight properties; `bridge:envelope` and
  `bridge:adapter` serve both). And `schema/manifest/bridge.shapes.ttl`, the
  SHACL shapes the crate and the test manifest validate against as one
  graph, including, as SHACL-SPARQL constraints, the links between the two:
  the adapter and manifest point at each other, and every envelope a test
  names is one the adapter lists. Both are the seed of the Bridge
  specification's vocabulary; no `cascade:` or `genomics:` term is minted.
- `schema/ClinVar_VCV_2.6.xsd`, NCBI's schema pinned byte for byte
  (md5 `a7b65e5a166dc5f36a7eea9127d56f4e`, NCBI's own). 2.6 rather than the
  2.5 the issue named, because the 2026-09 monthly release declares 2.6; the
  difference is one optional attribute.
- `schema/ClinVarResult-Set.xsd`, a wrapper that includes NCBI's schema and
  declares the efetch envelope root NCBI's schema leaves undeclared, so a
  whole efetch response validates and not only the records inside it. The root
  holds either one or more `VariationArchive` elements or the single empty
  `set` element efetch returns when a query matched no record, which is a
  response and not an error.
- `fixtures/manifest.ttl`, a W3C-style test manifest in Turtle: the four
  conformance oracles as `bridge:IsomorphicConversionTest` (blank nodes
  relabelled, IRIs and literals exact, the stamp predicates ignored, findings
  compared as a multiset), NCBI's official sample as `bridge:InputOnlyTest`,
  and the monthly
  release as a `bridge:DatasetCompletionTest` naming the crate's Dataset
  entity by IRI, its record count and output digest absent until the first
  run. The stamp predicates are stated once, on the manifest, and every
  entry inherits them. It names the crate's root entity as its `bridge:adapter`
  and each action's envelope by the crate entity's IRI, so the manifest and
  the crate load as one graph and every link is checkable in it.
- `fixtures/in/`: the four conformance inputs, byte-identical to
  `conformance/fixtures/genomics/clinvar/` at `0ea48bb`, and NCBI's sample
  `VCV_XML_VCV000091629.xml`. `fixtures/expected/` and `fixtures/findings/`:
  the four oracles' expected graphs and gaps sidecars, byte-identical.
- `docs/format.md`, ClinVar VCV XML as the adapter sees it, and
  `docs/stages.md`, the RFC's engine stages named with their Enterprise
  Integration Patterns equivalents.
- Editor configuration: `.vscode/extensions.json` (XML, XSLT/XPath, Turtle,
  EditorConfig, Kaoto), `.vscode/settings.json` (XSD association for
  `fixtures/in/`, no save-time rewriting of XML or Turtle), `.editorconfig`,
  `.gitattributes` (LF everywhere; verbatim copies never normalised).
- `LICENSE` (Apache-2.0), `README.md`, `CLAUDE.md`.

### Decided

- **Findings are compared as a multiset, not entry for entry.** The rule this
  repository first wrote, "in the conformance repository's sort order
  (sourceField, severity, reason)", was derived from cascade-cli's
  `localeCompare` sort and is under-specified: `localeCompare` is ICU
  collation, in which `/` precedes `@` against their code points, so a
  `bridge-engine-java` harness using `String.compareTo` or a browser harness
  using `<` orders the same correctly-mapped findings differently and reports
  a false failure. Nothing outside this repository required the ordering: the
  RFC ([spec#43](https://github.com/the-cascade-protocol/spec/issues/43)
  section 11) only describes the cli's byte assertion as current practice, and
  issue #1 said `compare.findings: exact` with no order. The contract is now
  the same entries with the same multiplicity in any order, entries compared
  as JSON objects. Multiplicity is kept because entries repeat. A sidecar
  written from scratch SHOULD be sorted by Unicode code point, as file hygiene
  and never as part of the comparison; the four committed sidecars are
  verbatim copies and are not re-sorted, so their order is incidental. To be
  carried to `jayostis/cascade-bridge-spec` and to spec#43.
- The general form of that rule, stated in the manifest header: a comparison
  is insensitive to everything the format does not mean. Blank node labels on
  the graph side, element order on the findings side.

[0.2.0]: https://github.com/jayostis/cascade-bridge-adapter-clinvar/releases/tag/v0.2.0
[0.1.0]: https://github.com/jayostis/cascade-bridge-adapter-clinvar/releases/tag/v0.1.0
