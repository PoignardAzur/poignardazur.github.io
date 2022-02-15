---
layout: post
title: "Rust 2030 Christmas list: Macro sources in compiler diagnostics"
---

This is the second entry on my Christmas list for Rust 2030.

These articles are me fantasizing about what I'd like [the Rust programming language](https://www.rust-lang.org/) to be like, if we had infinite resources to implement every possible feature. I'm not worrying about "is this important to implement right now" so much as "what should the language look like on the long term".

For my second article, I'd like to talk about compiler diagnostics in macros and proc-macros.


## A simple error

Let's write some very simple Rust code:

```rust
fn foobar() {
  let x : i33 = 0;
}
```

If we try to compile this, we get this error:

```
error[E0412]: cannot find type `i33` in this scope
 --> src/lib.rs:4:11
  |
4 |   let x : i33 = 0;
  |           ^^^ help: a builtin type with a similar name exists: `i32`
```

So far so good, it's a very readable error that tells us what went wrong: we made a typo when writing the type `i32`.

Now let's imagine some more complicated, but similar code, using declarative macros:

```rust
macro_rules! create_function {
    ($name:ident) => {

      fn foobar() {
        let x : i33 = 0;
      }

    };
}

create_function!(foobar);
```

If we try to compile this, we get a similar error:

```
error[E0412]: cannot find type `i33` in this scope
  --> src/lib.rs:5:17
   |
5  |         let x : i33 = 0;
   |                 ^^^ help: a builtin type with a similar name exists: `i32`
...
11 | create_function!(foobar);
   | ------------------------ in this macro invocation
   |
```

This is good: the compiler recognizes that the error comes from a macro instantiation, and tells us both where the macro was invoked, and where in the macro internals the error is.

Note that this doesn't work for integration tests:

```rust
// src/lib.rs
#[macro_export]
macro_rules! create_function {
    ($name:ident) => {

      fn foobar() {
        let x : i33 = 0;
      }

    };
}

//tests/test.rs
use test_rust_macro_error::create_function;

create_function!(foobar);
```

In that case we get the following error:

```
error[E0412]: cannot find type `i33` in this scope
 --> tests/test.rs:3:1
  |
3 | create_function!(foobar);
  | ^^^^^^^^^^^^^^^^^^^^^^^^^ help: a builtin type with a similar name exists: `i32`
  |
```

The error is now less specific, and doesn't point to the error location inside the macro. This is because integration tests are separate crates, so rustc treats `create_function` as if it were imported from an external crate.

The internals of imported macros are opaque; this usually makes some sense, since you don't have direct control over the code of your dependencies; but in this case it seems a bit inconvenient.


## Proc-macros

Now, everything so far has been mostly esoteric. It's rare that you need to care about the internals of a macro you've written yourself while debugging compile errors in integration tests.

Proc-macros are where things become interesting. Let's write the above example again, using a proc-macro crate:

```rust
// src/lib.rs
use quote::quote;

#[proc_macro]
pub fn create_function_proc(_item: TokenStream) -> TokenStream {
    return proc_macro::TokenStream::from(
        quote! {
            fn hello() {
                let x : i33 = 0;
            }
        }
    );
}

//tests/test.rs
use test_rust_proc_macro_error::create_function;

create_function_proc!();
```

We still get an opaque error:

```
error[E0412]: cannot find type `i33` in this scope
 --> tests/test.rs:3:1
  |
3 | create_function_proc!();
  | ^^^^^^^^^^^^^^^^^^^^^^^^ help: a builtin type with a similar name exists: `i32`
  |
```

This is more of a problem with proc-macros. As far as I'm aware, most testing with proc-macros is done in integration tests. Technically you can do unit tests with [proc_macro2](https://docs.rs/proc-macro2/latest/proc_macro2/), but I'm not aware of anyone doing that, and it's not that useful since what you want is the result of the compilation process.

For instance, the most well-known crate for proc-macro testing I know of is [dtolnay's trybuild](https://docs.rs/trybuild/latest/trybuild), which runs the compiler directly on your source code, using your proc-macros as a natural part of the compilation process, and then compares the result to stored snapshots.

So, my main point is this: **The only way to test your proc-macros is to compile them from another crate, but, when doing so, the compiler won't tell you where in the proc-macro code the error comes from**.

Ideally, given the test code above, I'd like the compiler error to be:

```
error[E0412]: cannot find type `i33` in this scope
  --> src/lib.rs:8:26
   |
8  |                 let x : i33 = 0;
   |                         ^^^ help: a builtin type with a similar name exists: `i32`
  --> tests/test.rs:3:1
   | create_function_proc!(foobar);
   | ------------------------ in this macro invocation
   |
```

Or at least, I'd like the *option* to have this be the error.

(Of course, in the example I gave the error isn't *that* useful; imagine that in real-world use cases you have hundred-lines long proc-macros where it's not immediately obvious where a given error can come from.)

Some people might reply that there is already `cargo-expand` and `-Z macro-backtrace`, but neither of these options give me what I want, which is a targeted error saying "this is the specific line you need to change to fix the problem".

\[EDIT:\] [Apparently](https://internals.rust-lang.org/t/rust-2030-christmas-list-macro-sources-in-compiler-diagnostics/15915/2) `-Z macro-backtrace` gives that information, but only for declarative macros.


## How to fix it

How could this be implemented? I'm honestly not sure.

I think it would require both changes in the quote crate and language changes.

As far as I'm aware, there's no way for quote to say "this token must have this specific span", except in very broad ways. The `proc_macro::Span` type has three constructors: `call_site()`, `mixed_site()` and `def_site()`, which are related to identifier hygiene, not to where error methods are located.

On the other hand, when you write `quote!( foo.bar() )`, the `foo`, `.`, `bar`, and `()` tokens should already have the span we need? Honestly, I don't know how proc-macro spans are handled, and I'm pretty confident guessing that there are less than 40 people in the world who do.

Either way, you'd want some compiler or library switch to turn these in-depth errors on or off (if only because you want `trybuild` to get the same errors a regular user would get).


## Conclusion

Again, I don't know how hard this would be to implement, and that's not the goal of this article.

My main point is that, right now, debugging huge macros can be kind of a pain, and I hope Rust improves the situation over time. More precise diagnostics in specific cases could be one of those improvements.

[Discussion on r/rust](https://www.reddit.com/r/rust/comments/rvtrr4/rust_2030_christmas_list_macro_sources_in/)
