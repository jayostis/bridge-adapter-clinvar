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
2. Routes an input here when the detect XPath,
   `exists(/ClinVarResult-Set/(VariationArchive|set)) or exists(/ClinVarVariationRelease/VariationArchive)`,
   is true: an efetch envelope holding records or the empty `set` efetch
   returns when a query matched none, or a release envelope holding records.
3. Splits the document on `VariationArchive` and validates each unit against
   `schema/ClinVar_VCV_2.6.xsd`. A unit that fails is a finding; it still goes
   through.
4. Runs the mapping (phase 2) on each unit, links the interpretation and
   submitter-assertion records to their Variant, stamps provenance, checks
   every predicate against the pinned vocabularies, validates with SHACL,
   and hands the graph and findings to the runtime.
5. In test, executes `fixtures/manifest.ttl`: for each
   `bridge:IsomorphicConversionTest`, compares the produced graph with the
   expected one up to blank-node relabelling with the stamp predicates
   removed, and the produced findings with the expected sidecar as a multiset -
   the same entries with the same multiplicity, in any order. For
   each `bridge:DatasetCompletionTest`, streams the referenced release and
   records the record count and output digest.

`docs/stages.md` names each of those stages with its Enterprise Integration
Pattern; `docs/format.md` describes the format the first three stages see.

## Layout

```
cascade-bridge-adapter-clinvar/
  README.md                  this file
  LICENSE                    Apache-2.0
  CHANGELOG.md
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
    stages.md                the RFC's engine stages as Enterprise Integration Patterns
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

The reasoning is in [issue #1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1);
the outcomes:

- **Standards, not inventions.** Enterprise Integration Patterns for stage
  names; XSLT 3, XSD and Schematron for the map and validation; RO-Crate 1.2
  for the manifest and provenance; BagIt-style digests for inputs; W3C's
  test-manifest vocabulary (`mf:`) for the cases and SHACL for their shape;
  the RFC's section 6 layout; the conformance repository's input / expected /
  gaps triplet as the fixture shape. The reason is the editor: every one of
  these already has tooling, so the repository lights up in VS Code without
  anyone writing a plugin, and every later choice is weighed against whether
  the standard editor still understands it.
- **The adapter manifest is the RO-Crate, at the root.** The RFC's section 6
  sketch names an `adapter.yaml`; this adapter has none. Everything the
  adapter says about itself is in one graph, `ro-crate-metadata.json`, whose
  root entity is the adapter (`Dataset` and `bridge:Adapter`): schema.org
  terms where they exist (`identifier`, `name`, `version`, `license`),
  Dublin Core's `conformsTo` (the key the RO-Crate context expands to
  `dcterms:conformsTo`, not a schema.org term), Bridge terms where neither
  has one (profiles, vocabulary pin and vocabularies, source media type and
  schema, envelopes, unit, detect rule, test manifest, and the tables and
  extension vocabulary phase 2 adds). No tier: the adapter requires the
  `xslt-3` profile, and which tier that makes it is measured, not declared
  (RFC section 11); phase 3's SPARQL CONSTRUCT rewrite is the experiment
  that decides whether XSLT 3 is Core or a profile. The licence is the SPDX entity, the pin is
  the commit, the vocabularies are their namespaces, the envelopes are
  entities the test manifest refers to by IRI, and the test manifest points
  back at the root, so every reference that used to be a string is a link
  SHACL can check. Phase 2 has a published profile waiting: Workflow
  RO-Crate, whose `mainEntity` is the transformation with its language
  declared. The cost is stated: JSON-LD is less pleasant to hand-edit than
  YAML and has no field completion; the RO-Crate validator and the shapes
  are the checks instead. That the manifest should be an RO-Crate and not a
  YAML dialect is a finding to report on spec#43.
- **The detect rule is one XPath expression**,
  `exists(/ClinVarResult-Set/(VariationArchive|set)) or exists(/ClinVarVariationRelease/VariationArchive)`,
  an XPath 3.1 boolean a Bridge's content-based router evaluates against the
  document, in place of a two-field root-element / contains rule of the
  project's own. It keys on an envelope holding something the adapter
  recognises rather than on the root element alone, so a response in another
  shape under the same root is not claimed; the `set` branch is there because
  an efetch response that matched no record is a response, not an error
  (`docs/format.md`).
- **The cases are a W3C-style test manifest in Turtle**,
  `fixtures/manifest.ttl`, not a YAML dialect of the project's own. Every W3C
  RDF-family test suite is an `mf:` manifest whose entry type carries the
  comparison rule; the test terms of the `bridge:` vocabulary on top of it
  live in the Cascade Bridge Specification, which this adapter seeded. The
  manifest and the crate load as one graph, so a dataset test names the
  release by the crate's own IRI, an action names its envelope by the crate's
  entity, and a query walks from a test to its input's digest and licence;
  validation is SHACL, the project's own language; and every RDF engine reads the file
  identically, which is the property the "same adapter, every runtime" claim
  rests on. One contract, not a menu: an adapter does not choose among test
  conventions.
- **Naming is deferred, and the layout does not care.** The converter mints
  record IRIs by SHA-1 over `genomics:Variant:clinvar:VCV…`; XSLT 3 has no
  hash function. Whether the mapping writes the recipe and the Bridge mints
  the name (recommended, and where [spec#38](https://github.com/the-cascade-protocol/spec/issues/38#issuecomment-5555482906)
  puts naming) or calls a Bridge-provided function is decided in phase 2.
- **Licence is Apache-2.0**, what conformance and cascade-cli use, because
  XSLT executes and a content licence sits awkwardly on files that execute.
  NCBI's terms for ClinVar data are recorded per file in the crate.
- **Schema pin is 2.6, not the 2.5 the issue named.** The 2026-09 monthly
  release declares 2.6, which appeared in NCBI's index in February 2026; the
  difference is one optional attribute. `docs/format.md` has the table.
- **An envelope wrapper schema exists** because NCBI's XSD declares the
  release root but not the efetch root, and every committed input is an
  efetch response. `schema/ClinVarResult-Set.xsd` includes NCBI's schema
  unchanged and adds that one element; the crate's `#envelope-efetch` names
  it as the envelope's document schema.
