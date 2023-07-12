# Identifier Interning

> String interning is a method of storing only one copy of each distinct string value, which must be immutable. Interning strings makes some string processing tasks more time- or space-efficient at the cost of requiring more time when the string is created or interned. The distinct values are stored in a string intern pool.

In Ry compiler, we have `Interner` (which lives in `ry_interner`), to intern strings, more precisely identifiers.

## Motivation

The motivation behind identifier interning is to reduce the number of allocations, and make comparing identifiers efficient (comparing identifiers == comparing `usize` values).

## Usage

In Ry every interned identifier is associated with its index in the intern pool:

```rs
type Symbol = usize;
```

Here is an example of how `Interner` can be used:

```rs
let interner = Interner::default();
let s1 = interner.get_or_intern("hello"); // allocation #1
let s2 = interner.get_or_intern("world"); // allocation #2
let s3 = interner.get_or_intern("hello");

assert_eq!(s1, s1);
assert_eq!(s1, s3);
assert_ne!(s1, s2);

// ID-s are sequential
assert_eq!(s1 + 1, s2);
assert_eq!(s2 - 1, s3);

assert_eq!(interner.resolve(s1), Some("hello"));
assert_eq!(interner.resolve(s2), Some("world"));
assert_eq!(interner.resolve(s3 + 1), None);
```

> Symbols that are interned by default, can be found in `symbols` module of `ry_interner` crate.
