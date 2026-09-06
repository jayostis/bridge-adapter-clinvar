# ClinVar VCV XML, as the adapter sees it

Everything below was read from `ftp.ncbi.nlm.nih.gov` on 2026-09-06 unless a
line says otherwise. NCBI's own descriptions are in
`pub/clinvar/xml/_README` (the release directory) and
`pub/clinvar/xsd_public/README.txt` (the schemas). ClinVar's data terms are
NCBI's, https://www.ncbi.nlm.nih.gov/home/about/policies/, and questions go to
clinvar@ncbi.nlm.nih.gov.

## What a record is

ClinVar aggregates submitted classifications of human genetic variants. The
**VCV** accession (`VCV000017661`) names one *variation* and everything ClinVar
holds about it. A VCV record is a `VariationArchive` element, and that element
is the unit this adapter declares: a Bridge splits every document on it and
hands the mapping one at a time.

`VariationArchive` carries, as attributes, `Accession`, `Version`,
`VariationID`, `VariationName`, `VariationType`, `RecordType`,
`DateCreated`, `DateLastUpdated`, `MostRecentSubmission`,
`NumberOfSubmissions` and `NumberOfSubmitters`. For example the BRCA1 fixture
is `Accession="VCV000017661" Version="157" DateLastUpdated="2026-05-03"`. The
crate records `Version` and `DateLastUpdated` for every committed input.

Inside it, one of:

- `ClassifiedRecord` (`RecordType="classified"`), the common case, holding
  - `SimpleAllele` (or `Haplotype` / `Genotype` for compound records): the
    variant itself, with gene, location, HGVS expressions, cross-references
    and consequences. The adapter's **Variant** class.
  - `RCVList`: one `RCVAccession` per variant-condition pair, with the
    aggregate classification and review status. The adapter's
    **interpretation** class (one record per RCV).
  - `ClinicalAssertionList`: one `ClinicalAssertion` per SCV accession, the
    submitter's own classification, evidence and dates. The adapter's
    **submitter assertion** class (one record per SCV).
  - `Classifications`: the germline, somatic clinical-impact and oncogenicity
    aggregates. Since XSD 2.0 (August 2025) these are separate elements; the
    retired format had one.
- `IncludedRecord` (`RecordType="included"`): a variant that appears only as
  part of a haplotype or genotype, with a `SimpleAllele` and no
  classifications of its own.

cascade-cli's converter (`src/lib/clinvar-converter/`, about 2,200 lines)
handles these as `simple-allele.ts`, `rcv-interpretation.ts` and
`scv-submitter-assertion.ts`, plus a seven-tier review-status table in
`review-status-map.ts`. Phase 2 re-expresses each as an XSLT module.

## The two envelopes

A `VariationArchive` never arrives alone. It is wrapped in one of two roots,
and the adapter declares both as entities in `ro-crate-metadata.json`, which
the test manifest refers to by IRI:

| envelope (crate entity) | root element | where it comes from | schema location declared |
|---|---|---|---|
| `efetch` (`#envelope-efetch`) | `ClinVarResult-Set` | E-utilities: `efetch.fcgi?db=clinvar&rettype=vcv&is_variationid&id=<VariationID>` | none |
| `release` (`#envelope-release`) | `ClinVarVariationRelease` (attribute `ReleaseDate`) | the monthly and weekly release files | `xsi:noNamespaceSchemaLocation` pointing at the XSD on the FTP site |

All five committed inputs are `efetch` envelopes. The 2026-09 monthly release,
read from its first 256 KB, opens
`<ClinVarVariationRelease ... xsi:noNamespaceSchemaLocation="http://ftp.ncbi.nlm.nih.gov/pub/clinvar/xsd_public/ClinVar_VCV_2.6.xsd" ReleaseDate="2026-08-29">`.

One naming trap. NCBI's release README calls `ClinVarVariationRelease` "the
old format for VCV XML", retired 7 August 2025, and the current files
`ClinVarVCVRelease`. That refers to the *file name family*: the current files
are `ClinVarVCVRelease_YYYY-MM.xml.gz`, the retired ones were
`ClinVarVariationRelease_YYYY-MM.xml.gz` under `VCV_xml_old_format/`. The
*root element* of the current files is still `ClinVarVariationRelease`, as
the schema declares and the release file confirms. The adapter names the
element, not the file.

