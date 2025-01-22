---
layout: post
title: Pre-RFC - Rename annotations
---

This is an informal proposal for improving [the Rust programming language](https://www.rust-lang.org/). I'll assume familiarity with the language.

## The problem

Let's imagine you're writing a crate.

Your crate has a single `lib.rs` file, with two modules and an arbitrary number of items:

```rust
pub mod foo {
    // ...
}

pub mod bar {
    // ...

    pub struct Foobar(i32, f32, String, u8);

    // ...
}
```

After a while, you decide that Foobar should really be exported from the `foo` module. It's a breaking change, but you're fine with releasing a new major version:

```rust
pub mod foo {
    // ...

    pub struct Foobar(i32, f32, String, u8);

    // ...
}

pub mod bar {
    // ...
}
```

Any user previously importing from your crate will get this error when they bump the version number:

```
error[E0432]: unresolved import `best_crate::bar::Foobar`
  --> src/lib.rs:12:5
   |
12 | use best_crate::bar::Foobar;
   |     ^^^^^^^^^^^^^^^^^^^^^^^ no `Foobar` in `best_crate::bar`
   |
help: consider importing this struct instead
   |
12 | use best_crate::foo::Foobar;
   |     ~~~~~~~~~~~~~~~~~~~~~~~~
```

This isn't ideal, but at least there's the "consider importing this instead" message giving these users an easy way to fix this.

But now let's say you decide that "Foobar" is a terrible name, and your struct should really be named "Foofoo" instead for consistency:

```rust
pub mod foo {
    // ...

    pub struct Foofoo(i32, f32, String, u8);

    // ...
}

pub mod bar {
    // ...
}
```

Now your users will get this message:

```
error[E0432]: unresolved import `best_crate::bar::Foobar`
  --> src/lib.rs:12:5
   |
12 | use best_crate::bar::Foobar;
   |     ^^^^^^^^^^^^^^^^^^^^^^^ no `Foobar` in `best_crate::bar`
```

No message to help them figure out what to use instead.


## The solution

Rust should have an attribute to inform the compiler that an item previously existed, but has been moved and/or renamed:

```rust
pub mod foo {
    #![diagnostic::renamed(Foobar, Foofoo)]

    // ...

    pub struct Foofoo(i32, f32, String, u8);

    // ...
}

pub mod bar {
    #![diagnostic::renamed(Foobar, crate::foo::Foofoo)]

    // ...
}
```

You might want to put the attribute on the "new" item instead, in which case the syntax would be:

```rust
pub mod foo {
    // ...

    #[diagnostic::renamed(from = crate::bar::Foobar)]
    #[diagnostic::renamed(from = Foobar)]
    pub struct Foofoo(i32, f32, String, u8);

    // ...
}

pub mod bar {
    // ...
}
```

You'd probably want both, for cases where either the module of origin is removed, or the destination is no longer in the same crate (e.g. because you've split your crate into sub-crates).

This diagnostic could help the compiler give more helpful error messages:

```
error[E0432]: unresolved import `best_crate::bar::Foobar`
  --> src/lib.rs:12:5
   |
12 | use best_crate::bar::Foobar;
   |     ^^^^^^^^^^^^^^^^^^^^^^^ no `Foobar` in `best_crate::bar`
   |
help: this item has been renamed to `best_crate::foo::Foofoo`
   |
12 | use best_crate::foo::Foofoo;
   |     ~~~~~~~~~~~~~~~~~~~~~~~~
```

Because the compiler is *told* that this is the correct move and not just guessing based on name similarity, `cargo fix` and similar tools would be able to automatically apply the rename.

Rename annotations would be helpful as a "grace period" after a crate's major version change, but they would also be useful for purely internal refactors, using `cargo fix` to change `use` directives throughout your codebase.

All in all, this feels like a pretty useful feature which, thanks to the `diagnostics` namespace's relaxed constraints, could be implemented relatively swiftly in the Rust toolchain.
