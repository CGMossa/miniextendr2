# Factor construction roots (#1408)

The factor builders returned raw SEXPs before callers could protect allocations
made inside the constructors. Installing a previously unseen level symbol can
collect the fresh STRSXP container. The first factor-class cache lookup allocates
while the integer payload is unrooted. Cached levels also need a lexical root
until permanent preservation is established.

A new no-argument fixture covers raw levels, one-shot construction, and cached
levels. Its subprocess starts with untouched Rust factor caches, enables
`gctorture(TRUE)` after loading miniextendr, and retains three conversion rounds
for assertions on codes, NA, levels, class, and character values. It uses eight
levels with similarly sized fresh names and a two-code first factor so the
allocations can reuse the vulnerable containers' size classes. The initial
three-level fixture passed against the old implementation; allocation pressure
alone did not establish regression coverage. The first fixture build also
required moving `ProtectScope::new()` into its unsafe R-thread block.

The implementation keeps levels rooted from allocation through symbol
installation and, for cached levels, through `R_PreserveObject`. The factor
payload stays rooted through attribute assignment and cold class initialization.
The raw-return contract is unchanged: callers still root returned SEXPs before
allocating again, and `build_factor` callers keep input levels rooted.

The fresh-process test runs in GC-stress CI shard 1. The no-argument fixture also
participates in the existing dynamic sweep; the dedicated subprocess assertion
is needed because that sweep can warm the caches in earlier fixtures.

With the final allocation sizes, the original constructors pass three rounds
without forced GC but fail the fresh-process stress regression with a
`callr_timeout_error` after 120 seconds. The protected constructors pass all 39
assertions in that same regression. Existing factor, Arrow, dataframe-reader,
dataframe-enum, and columnar-flatten suites pass 827 assertions with no
failures, warnings, or skips.

`just fmt`, `just check`, full `just test` (including both UI suites),
`just clippy -- -D warnings`, and all three CI Clippy configurations pass.
The UI suite's deliberate impl-tag warning fixtures remain expected output.
The worktree R package was configured, installed, and documented; the new export
was installed twice before runtime testing. The Rust API corpus was regenerated.
`just templates-check` passes; this runtime change touches no source in the
`templates-sources` mapping.