cascade-cli's detector (`clinvar-converter/detect.ts`) matches exactly five
roots: `ClinVarResult-Set`, `ReleaseSet`, `ClinVarSet`, `VariationArchive`
and `VariationReport`. That set is neither a superset nor a subset of the
adapter's two envelopes. It does not accept `ClinVarVariationRelease`, so the
existing converter has never read a release file: the release envelope is
exercised for the first time by the Bridge, whose router is the first thing
that will read `ClinVarVCVRelease_YYYY-MM.xml.gz`. A bare `VariationArchive`
root, which the cli does accept, is the unit here and not an envelope.
`ReleaseSet`, `ClinVarSet` and `VariationReport` are RCV-era and legacy
shapes and are out of scope; RCV XML lives under `RCV_release/` with its own
XSD under `xsd_public/RCV/`.

## The schema, and why 2.6 is pinned

NCBI publishes the VCV schema history under `pub/clinvar/xsd_public/`:

| file | version attribute | md5 (NCBI's) | Last-Modified |
|---|---|---|---|
| `ClinVar_VCV_2.0.xsd` to `ClinVar_VCV_2.4.xsd` | the lineage since the August 2025 format change | published beside each | |
| `ClinVar_VCV_2.5.xsd` | `2.5; August 6, 2025` | `169e958e2f06c6a3ba8a96aae3a949a9` | 2025-08-07 |
| `ClinVar_VCV_2.6.xsd` | `2.6; February 26, 2026` | `a7b65e5a166dc5f36a7eea9127d56f4e` | 2026-02-27 |
| `ClinVar_VCV.xsd`, `ClinVar_VCV_weekly.xsd` | same bytes as 2.6 | `a7b65e5a166dc5f36a7eea9127d56f4e` | 2026-02-27 |
| `ClinVar_VCV_2.4_old_vs_2.5_new_xsd_diff.html` | NCBI's own diff for the previous step | | |

The pinned copy is **2.6**, at `schema/ClinVar_VCV_2.6.xsd`, byte for byte,
its md5 checked against NCBI's `.md5`. Issue #1 named 2.5, from a listing that
predated 2.6's appearance in the index; the 2026-09 release declares 2.6, so
pinning 2.5 would put the referenced dataset and the pinned schema out of
step. The whole difference between the two is one optional attribute,
`derived_by` (enumerated to `NCBI`), on `SetElementSetType`, the type behind
the `Name`-and-`XRef` sets. All five committed inputs validate against both.

The XSD declares exactly one document root, `ClinVarVariationRelease`, and
declares `VariationArchive`, `ClassifiedRecord` and `IncludedRecord` as global
elements too. It does not declare `ClinVarResult-Set`. So:

- a release file validates as a document against the XSD;
- an efetch response does not, at its root, though every `VariationArchive`
  inside it does;
- `schema/ClinVarResult-Set.xsd` includes NCBI's XSD unchanged and adds that
  one root, so that whole efetch responses validate too; the crate names it
  as `#envelope-efetch`'s `bridge:documentSchema`, and
  `.vscode/settings.json` associates `fixtures/in/*.xml` with it.

XSD 1.0 is enough; the schema uses nothing from XSD 1.1.

**An efetch response that matched nothing is still a response.** Asked for an
id no record carries, efetch answers
`<ClinVarResult-Set><set/></ClinVarResult-Set>`: an empty `set` element in
place of the records, not an error and not an empty root. Observed on
2026-09-06 for three such queries, with and without `is_variationid`. The
wrapper schema therefore allows the root to hold either one or more
`VariationArchive` elements or a single empty `set`, so a Bridge validating a
whole response accepts this one and yields zero units. Rejecting it would
report a finding against NCBI for answering truthfully. Anything else inside
the root, including a `set` carrying content, is still rejected.

## Releases

Under `pub/clinvar/xml/`:

- **Monthly**: `ClinVarVCVRelease_YYYY-MM.xml.gz`, generated the first
  Thursday of the month, with a sibling `.md5`. `ClinVarVCVRelease_00-latest.xml.gz`
  is a symlink to the newest one and is never the reference; the crate names
  the dated file. The 2026-09 release is 5,849,397,646 bytes gzipped,
  md5 `5da42f3becdd8e9a90cd46b1727c08b8`, Last-Modified 2026-09-03T04:07:26Z,
  `ReleaseDate="2026-08-29"`. Releases back to 2025-01 sit beside it; earlier
  years are under `archive/`.
- **Weekly**: `weekly_release/ClinVarVCVRelease_YYYY-MMDD.xml.gz`, generated
  as the website updates, with the directory cleared when the monthly release
  is generated. On 2026-09-06 it held only the symlink
  `ClinVarVCVRelease_00-latest_weekly.xml.gz -> ../ClinVarVCVRelease_2026-09.xml.gz`
  and its `.md5`. Resolve the symlink (FTP `LIST` shows the target; HTTPS
  does not) and record the dated name before a run. The crate lists no
  weekly entity and the test manifest no weekly test until a dated file
  exists to record: an entity that only resolves to the monthly would be a
  second pass over the same bytes.
- **Sample**: `sample_xml/VCV_XML_VCV000091629.xml`, NCBI's one published
  efetch example, 193,155 bytes, committed as `fixtures/in/VCV_XML_VCV000091629.xml`.
  NCBI publishes no `.md5` for it. The directory also holds
  `document_summary_VCV000091629.xml`, an esummary docsum, which is not VCV
  XML and not an input.

## The committed inputs, and what is not known about them

The four oracle inputs were copied from
`conformance/fixtures/genomics/clinvar/` at `0ea48bb` (the files last changed
at `8a5e203`, 2026-05-06). They are efetch envelopes with no schema location.
The fetch date and query were not recorded when they were captured, and an
efetch of `VCV000208804` on 2026-09-06 returned the same `Version` and
`DateLastUpdated` but a different byte count, so the captures are not
reproducible from NCBI today. The crate records what the XML itself says
(accession, `Version`, `DateLastUpdated`), the conformance commit, the digest,
and that the rest is unrecorded.

Two of the four file names, which are conformance's and are kept so the copies
stay traceable, do not describe the record inside:

| file | the record | note |
|---|---|---|
| `VCV000007105-CFTR-deltaF508` | `NM_000492.3(CFTR):c.1521_1523del (p.Phe508del)`, v206 | as named |
| `VCV000017661-BRCA1` | `NM_007294.4(BRCA1):c.181T>G (p.Cys61Gly)`, v157 | as named |
| `VCV000055448-BRCA2-pathogenic` | `NM_007294.4(BRCA1):c.5193+1G>C`, v15 | BRCA1, not BRCA2 |
| `VCV000208804-MLH1-LynchSyndrome` | `NM_001017980.4(VMA21):c.164-6T>G`, v2 | VMA21 interpreted for MONDO:0010684, not MLH1 |

`manifest.ttl` and the crate state the actual record for each.

## What the mapping will read (phase 2)

For orientation only; the mapping is not written yet.

- Variant: `SimpleAllele` attributes and children (`GeneList/Gene`,
  `Location/SequenceLocation`, `HGVSlist`, `XRefList`, `ProteinChange`,
  `FunctionalConsequence`), plus `VariationArchive/@Accession` as the source
  id.
- Interpretation: each `RCVList/RCVAccession` with its `ClassifiedConditionList`
  and `RCVClassifications` (review status, description, dates).
- Submitter assertion: each `ClinicalAssertionList/ClinicalAssertion` with its
  `ClinVarAccession`, `Classification`, `ClinVarSubmissionID`, submitter and
  dates.
- The review-status strings map through a nine-row table onto seven tiers
  (`tables/review-status.csv`, from `review-status-map.ts`: three phrasings
  of "criteria provided, conflicting ..." that ClinVar has used over time all
  map to `genomics:ConflictingSubmissions`); everything with no Cascade term
  is a findings entry or an extension term in `vocab/`.
