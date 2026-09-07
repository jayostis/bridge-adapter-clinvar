# workflows — Agent Context

`validate.yml` has no logic on purpose and must not grow any. The checks live in
the Cascade Bridge Specification and are published from it as an action; this
workflow checks the package out and calls that action at the tag naming the
pinned commit. A check that needs writing is a change to the specification's
lint, made there and consumed here by moving the pin.

The `uses:` ref and `bridge:specPin` in the crate name one commit, machine and
reader. The lint compares them and fails the run when they differ, so they move
together, in the pull request that needs them.

This is a lint, not a fixture run, so "no tests here" stands: executing an
adapter's fixtures is a Bridge's job, and nothing here runs a mapping or compares
a graph. Checking that the package is well formed belongs where the package
lives.
