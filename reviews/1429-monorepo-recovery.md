# Interrupted monorepo build recovery (#1429)

A monorepo path-dependency build can freeze src/rust/Cargo.toml to vendor
paths. Removing only inst/vendor.tar.xz leaves those paths behind, so the
next metadata/configure operation fails if vendor is absent. The scaffolder
also inferred package names from arbitrary name lines, assumed every core
crate lived in a child directory, and omitted crate_name_rs. Starting R from
the suggested package subdirectory could select an older helper installation.

The freeze writer also emitted an empty [patch] parent heading. Make that
parent implicit while retaining [patch.crates-io], with a serialized-output
regression assertion.

An initial edit embedded quote-producing code inside a quoted CLI message and
failed R parsing. Construct the displayed build call separately, then interpolate
that value into the message.

The existing-project scaffold now asks Cargo for its workspace packages and
selects a library member (the root library by default; an explicit `crate_name`
resolves ambiguous virtual workspaces). It derives the dependency path from the
selected manifest, names the binding package separately from its library, and
uses a `core_library` dependency alias so a package named `core` does not shadow
Rust's standard library. Existing libraries have arbitrary APIs, so their
scaffold does not emit a call to a presumed `hello()` function. The newly created
library template still provides and demonstrates that function.

The first regression run passed Cargo metadata assertions for both layouts but
failed two message assertions: CLI wrapped “from this workspace root” across
lines. The assertion now normalizes display whitespace before checking the
workspace-root build instructions.
