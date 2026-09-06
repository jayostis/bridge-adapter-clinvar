# bridge-adapter-clinvar

The **Cascade Bridge Adapter** for ClinVar VCV XML: the package of data a
**Cascade Bridge** runs to turn NCBI ClinVar variation records into Cascade
RDF. It is the pilot adapter proposed in section 16 of the Cascade Bridge RFC,
[the-cascade-protocol/spec#43](https://github.com/the-cascade-protocol/spec/issues/43):
import-only, a published NCBI schema, no vendor quirks, an existing converter
of about 2,200 lines in cascade-cli to re-express as data, and four conformance
oracles already written.

There is no code here and there will be none. An adapter is mappings, schemas,
fixtures and a manifest; the thing that runs it is a Bridge. Nothing in this
repository executes.

## Status

**Phase 1, version 0.1.0: the adapter laid out.** Manifest, pinned schema,
fixtures with recorded provenance, references to the large datasets, editor
configuration. No mapping, no runner. Done when the repository opens in
VS Code with everything validating and nothing executes.

| phase | what | where | done when |
|---|---|---|---|
| 1 | this layout | here ([#1](https://github.com/jayostis/bridge-adapter-clinvar/issues/1)) | it opens in VS Code with everything validating |
| 2 | the first mapping (XSLT 3, one module per record class) and the engine that runs it | here and `bridge-engine-java` (Camel YAML route, Saxon-HE, riot) | the four committed cases compare isomorphic |
| 3 | the same mapping as SPARQL CONSTRUCT over the Bridge's XML lift | `in/sparql/` | the two graphs are diffed; the result is a finding for spec#43 |
| 4 | the same adapter, byte for byte, through JavaScript engines | `bridge-engine-browser` | the same `cases.yaml` passes in both |

## How a Bridge runs it

1. Reads `adapter.yaml`: format id `clinvar`, the two envelopes, the unit
   `VariationArchive`, the detect rule, the `xslt-3` profile it must offer,
   the vocabulary pin, and the path to the cases.
2. Routes an input here when its document element is `ClinVarResult-Set`
   (efetch) or `ClinVarVariationRelease` (a release file) and it contains a
   `VariationArchive`.
3. Splits the document on `VariationArchive` and validates each unit against
   `schema/ClinVar_VCV_2.6.xsd`. A unit that fails is a finding; it still goes
   through.
4. Runs the mapping (phase 2) on each unit, links the interpretation and
   submitter-assertion records to their Variant, stamps provenance, checks
   every predicate against the pinned vocabularies, validates with SHACL,
   and hands the graph and findings to the runtime.
5. In test, executes `fixtures/cases.yaml`: for each committed case, compares
   the produced graph with the expected one up to blank-node relabelling with
   the stamp predicates removed, and the produced findings with the expected
   sidecar exactly. For each dataset case, streams the referenced release and
   records the record count and output digest.

`docs/stages.md` names each of those stages with its Enterprise Integration
Pattern; `docs/format.md` describes the format the first three stages see.

## Layout

```
bridge-adapter-clinvar/
  README.md                  this file
  LICENSE                    Apache-2.0
  CHANGELOG.md
  CLAUDE.md                  agent context: the rules and the sibling checkouts
  adapter.yaml               the Bridge-facing manifest
  .gitattributes             LF everywhere; verbatim copies never normalised
  .editorconfig
  .vscode/
    extensions.json          XML, XSLT/XPath, YAML, Turtle, Kaoto
    settings.json            XSD association for fixtures/in, YAML schema associations
  docs/
    format.md                ClinVar VCV XML as the adapter sees it
    stages.md                the RFC's engine stages as Enterprise Integration Patterns
  schema/
    ClinVar_VCV_2.6.xsd      NCBI's schema, pinned byte for byte (md5 a7b65e5a166dc5f36a7eea9127d56f4e)
    ClinVarResult-Set.xsd    the efetch envelope root NCBI's schema does not declare; includes the above
    manifest/
      adapter.schema.json    JSON Schema for adapter.yaml
      cases.schema.json      JSON Schema for fixtures/cases.yaml
  fixtures/
    cases.yaml               the cases and how to judge each
    ro-crate-metadata.json   provenance for every file under fixtures/ and every remote dataset
    in/                      four conformance inputs and NCBI's official sample
    expected/                the four expected graphs, byte-identical to conformance
    findings/                the four expected gaps sidecars, byte-identical to conformance
  in/xslt/                   phase 2: clinvar.xsl entry, one module per record class, findings.xsl
  in/sparql/                 phase 3: the same mapping as CONSTRUCT over the Bridge's XML lift
  tables/                    phase 2: review-status.csv, from cascade-cli's review-status-map.ts
  vocab/                     phase 2: clinvar-ext.ttl, the adapter's namespace for unmapped values
```

The four directories marked phase 2 or 3 do not exist yet; they are named
here and in `adapter.yaml` so the shape is visible.

## Decisions

The reasoning is in [issue #1](https://github.com/jayostis/bridge-adapter-clinvar/issues/1);
the outcomes:

- **Standards, not inventions.** Enterprise Integration Patterns for stage
  names; XSLT 3, XSD and Schematron for the map and validation; RO-Crate 1.2
  for provenance; BagIt-style digests for inputs; the RFC's section 6 layout;
  the conformance repository's input / expected / gaps triplet as the fixture
  shape. The reason is the editor: every one of these already has tooling, so
  the repository lights up in VS Code without anyone writing a plugin, and
  every later choice is weighed against whether the standard editor still
  understands it.
- **Naming is deferred, and the layout does not care.** The converter mints
  record IRIs by SHA-1 over `genomics:Variant:clinvar:VCV…`; XSLT 3 has no
  hash function. Whether the mapping writes the recipe and the Bridge mints
  the name (recommended, and where [spec#38](https://github.com/the-cascade-protocol/spec/issues/38#issuecomment-5555482906)
  puts naming) or calls a Bridge-provided function is decided in phase 2.
- **Provenance is an RO-Crate**, one `fixtures/ro-crate-metadata.json`, over
  hand-written YAML: a published specification with validators, in JSON-LD.
- **Licence is Apache-2.0**, what conformance and cascade-cli use, because
  XSLT executes and a content licence sits awkwardly on files that execute.
  NCBI's terms for ClinVar data are recorded per file in the crate.
- **Schema pin is 2.6, not the 2.5 the issue named.** The 2026-09 monthly
  release declares 2.6, which appeared in NCBI's index in February 2026; the
  difference is one optional attribute. `docs/format.md` has the table.
- **An envelope wrapper schema exists** because NCBI's XSD declares the
  release root but not the efetch root, and every committed input is an
  efetch response. `schema/ClinVarResult-Set.xsd` includes NCBI's schema
  unchanged and adds that one element.
- **Five committed inputs**: the four conformance oracles, whose fetch date
  and method were never recorded and whose crate entries say so, and NCBI's
  official sample, the one input whose provenance is unimpeachable, committed
  input-only. Two of the four oracle file names do not describe the record
  inside (a BRCA1 variant named BRCA2, a VMA21 variant named MLH1); the names
  are conformance's, kept for traceability, and the crate and cases state the
  actual record.

## Verification

There is no test suite here by design. What can be checked before a Bridge
exists, with generic tools:

- every YAML and JSON file parses, and the two manifests validate against
  `schema/manifest/*.schema.json`;
- the crate validates with the RO-Crate validator
  (`rocrate-validator validate fixtures/ --profile-identifier ro-crate-1.2`);
- every `fixtures/in/*.xml` validates against `schema/ClinVarResult-Set.xsd`;
- every `fixtures/expected/*.ttl` parses as Turtle;
- every digest in the crate matches its file, and the XSD's md5 matches
  NCBI's published `.md5`;
- the four oracle triplets are byte-identical to
  `conformance/fixtures/genomics/clinvar/` at `0ea48bb`;
- opening the repository in VS Code with the recommended extensions shows
  XSD validation on a fixture input and schema completion in `adapter.yaml`.

The real verification is phase 2: `bridge-engine-java` runs `cases.yaml` and
the four graphs compare isomorphic, which is the moment the adapter is proven
rather than described.

## Related repositories

- [spec](https://github.com/the-cascade-protocol/spec): the vocabularies,
  pinned at `e77ba5e`; the RFC (spec#43) and the identity RFC (spec#38).
- [conformance](https://github.com/the-cascade-protocol/conformance):
  `fixtures/genomics/clinvar/`, the source of the four oracles.
- [cascade-cli](https://github.com/the-cascade-protocol/cascade-cli):
  `src/lib/clinvar-converter/`, the converter this adapter re-expresses, and
  `tests/clinvar-conformance.test.ts`, the comparison the cases file restates.
- `bridge-engine-java`, `bridge-engine-browser`: the Bridges that will run
  this adapter; not yet created.

## Licence

Apache-2.0 for the package. ClinVar data is NCBI's, under
https://www.ncbi.nlm.nih.gov/home/about/policies/; each committed NCBI-derived
file's entry in `fixtures/ro-crate-metadata.json` says so.