- **The specification is pinned, and the pin is checked.** The crate's
  `bridge:specPin` names a commit of
  [cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec), and
  `conformsTo` names the adapter profile IRI beside the RFC: the profile is the
  machine-checkable claim a validator tests, the RFC is the standard being
  implemented, and RO-Crate permits both. `.github/workflows/validate.yml`
  calls that repository's published lint at the tag pointing at the same
  commit, so the pin the reader sees and the pin the machine uses are held to
  agreeing. Until 0.2.0 this adapter carried its own copies of the `bridge:`
  vocabulary and SHACL shapes under `schema/manifest/`; they were the seed of
  that repository, and once it existed a copy here was a second statement of a
  contract, which is a statement that can disagree with it. They are deleted.
- **A lint is not a test run.** "No tests here" stands: executing this
  adapter's fixtures is a Bridge's job, and no Bridge exists. Checking that the
  package is well formed is a lint, it belongs where the package lives, and it
  is one `uses:` line.
- **Five committed inputs**: the four conformance oracles, whose fetch date
  and method were never recorded and whose crate entries say so, and NCBI's
  official sample, the one input whose provenance is unimpeachable, committed
  input-only. Two of the four oracle file names do not describe the record
  inside (a BRCA1 variant named BRCA2, a VMA21 variant named MLH1); the names
  are conformance's, kept for traceability, and the crate and the test
  manifest state the actual record.

## Verification

There is no test suite here by design, and there is still nothing here that
executes an adapter's fixtures: that is a Bridge's job. What can be checked
before a Bridge exists is a lint, and most of it now runs in CI.

**CI runs three checks on every pull request**, by calling the
specification's published lint: the crate is a valid RO-Crate 1.2; the crate
and `fixtures/manifest.ttl`, loaded as one graph with their own locations as
base, conform to that repository's SHACL shapes at the pinned commit, which
also holds the crate's `bridge:specPin` to naming the revision the workflow
calls; and every git-tracked file is either described by the crate with a
declared `encodingFormat` in the allowed set, or is one of the four repository
documents, `ro-crate-metadata.json` or a dotfile. That last check is what makes
"no code here" measured rather than claimed.

The shapes carry more than the cardinalities: every IRI naming a committed file
— `bridge:input`, `bridge:graph`, `bridge:findings`, `bridge:sourceSchema` and
each `bridge:documentSchema` — must be a crate `File` entity, so a typo that
names nothing fails rather than passing quietly, and the links between the crate
and the manifest are checked in both directions.

What CI does not do, and a contributor should still run before pushing, because
SHACL cannot see the filesystem and none of it needs a Bridge:

- that each crate `File` the shapes accept exists on disk;
- every `fixtures/in/*.xml` validates against `schema/ClinVarResult-Set.xsd`,
  and the detect XPath is true for each of them and for the root of the
  monthly release;
- every `fixtures/expected/*.ttl` parses as Turtle;
- every digest in the crate matches its file, and the XSD's md5 matches
  NCBI's published `.md5`;
- the four oracle triplets are byte-identical to
  `conformance/fixtures/genomics/clinvar/` at `0ea48bb`, compared against
  the blobs (`git hash-object` here against
  `git -C ../conformance ls-tree 0ea48bb fixtures/genomics/clinvar/`, or
  `git -C ../conformance show 0ea48bb:<path> | cmp - <file>`), never against
  the sibling checkout's worktree, whose files a `core.autocrlf=true` clone
  holds as CRLF;
- opening the repository in VS Code with the recommended extensions shows
  XSD validation on a fixture input and Turtle support in the manifest.

The real verification is phase 2: `bridge-engine-java` runs `manifest.ttl` and
the four graphs compare isomorphic, which is the moment the adapter is proven
rather than described.

## Related repositories

- [spec](https://github.com/the-cascade-protocol/spec): the vocabularies;
  the RFC (spec#43) and the identity RFC (spec#38). The pin,
  [`e77ba5e`](https://github.com/jayostis/spec/commit/e77ba5e8004bf57f34d42a1bee69d1cfb95f86e3),
  is on the fork `jayostis/spec` and not yet on the org's `main`; the crate's
  pin entity says why, and the crate links the fork for that reason.
- [conformance](https://github.com/the-cascade-protocol/conformance):
  `fixtures/genomics/clinvar/`, the source of the four oracles, at
  [`0ea48bb`](https://github.com/jayostis/conformance/tree/0ea48bb1c044ac9bf8086d456601357135cf79e6/fixtures/genomics/clinvar),
  likewise a commit on the fork `jayostis/conformance`; the files last
  changed at `8a5e203`, which is on the org's `main` with identical blobs.
- [cascade-cli](https://github.com/the-cascade-protocol/cascade-cli):
  `src/lib/clinvar-converter/`, the converter this adapter re-expresses, and
  `tests/clinvar-conformance.test.ts`, the comparison the test manifest restates.
- `bridge-engine-java`, `bridge-engine-browser`: the Bridges that will run
  this adapter; not yet created.

## Licence

Apache-2.0 for the package. ClinVar data is NCBI's, under
https://www.ncbi.nlm.nih.gov/home/about/policies/; each committed NCBI-derived
file's entry in `ro-crate-metadata.json` says so.
