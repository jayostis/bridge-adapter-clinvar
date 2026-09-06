# Changelog

All notable changes to this adapter are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
semantic versioning and are the `version` in `adapter.yaml`.

## [0.1.0] - 2026-09-06

Phase 1 of [#1](https://github.com/jayostis/bridge-adapter-clinvar/issues/1):
the adapter laid out, with no mapping and nothing that executes.

### Added

- The layout of section 6 of the Cascade Bridge RFC
  ([spec#43](https://github.com/the-cascade-protocol/spec/issues/43)):
  `adapter.yaml`, `schema/`, `fixtures/` with `in/`, `expected/` and
  `findings/`, and `docs/`. The phase 2 and 3 directories (`in/xslt/`,
  `in/sparql/`, `tables/`, `vocab/`) are named in the README and the manifest
  and not yet created.
- `adapter.yaml`, the Bridge-facing manifest: format id `clinvar`, the two
  envelopes and the `VariationArchive` unit, the detect rule, the `xslt-3`
  profile requirement, the vocabulary pin (spec `e77ba5e`, `genomics
  v1-draft`, `core 3.13`), and the path to the cases file.
- `schema/manifest/adapter.schema.json` and `schema/manifest/cases.schema.json`,
  JSON Schemas for the two manifests, referenced from each YAML file's
  `# yaml-language-server: $schema=` header and from `.vscode/settings.json`.
- `schema/ClinVar_VCV_2.6.xsd`, NCBI's schema pinned byte for byte
  (md5 `a7b65e5a166dc5f36a7eea9127d56f4e`, NCBI's own). 2.6 rather than the
  2.5 the issue named, because the 2026-09 monthly release declares 2.6; the
  difference is one optional attribute.
- `schema/ClinVarResult-Set.xsd`, a wrapper that includes NCBI's schema and
  declares the efetch envelope root NCBI's schema leaves undeclared, so a
  whole efetch response validates and not only the records inside it.
- `fixtures/cases.yaml`: the four conformance oracles with the comparison
  rule declared (`isomorphic`, stamp predicates ignored, findings `exact`),
  NCBI's official sample as an input-only case, and the monthly and weekly
  releases as dataset cases whose record count and output digest are `null`
  until the first run.
- `fixtures/ro-crate-metadata.json` (RO-Crate 1.2): provenance, digest,
  version, date and licence for every committed fixture file, the two remote
  releases and the pinned XSD. It states that the four oracle inputs have no
  recorded fetch date or method.
- `fixtures/in/`: the four conformance inputs, byte-identical to
  `conformance/fixtures/genomics/clinvar/` at `0ea48bb`, and NCBI's sample
  `VCV_XML_VCV000091629.xml`. `fixtures/expected/` and `fixtures/findings/`:
  the four oracles' expected graphs and gaps sidecars, byte-identical.
- `docs/format.md`, ClinVar VCV XML as the adapter sees it, and
  `docs/stages.md`, the RFC's engine stages named with their Enterprise
  Integration Patterns equivalents.
- Editor configuration: `.vscode/extensions.json` (XML, XSLT/XPath, YAML,
  Turtle, Kaoto), `.vscode/settings.json` (XSD association for
  `fixtures/in/`, YAML schema associations), `.editorconfig`,
  `.gitattributes` (LF everywhere; verbatim copies never normalised).
- `LICENSE` (Apache-2.0), `README.md`, `CLAUDE.md`.

[0.1.0]: https://github.com/jayostis/bridge-adapter-clinvar/releases/tag/v0.1.0
