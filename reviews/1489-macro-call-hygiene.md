# Mixed macro_rules method spans (#1489)

A compile-pass fixture generated a class impl inside `macro_rules!` while
inserting methods supplied by the caller. `just test-ui` reproduced E0425 in
seven conversion paths: ordinary values, vectors, borrowed strings, scalar and
vector coercion, mutable match-argument slices, and worker string conversion.
Rust could see a same-named `__miniextendr_call` declaration but rejected its
use because the two identifiers had different macro syntax contexts.

The wrapper declared that parameter using its expansion scope. Conversion
error arms wrote its name literally inside `quote_spanned!` blocks carrying
user-type spans, which changed the identifier's scope. Supply the wrapper's
identifier to `RustConversionBuilder` and interpolate it in every error arm.
Keep the surrounding user-type spans so compile-fail diagnostic locations
remain useful. The regression fixture mixes macro-authored and caller-authored
methods and covers ordinary, strict, coercing, and worker conversions.
