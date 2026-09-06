# Engine stages, in the vocabulary integration engines already use

Section 5 of the Cascade Bridge RFC
([spec#43](https://github.com/the-cascade-protocol/spec/issues/43)) lists the
stages a Bridge runs *around* an adapter's mapping. Each is an established
Enterprise Integration Pattern (Hohpe and Woolf, 2003), so the design reads in
the terms every integration engine, Apache Camel included, already documents.
The third column says what this adapter contributes to the stage, and in which
phase.

| RFC stage | Enterprise Integration Pattern | this adapter |
|---|---|---|
| read and chunk | **Splitter**, on `VariationArchive` | declares the unit (`source.unit` in `adapter.yaml`, phase 1) |
| detect and route | **Content-Based Router** | declares the detect rule (`detect`, phase 1) |
| transform | **Message Translator** | the XSLT under `in/xslt/` (phase 2); the same mapping as SPARQL CONSTRUCT under `in/sparql/` (phase 3) |
| Cascade RDF as target | **Canonical Data Model** | writes it, in the vocabularies `adapter.yaml` pins |
| link within the batch | **Aggregator** | the interpretation (RCV) and submitter assertion (SCV) records link to their Variant; the mapping emits the links, the Bridge resolves them within one import |
| stamp | **Message History** | receives it: `cascade:dataProvenance`, `cascade:schemaVersion`, source identity, import time are added by the Bridge, and `fixtures/manifest.ttl` ignores them (`bt:ignorePredicate`) when comparing |
| check, validate | **Message Validator**, findings to an **Invalid Message Channel** | declares the XSD every unit is validated against (`source.schema`, phase 1) |
| findings | **Dead Letter Channel** / **Invalid Message Channel** | the findings sidecar, `fixtures/findings/*.gaps.json`, is the expected content of that channel |
| re-import as no-op | **Idempotent Receiver** | follows from input-derived names, whichever way naming is decided in phase 2 |
| vendor quirks | **Normalizer** | none: ClinVar has one publisher and no vendor dialects, so the adapter has no `quirks/` directory |

## Notes on the mapping to patterns

**Splitter.** A ClinVar document is an envelope around one or more
`VariationArchive` elements (see `format.md`). The Bridge opens the envelope,
splits on the unit, and hands the mapping one unit at a time with its raw bytes
preserved. Because `VariationArchive` is a global element in NCBI's schema, a
unit validates on its own; the Bridge need not validate the multi-gigabyte
release as a single document.

**Content-Based Router.** The rule is the document element (one of the two
envelope roots) plus the presence of the unit. cascade-cli's detector matches
a different set of five roots (`ClinVarResult-Set`, `ReleaseSet`,
`ClinVarSet`, `VariationArchive`, `VariationReport`): three belong to older
or different ClinVar shapes, one is the bare unit, and `ClinVarVariationRelease`
is not among them, so the release envelope reaches this router untested by the
existing converter (`docs/format.md`, "The two envelopes"). The adapter
declares only the two roots NCBI publishes today, and a document in another
shape is routed elsewhere or reported, never guessed at.

**Message Translator and Canonical Data Model.** The mapping is the only
format-specific thing that runs, and it runs inside an engine the Bridge
already ships (Saxon for XSLT 3; a SPARQL engine over the Bridge's XML lift in
phase 3). The target is Cascade RDF in the pinned vocabularies, which is the
canonical model every adapter writes to and nothing else reads from the
adapter.

**Aggregator.** One `VariationArchive` yields one Variant, one interpretation
per RCV accession and one submitter assertion per SCV. The latter two point at
the Variant. How the pointer is expressed (a blank node the Bridge resolves, or
a name minted by the rule spec#38 settles on) is the phase 2 naming decision;
the pattern is the same either way.

**Message History.** The stamp is the Bridge's, not the adapter's. This is why
`fixtures/manifest.ttl` declares `bt:ignorePredicate` on each
`bt:IsomorphicConversionTest`: the oracles were produced by
cascade-cli, which stamps `cascade:dataProvenance` and `cascade:schemaVersion`
itself, and a Bridge stamps its own, so the comparison removes them from both
sides.

**Message Validator and Invalid Message Channel.** D-OPENWORLD-1, restated by
RFC section 8: validation reports; it never refuses and never destroys. A unit
that fails the XSD is a finding about the input, and the unit still goes
through. Findings from source validation, the mapping, the undeclared-predicate
check and SHACL all land in the Bridge's one findings model; the gaps sidecar
this adapter expects is the mapping's contribution to it.

**Idempotent Receiver.** Importing the same VCV twice must change nothing. The
converter today achieves this by deriving each record's name from its
accession; the RFC's stamp stage and spec#38 put that rule in the Bridge. The
adapter's obligation is only that everything it emits is a function of the
input, which XSLT guarantees by construction.

**Normalizer.** Formats with several publishers (C-CDA from Epic and Cerner)
need a normalising pass per vendor before the translator. ClinVar is published
by NCBI alone, so this adapter has no such pass and declares none.
