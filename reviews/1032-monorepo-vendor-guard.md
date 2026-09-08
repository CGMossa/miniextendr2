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
