# smol-symbol 💠

[![Crates.io](https://img.shields.io/crates/v/smol-symbol)](https://crates.io/crates/smol-symbol)
[![docs.rs](https://img.shields.io/docsrs/smol-symbol?label=docs)](https://docs.rs/smol-symbol/latest/smol-symbol/)
[![Build Status](https://img.shields.io/github/actions/workflow/status/sam0x17/smol-symbol/ci.yaml)](https://github.com/sam0x17/smol-symbol/actions/workflows/ci.yaml?query=branch%3Amain)
[![MIT License](https://img.shields.io/github/license/sam0x17/smol-symbol)](https://github.com/sam0x17/smol-symbol/blob/main/LICENSE)

This crate provides the ability to create globally unique (per input value), human-readable
`Symbol`s at compile-time as well as at run-time, that are meant to be reminiscent of the
`Symbol` type in the Crystal programming language.

Where this crate differs is the alphabet and length of our `Symbol` is a bit more restrictive,
allowing us to encode the entire text of each `Symbol` as a `u128` internally. The only caveat
is we are limited to 25 characters of length and an alphabet consisting of lowercase a-z as
well as `_`. No other characters are permitted.

The `Symbol` type can be created at compile-time using the convenient `s!` macro, and can also
be created using the `TryFrom<AsRef<str>>` impl at runtime, though this is not as efficient as
doing this at compile-time using the `s!` macro.

The `Symbol` type can also be turned into a `String` via a convenient `Into<String>` impl.

We also provide the ability to define custom alphabets that use the more general `CustomSymbol`
type via a handy `custom_alphabet!` macro, allowing you to alter these restrictions directly
(smaller alphabet = larger max length for a symbol) and add support for other languages or less
restrictive character sets. The only invariant that can't be customized at the moment is
`CustomSymbol` will always use a `u128` as its backing data store.

### Example
```rust
#[test]
fn symbol_example() {
    // Symbols can be stored in variables
    let sym1 = s!(hello_world);

    // Symbols can be used in const contexts
    const SYM2: Symbol = s!(goodnight);

    // Symbols can be compared with each other
    let sym3 = s!(hello_world);
    assert_eq!(sym1, sym3);
    assert_ne!(sym1, SYM2);
    assert_ne!(s!(this_is_a_triumph), s!(im_making_a_note_here));

    // Symbols are 16 bytes
    assert_eq!(std::mem::size_of_val(&sym1), 16);
    assert_eq!(std::mem::size_of_val(&sym1), std::mem::size_of::<u128>());

    // Symbols can even be created dynamically at runtime!
    let some_string = String::from("some_random_string");
    let dynamic_sym = Symbol::try_from(some_string).unwrap();
    assert_eq!(dynamic_sym, s!(some_random_string));

    // Can't be longer than 25 characters
    assert!(Symbol::try_from("this_is_too_long_to_store_").is_err());
    assert!(Symbol::try_from("this_is_just_short_enough").is_ok());

    // Character alphabet is limited to lowercase a-z and _
    assert!(Symbol::try_from("this-is-invalid").is_err());
    assert!(Symbol::try_from("this is_invalid").is_err());
    assert!(Symbol::try_from("this.is.invalid").is_err());
}
```

See the docs for `Symbol` and `s!` for more detailed information.
