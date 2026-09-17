# workflows — Agent Context

`validate.yml` has no logic on purpose and must not grow any. The checks live in
the Cascade Bridge Specification and are published from it as the `start`
action, which reads the crate's `bridge:specPin` and runs the specification's
checks at that commit. A check that needs writing is a change to the
specification, made there and consumed here by moving `bridge:specPin`.

The job names are required status checks on `main`, matched by name: renaming
one silently drops it from the merge gate. No job `needs:` another; a job
skipped because its dependency failed counts as passing.

This is a lint, not a fixture run, so "no tests here" stands: executing an
adapter's fixtures is a Bridge's job, and nothing here runs a mapping or compares
a graph. Checking that the package is well formed belongs where the package
lives.
