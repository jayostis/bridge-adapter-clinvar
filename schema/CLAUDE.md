# schema — Agent Context

`ClinVar_VCV_2.6.xsd` is NCBI's, pinned byte for byte. It is a verbatim copy: to
change it, replace it from NCBI and update the crate's digest, size and
description in the same commit. Its md5 in the crate is NCBI's own, published
beside the file; re-verify against `ftp.ncbi.nlm.nih.gov` before moving the pin,
and record the date. `../docs/format.md` says why 2.6 and not 2.5.

`ClinVarResult-Set.xsd` is this repository's, and the only schema here that may
be edited. NCBI's XSD declares the records but not the efetch envelope that
wraps them, so a whole efetch response cannot be validated against it. The
wrapper includes NCBI's and declares that root: either one or more
`VariationArchive` elements, or the single empty `set` element efetch returns
when a query matched no record, which is a response and not an error.
