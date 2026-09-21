# fixtures — Agent Context

The cases a Bridge's harness runs, and the inputs and expected outputs they name.
Nothing here executes; `manifest.ttl` declares the cases and the rule each is
judged by, and the crate records where every file came from.

## Every byte is recorded, and inputs are never edited

Every file under `in/`, `expected/` and `findings/` has its digest in the crate,
and `.gitattributes` and `.editorconfig` protect them from normalisation. A
change updates the crate's digest, size and description in the same commit.

An input and an expected graph come from `../../conformance`,
`fixtures/genomics/clinvar/` at the commit the crate names: `X.input.xml`
becomes `in/X.xml`, and `X.expected.ttl` becomes `expected/X.ttl`. An input
stays a byte-for-byte copy. An expected file is corrected here where the
converter that produced it was wrong. A findings file is not a copy of anything:
it is a Bridge's own `convert --findings` output over the input beside it,
committed unedited.

Two of the four file names do not describe the record inside; the per-entry
`rdfs:comment` in `manifest.ttl` says which. The names are the conformance
repository's and are kept so each copy stays traceable.

## Big datasets are referenced, not committed

A multi-gigabyte release is a URL, a version, a digest and a licence in the
crate, plus a `bridge:DatasetCompletionTest` naming that entity by its IRI.

## Checks CI cannot run

- `../in/sparql/detect.rq`, the crate's `bridge:detectQuery`, is true for each
  input. Run the ASK, in any SPARQL 1.1 engine, over the document's envelope
  skeleton, which the specification's `engine/sparql.md` defines. It must also
  be true for
  `<ClinVarResult-Set><set/></ClinVarResult-Set>`, the response efetch returns
  when a query matched no record, and false for a document under a known root
  holding neither.
- Each `sameAs` copy is byte-identical to conformance. Compare against the
  **blobs**, never the worktree: a clone with `core.autocrlf=true` holds those
  files as CRLF, so `cmp` on the worktree reports every input as differing at
  byte 40, and "fixing" that would break the recorded digests.

  ```
  git -C ../../conformance show <commit>:<path> | cmp - in/X.xml
  ```
