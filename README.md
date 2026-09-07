# cascade-bridge-adapter-clinvar

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

It is written against a **pinned revision of the Cascade Bridge Specification**,
[cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec), named by
`bridge:specPin` in the crate, and it validates itself against that revision in
its own CI. The specification does not know this adapter exists; the arrow runs
this way and only this way.

## Status

**Phase 1, version 0.2.0: the adapter laid out, and checked against the
specification it pins.** Manifest, pinned schema, fixtures with recorded
provenance, references to the large datasets, editor configuration, and a
workflow that calls the specification's published lint. No mapping, no runner.

| phase | what | where | done when |
|---|---|---|---|
| 1 | this layout | here ([#1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1)) | it opens in VS Code with everything validating |
| 2 | the first mapping (XSLT 3, one module per record class) and the engine that runs it | here and `bridge-engine-java` (Camel YAML route, Saxon-HE, riot) | the four committed cases compare isomorphic |
| 3 | the same mapping as SPARQL CONSTRUCT over the Bridge's XML lift | `in/sparql/` | the two graphs are diffed; the result is a finding for spec#43 |
| 4 | the same adapter, byte for byte, through JavaScript engines | `bridge-engine-browser` | the same `manifest.ttl` passes in both |

## How a Bridge runs it

1. Reads `ro-crate-metadata.json`, the adapter's manifest and its provenance
   record in one RO-Crate. The root entity is the adapter: format id
   `clinvar`, the two envelopes (`#envelope-efetch`, `#envelope-release`),
   the unit `VariationArchive`, the detect rule, the `xslt-3` profile it
   must offer, the vocabulary pin, and the test manifest, every one of them
   a link to an entity in the same graph.
2. Routes an input here when the crate's detect rule, one XPath 3.1 boolean, is
   true of it: an efetch envelope holding records, the empty `set` efetch
   returns when a query matched none, or a release envelope. `docs/format.md`
   has the expression and which roots were deliberately not claimed.
3. Splits the document on `VariationArchive` and validates each unit against
   `schema/ClinVar_VCV_2.6.xsd`. A unit that fails is a finding; it still goes
   through.
4. Runs the mapping (phase 2) on each unit, links the interpretation and
   submitter-assertion records to their Variant, stamps provenance, checks
   every predicate against the pinned vocabularies, validates with SHACL,
   and hands the graph and findings to the runtime.
5. In test, executes `fixtures/manifest.ttl`. Each entry's type carries how it
   is judged, and the rule is the `rdfs:comment` on that type in the
   specification's vocabulary, not anything this repository states.

Those stages are the Bridge's, not this adapter's; the specification's
[`docs/engine/stages.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/docs/engine/stages.md)
names each with its Enterprise Integration Pattern. `docs/format.md` describes
the format the first three stages see.

## Layout

```
cascade-bridge-adapter-clinvar/
  README.md                  this file
  LICENSE                    Apache-2.0
  CLAUDE.md                  agent context: the rules and the sibling checkouts
  ro-crate-metadata.json     the adapter manifest, and provenance for every committed fixture, schema and document, and every remote dataset (RO-Crate 1.2)
  .gitattributes             LF everywhere; verbatim copies never normalised
  .editorconfig
  .github/workflows/
    validate.yml             calls the specification's lint at the pinned tag; no logic of its own
  .vscode/
    extensions.json          XML, XSLT/XPath, Turtle, EditorConfig, Kaoto
    settings.json            XSD association for fixtures/in
  docs/
    format.md                ClinVar VCV XML as the adapter sees it
  schema/
    ClinVar_VCV_2.6.xsd      NCBI's schema, pinned byte for byte (md5 a7b65e5a166dc5f36a7eea9127d56f4e)
    ClinVarResult-Set.xsd    the efetch envelope root NCBI's schema does not declare; includes the above
  fixtures/
    manifest.ttl             the test manifest: the cases and how to judge each
    in/                      four conformance inputs and NCBI's official sample
    expected/                the four expected graphs, byte-identical to conformance
    findings/                the four expected gaps sidecars, byte-identical to conformance
  in/xslt/                   phase 2: clinvar.xsl entry, one module per record class, findings.xsl
  in/sparql/                 phase 3: the same mapping as CONSTRUCT over the Bridge's XML lift
  tables/                    phase 2: review-status.csv, from cascade-cli's review-status-map.ts
  vocab/                     phase 2: clinvar-ext.ttl, the adapter's namespace for unmapped values
```

The four directories marked phase 2 or 3 do not exist yet; they are named
here so the shape is visible. Phase 2 adds them to the crate: the XSLT as
the crate's `mainEntity` (Workflow RO-Crate), the tables as `bridge:table`,
the extension vocabulary as `bridge:extensionVocabulary`.

## Decisions

The decisions that shaped this adapter — the manifest being an RO-Crate rather
than the RFC's `adapter.yaml`, the cases being a W3C `mf:` test manifest, the
detect rule being one XPath expression, standards over inventions — are not
restated here. Each is a property of the **adapter package format**, and the
authority for that format is the Cascade Bridge Specification:
[`docs/adapter/manifest.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/docs/adapter/manifest.md),
[`docs/adapter/test-manifest.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/docs/adapter/test-manifest.md)
and [`docs/pinning.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/docs/pinning.md).

What is specific to ClinVar rather than to adapters in general is in
[`docs/format.md`](docs/format.md): why the schema pin is 2.6, why an envelope
wrapper schema exists, and what is and is not known about the committed inputs.
Why this adapter is laid out the way it is, and the phases, are in
[issue #1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1).

Naming is the one decision still open: how a record's IRI is minted is settled
in phase 2, on the basis in [spec#38](https://github.com/the-cascade-protocol/spec/issues/38#issuecomment-5555482906).

## Verification

This package must conform to the Cascade Bridge Specification at the revision
its crate pins. `.github/workflows/validate.yml` calls that specification's
published lint at the matching tag on every pull request; what it checks, and
what a failure means, is
[`docs/adapter/validation.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/docs/adapter/validation.md)
there. Failures come out of the build.

A few things a lint cannot see — that a file exists on disk, that an input
validates against its schema, that the oracle copies are byte-identical to
conformance — are run by hand before pushing; `fixtures/CLAUDE.md` and
`schema/CLAUDE.md` say how.

There is no test suite here by design. Executing an adapter's fixtures is a
Bridge's job, and no Bridge exists yet: phase 2 is when this adapter is proven
rather than described.

## Related repositories

Which commit of each is pinned, and why a pin points at a fork where it does, is
recorded in `ro-crate-metadata.json`. It is not repeated here.

- [cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec): the
  contract this adapter is written against, and the lint its CI calls.
- [spec](https://github.com/the-cascade-protocol/spec): the Cascade
  vocabularies; the RFC ([#43](https://github.com/the-cascade-protocol/spec/issues/43))
  and the identity RFC ([#38](https://github.com/the-cascade-protocol/spec/issues/38)).
- [conformance](https://github.com/the-cascade-protocol/conformance):
  `fixtures/genomics/clinvar/`, the source of the four oracles.
- [cascade-cli](https://github.com/the-cascade-protocol/cascade-cli):
  `src/lib/clinvar-converter/`, the converter this adapter re-expresses as data.
- `bridge-engine-java`, `bridge-engine-browser`: the Bridges that will run this
  adapter; neither exists yet.

## Licence

Apache-2.0 for the package. ClinVar data is NCBI's, under
https://www.ncbi.nlm.nih.gov/home/about/policies/; each committed NCBI-derived
file's entry in `ro-crate-metadata.json` says so.
