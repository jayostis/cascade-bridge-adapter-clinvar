# cascade-bridge-adapter-clinvar

[![compatibility](https://github.com/jayostis/cascade-bridge-adapter-clinvar/actions/workflows/validate.yml/badge.svg?branch=main)](https://github.com/jayostis/cascade-bridge-adapter-clinvar/actions/workflows/validate.yml?query=branch%3Amain)

The **Cascade Bridge Adapter** for ClinVar VCV XML: the package of data a
**Cascade Bridge** runs to turn NCBI ClinVar variation records into Cascade
RDF. The pilot adapter: import-only, a published NCBI schema, no vendor quirks,
an existing converter of about 2,200 lines in cascade-cli to re-express as data,
and four conformance oracles already written.

The contract it is written against is the
[Cascade Bridge Specification](https://github.com/jayostis/cascade-bridge-spec).
That repository is the authority.

There is no code here and there will be none. An adapter is mappings, schemas,
fixtures and a manifest; the thing that runs it is a Bridge. Nothing in this
repository executes.

The specification does not know this adapter exists; the arrow runs this way and
only this way.

## Status

**Phase 2, version 0.4.0: the mapping, as SPARQL.**

| phase | what | where | done when |
|---|---|---|---|
| 1 | this layout | here ([#1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1)) | it opens in VS Code with everything validating |
| 2 | the mapping, as SPARQL 1.1 over the Bridge's XML lift, and the engine that runs it | `in/sparql/`, and `cascade-bridge-rs` | the four committed cases compare isomorphic |
| 3 | the same adapter, byte for byte, through the same engine in a browser | `cascade-bridge-rs` | the same `manifest.ttl` passes in both |

## How a Bridge runs it

1. Reads `ro-crate-metadata.json`, the adapter's manifest and its provenance
   record in one RO-Crate. The root entity is the adapter: format id
   `clinvar`, the two envelopes (`#envelope-efetch`, `#envelope-release`),
   the unit `VariationArchive`, the detect rule, the `sparql-1.1` profile it
   must offer, the vocabulary pin and the paths, in the repository it pins, of
   the ontology and shapes files the adapter is read against, the gap scheme
   the findings take their bodies from, the accounting of every path its source
   carries, and the test manifest.
2. Routes an input here when the detect rule, `in/sparql/detect.rq`, is true of
   the document's envelope skeleton; `docs/format.md` has which roots were
   deliberately not claimed.
3. Splits the document on `VariationArchive` and validates each unit against
   `schema/ClinVar_VCV_2.6.xsd`. A unit that fails is a finding; it still goes
   through.
4. Lifts each unit to RDF and runs the queries in `in/sparql/` over it: the
   mappings write the records, which name each other by the IRIs the queries
   mint, and the findings queries write Web Annotations on the source document.
   Then stamps provenance, checks every predicate against the pinned
   vocabularies, validates with SHACL, and hands the graph and findings to the
   runtime.
5. In test, executes `fixtures/manifest.ttl`. Each entry's type carries how it
   is judged, and the rule is the `rdfs:comment` on that type in the
   specification's vocabulary, not anything this repository states.

Those stages are the Bridge's, not this adapter's; the specification's
[`engine/stages.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/engine/stages.md)
names each with its Enterprise Integration Pattern. `docs/format.md` describes
the format the first three stages see.

## Layout

```
cascade-bridge-adapter-clinvar/
  README.md                  this file
  LICENSE                    Apache-2.0
  CLAUDE.md                  agent context
  compatibility.json         the Bridges that must pass this adapter
  ro-crate-metadata.json     the adapter manifest, and provenance for every committed fixture, schema and document, and every remote dataset (RO-Crate 1.2)
  .gitattributes             LF everywhere; verbatim copies never normalised
  .editorconfig
  .github/workflows/
    validate.yml             runs the specification's checks; no logic of its own
  .vscode/
    extensions.json          XML, SPARQL, Turtle, EditorConfig
    settings.json            XSD association for fixtures/in
  docs/
    format.md                ClinVar VCV XML as the adapter sees it
  schema/
    ClinVar_VCV_2.6.xsd      NCBI's schema, pinned byte for byte (md5 a7b65e5a166dc5f36a7eea9127d56f4e)
    ClinVarResult-Set.xsd    the efetch envelope root NCBI's schema does not declare; includes the above
  in/sparql/                 the mapping: a CONSTRUCT and a findings query per record class, and detect.rq
  vocab/
    clinvar-gaps.ttl         the gaps this adapter reports, one concept per gap, the body of every finding
    clinvar-accounting.ttl   what this adapter does with each path of its source, one entry per path
    clinvar-acmg-classifications.ttl, clinvar-review-statuses.ttl, clinvar-submitter-categories.ttl
                             the three lookup tables, one SKOS concept map each, joined on a folded and trimmed key
  fixtures/
    manifest.ttl             the test manifest: the cases and how to judge each
    in/                      four conformance inputs and NCBI's official sample
    expected/                the four expected graphs, from conformance, corrected where it was wrong
    findings/                the four expected findings graphs, a Bridge's convert --findings over the input beside each
```

## Decisions

The decisions that shaped this adapter — the manifest being an RO-Crate rather
than an `adapter.yaml`, the cases being a W3C `mf:` test manifest, the
detect rule being one query, standards over inventions — are not
restated here. Each is a property of the **adapter package format**, and the
authority for that format is the Cascade Bridge Specification:
[`adapter/ro-crate-metadata.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/adapter/ro-crate-metadata.md),
[`adapter/fixtures/manifest.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/adapter/fixtures/manifest.md)
and [`pinning.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/pinning.md).

What is specific to ClinVar rather than to adapters in general is in
[`docs/format.md`](docs/format.md): why the schema pin is 2.6, why an envelope
wrapper schema exists, and what is and is not known about the committed inputs.
Why this adapter is laid out the way it is, and the phases, are in
[issue #1](https://github.com/jayostis/cascade-bridge-adapter-clinvar/issues/1).

## Verification

This package must conform to the Cascade Bridge Specification.
`.github/workflows/validate.yml` runs that specification's lint on every pull
request and nightly; what it checks, and what a failure means, is
[`adapter/validation.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/adapter/validation.md)
there. Failures come out of the build.

A few things the lint cannot see, such as the detect rule's answer for each
input and each `sameAs` copy being byte-identical to conformance, are run by
hand before pushing; `fixtures/CLAUDE.md` and `schema/CLAUDE.md` say how.

There is no test code here by design. Executing an adapter's fixtures is a
Bridge's job, and `compatibility.json` names the Bridges CI runs on this
adapter; the specification's
[`engine/executing.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/engine/executing.md)
says how.

## Licence

Apache-2.0 for the package. ClinVar data is NCBI's, under
https://www.ncbi.nlm.nih.gov/home/about/policies/; each committed NCBI-derived
file's entry in `ro-crate-metadata.json` says so.
