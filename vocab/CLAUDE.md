# vocab — Agent Context

This adapter's own namespace, `https://ns.cascadeprotocol.org/adapter/clinvar/v1-draft#`,
prefix `clinvar:`. No Cascade term is minted here.

## A lookup table is a concept map, and its rows are transcribed

One `skos:ConceptScheme` per file, one `skos:Concept` per source phrase,
carrying exactly one `skos:notation` and exactly one `skos:exactMatch` or
`skos:closeMatch`. Two concepts may share a match target; two may not share a
notation.

- **A notation is the key**: the source phrase lowercased and stripped of
  leading and trailing space, tab, carriage return and line feed, and of no
  other character. A notation written any other way matches nothing, and every
  value at that path is reported as a miss for as long as it stands.
- `skos:closeMatch` where the phrase and the term do not mean the same thing.
- **A row is transcribed, never invented.** Adding one, removing one or changing
  what a phrase maps to changes what this adapter carries; the crate's
  `schema:isBasedOn` for the file says where the rows came from.
- The entry of `clinvar-accounting.ttl` for the path holding those values names
  the map with `bridge:lookupIn` and the miss with `bridge:lookupNamesGap`, and
  the mapping query joins the same scheme on the same key. The two halves fold a
  value the same way or a value is mapped and reported as unmapped at once.

## A gap is a kind of problem, never an instance of one

`clinvar-gaps.ttl` is the scheme the crate names as `bridge:gapScheme`, and the
only place a findings query may take a body from.

- A `skos:prefLabel` says what the source carries and that nothing carries it
  across. It names no value read from a record, no accession, no date, no list
  and no vocabulary version: a value the rule fired on is the finding's
  `sh:value`, and which node it is about is the finding's selector.
- Two sentences that differ only by a value are one gap. Two source elements
  whose loss reads as two sentences are two gaps, even where one rule reports
  both; where one sentence covers both, they open one gap between them.
- Every `skos:broader` names a concept of the specification's `bridge:gapKinds`.
  Adding a kind is a change to [cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec).
- A gap whose sentence proposed two `bridge:closedBy` terms names neither until
  the rule is split.

## Adding, removing or renaming a gap

A body a findings query constructs is a gap of this scheme, and outside it is
red. So a findings query, the four `fixtures/findings/*.gaps.ttl` and
`ro-crate-metadata.json` change in the same commit.

A gap no findings query constructs is **not** red. An entry of
`clinvar-accounting.ttl` may report it instead, from its verdict or from its
`bridge:lookupIn`; which entries report, and how many findings each one yields,
is [`engine/sparql.md`](https://github.com/jayostis/cascade-bridge-spec/blob/main/engine/sparql.md)'s
to say. Retiring a query whose gap an entry reports therefore silences nothing,
and a rule that writes what its entry writes goes. A gap nothing names at all —
no query, no entry — is dead, and goes.

A gap whose findings are not `sh:Info` carries its own `sh:resultSeverity`:
severity belongs to the kind of problem, and an entry has no query in which to
write one.

## A verdict on a path is read, never inferred from its name

`clinvar-accounting.ttl` is the file the crate names as `bridge:sourceAccounting`:
one `bridge:PathEntry` for each element and attribute path the four judged inputs
carry *below* the record element, written from that element so that it holds
under both envelopes. The record element's own path carries no entry; each of its
attributes carries one.

- The same leaf name is carried under one parent and ignored under another, so a
  verdict is settled against `../in/sparql/`, not against the name.
- Several entries may name one gap: a gap is a kind of problem, and the paths
  that open it are as many as the source has.
- A gap no entry names is a rule about an absence, a construct the four judged
  inputs do not carry, or a path whose verdict is not a gap at all. Which one it
  is goes in the pull request, never into a gap minted to close the join.

## Which fact is restated where

`bridge:redundantWith` names another path of the **source** that states the same
fact. Whether that path reaches the graph is its own verdict's business, so a
target may be `bridge:consumed` or itself a gap; what the verdict buys is that
the backlog records a fact once, where the source states it canonically. Every
`bridge:sameFactAs` in `clinvar-accounting.ttl` is one of these:

- **A clinical assertion restates the aggregate record.** Each path under
  `ClinicalAssertion/SimpleAllele` names its counterpart under
  `ClassifiedRecord/SimpleAllele`, and each under `ClinicalAssertion/TraitSet`
  its counterpart under `Classifications/GermlineClassification/ConditionList/TraitSet`.
  Where the submitter spells a fact in another place — an `AttributeSet` holding
  the HGVS string, a gene cross-reference holding the gene identifier — the
  entry names that place instead.
- **An HGVS expression's parts restate the expression.** Every
  `NucleotideExpression` and `ProteinExpression` attribute names the `Expression`
  it was parsed out of, as do `SimpleAllele/ProteinChange` and the record's
  `@VariationName`; `SimpleAllele/Name` and an RCV's `@Title` name
  `@VariationName`, which names the expression in turn.
- **A sequence location's other spellings restate its coordinates**:
  `@display_start` and `@display_stop` name `@start` and `@stop`, and
  `@AssemblyAccessionVersion` names `@Assembly` in NCBI's accession form, as a
  submitter's `hg19` names `GRCh37`. A value *derived* from two of them is not a
  restatement of either, and takes a verdict of its own: `@positionVCF` takes the
  start and the allele shape, as `@variantLength` takes the start and the stop.
- **A second identifier restates the first**: an allele's `@VariationID` names
  the record's, a gene's `@GeneID` names its `@HGNC_ID`, a molecular
  consequence's `@DB` names its `@ID`.
- **A summary attribute restates what it summarises**: the record's
  `@VariationType` names `SimpleAllele/VariantType`, its `@RecordType` names
  `ClassifiedRecord`.
- **A copied field restates its origin**: a `ClinVarAccession`'s dates name the
  assertion's, its `@OrgAbbreviation` names `@SubmitterName`, and an observed
  sample's `Species` names the record's.
