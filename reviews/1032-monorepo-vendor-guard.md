# Monorepo configure accepted leaked vendor tarballs (#1032)

The generated monorepo configure script was run against eight small package
fixtures. Both a repository directory and a worktree-style `.git` file above
the package let a leftover `inst/vendor.tar.xz` select tarball mode. Configure
unpacked it and generated build files instead of reporting the leak.

The monorepo template lacked the existing source-tree classification and
leaked-tarball guard from rpkg and the standalone template. Ported that guard,
including the `Packaged:`/`Built:` exemption and the explicitly set bootstrap
exemption, before mode selection. The guard only diagnoses the leaked file;
it does not remove the archive or rewrite the source manifest.

The regression runs autoconf and the complete generated configure script.
It covers normal source mode, leaked archives under both `.git` forms,
packaged/installed metadata beneath a repository, bootstrap (including an
empty but explicitly set value), and tarballs outside a repository. Cases
use paths with spaces, real vendor archives, and a dependency-free locked
crate, so no network or Rust compilation is needed. Sources and archives
are checked for content changes; rejected cases must not create build files
or unpack vendor sources. Before the fix, the two leak cases failed twelve
assertions while the legitimate install cases passed.

The first package check caught an undeclared processx dependency in the new
test harness. The harness now uses base R's `system2()` with output directed
to log files, so expected configure failures return exit statuses without
adding a dependency or emitting R warnings.

Final validation passed: all 58 configure assertions; the standard R suite
(849 assertions, zero failures/warnings); `just fmt`, `just check`, full
`just test`, `just clippy -- -D warnings`, and all three CI Clippy feature
configurations. `just templates-approve` and `just templates-check` agree.
R CMD check reports zero errors and zero warnings. Its sole AGENTS.md NOTE
is tracked by #1409 and already addressed in PR #1487.
