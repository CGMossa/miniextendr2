# Scalar Option method returns (#1415)

Attempted to return an absent comparison from `MyFloat$nan()$RPartialOrd$partial_cmp()`.
The method raised “returned no value” although the documented scalar Option contract
promises `NA_integer_`; the installed-package regression reproduced that error.

The shared method return detector selected an unwrap-or-error strategy for all
ordinary Option types. Standalone wrappers instead converted the whole Option.
Recognize the finite set of scalar Option types with typed-NA IntoR implementations
in the shared detector. Keep unit, raw SEXP, Self, and unrecognized types on their
existing strategies. Runtime fixtures cover class dispatch, the trait path, strict
conversion, and worker execution; detector tests guard the boundary of the set.

The initial regression fixture used standalone `strict`/`worker` attribute syntax
on methods, which the method parser rejected. Strict conversion belongs on the
impl block; method worker dispatch uses `env(worker)`. Corrected the fixture to
exercise those supported paths.

A helper `macro_rules!` fixture that spliced caller-authored methods into an
impl exposed a separate `__miniextendr_call` hygiene failure. The runtime matrix
now uses explicit impls, matching ordinary class definitions. The first draft
also named the removed `ffi` module; Rcomplex is exported at the API crate root.
