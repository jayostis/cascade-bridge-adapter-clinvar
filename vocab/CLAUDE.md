# vocab — Agent Context

This adapter's own namespace, `https://ns.cascadeprotocol.org/adapter/clinvar/v1-draft#`,
prefix `clinvar:`. No Cascade term is minted here.

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
- The five `skos:broader` kinds are the specification's `bridge:gapKinds`. Adding
  a kind is a change to [cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec).
- `bridge:closedBy` names the one Cascade term that would close the gap. A gap
  whose sentence proposed two terms names neither until the rule is split.

## Adding, removing or renaming a gap

The three `in/sparql/*-findings.rq`, the four `fixtures/findings/*.gaps.ttl` and
`ro-crate-metadata.json` change in the same commit: a body outside this scheme
and a gap no query constructs are both red.

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
  `@display_start`, `@display_stop`, `@positionVCF` and `@variantLength` name
  `@start` or `@stop`; `@Accession` names `@Chr`; `@AssemblyAccessionVersion` and
  `@AssemblyStatus` name `@Assembly`.
- **A second identifier restates the first**: an allele's `@VariationID` names
  the record's, a gene's `@GeneID` names its `@HGNC_ID`, a molecular
  consequence's `@DB` names its `@ID`.
- **A summary attribute restates what it summarises**: the record's
  `@VariationType` names `SimpleAllele/VariantType`, its `@RecordType` names
  `ClassifiedRecord`.
- **A copied field restates its origin**: a `ClinVarAccession`'s dates name the
  assertion's, its `@OrgAbbreviation` names `@SubmitterName`, and an observed
  sample's `Species` names the record's.
