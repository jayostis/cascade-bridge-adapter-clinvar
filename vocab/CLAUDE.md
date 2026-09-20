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
- Two sentences that differ only by a value are one gap. Two source elements are
  two gaps, even where one rule reports both.
- The five `skos:broader` kinds are the specification's `bridge:gapKinds`. Adding
  a kind is a change to [cascade-bridge-spec](https://github.com/jayostis/cascade-bridge-spec).
- `bridge:closedBy` names the one Cascade term that would close the gap. A gap
  whose sentence proposed two terms names neither until the rule is split.

## Adding, removing or renaming a gap

The three `in/sparql/*-findings.rq`, the four `fixtures/findings/*.gaps.ttl` and
`ro-crate-metadata.json` change in the same commit: a body outside this scheme
and a gap no query constructs are both red.
