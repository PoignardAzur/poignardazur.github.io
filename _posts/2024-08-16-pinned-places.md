---
layout: post
title: Expanding on withoutboat's pinned places
---

Let's talk about pinning in [the Rust programming language](https://www.rust-lang.org/).

Recently, Rust maintainer **withoutboats** (secret codename "Boats") released an article called [Pinned places](https://without.boats/blog/pinned-places/) where he argues that we could solve most of the problems with `Pin` by adding a concept of pinned places to the language.


## The Context

Very long story short: in Rust, all values can be "moved", meaning their bytes are copied from one place to another and the value in the old place becomes invalid. In any situation where you have a `&mut T`, you can move the `T`.

This is a problem because some values become inherently unsound if you move their bytes around to a different place, most notably self-referential types (types with a reference pointing to a value owned by the same object as the pointer).

People have suggested solutions to address these use cases. In a [previous article](https://without.boats/blog/pin/), Boats goes over these solutions and why most of them are not feasible to implement without major disruption to the Rust ecosystem.

The solution that the language *did* settle on is pinned pointers: a `Pin<&mut T>` is a wrapper around a `&mut T` which lets you access its value and in some cases mutate its fields, but does not let you move the `T` by default. More than that, by creating a `Pin<&mut T>`, you are *declaring* that the value will not be moved for the remainder of its lifetime, even after the `Pin<&mut T>` is dropped. Since the compiler can't check that you hold to that commitment, `Pin::new_unchecked` is an unsafe function. (I'm skipping a lot of details here.)

Because of these constraints, Pin ends up being somewhat unwieldy. You need a macro to create a pinned reference on the stack, pin projection is hard, reborrowing must happen manually, methods end up looking like `fn foo(self: Pin<&mut Self>)`, etc.

This was just the cliff notes. Withoutboat's two articles are worth the read if you want more details.


## Adding new syntax

Withoutboats' solution is to add a concept of "pinned places" to the language, a keyword to declare these places, and a lot of syntactic sugar for code that uses them.

For the sake of this discussion, "places" are essentially regions of memory where a value can be stored. They're what C++ calls "lvalues" because only they can be on the left side of the `=` operand. Boats mentions three: local variables, references, and struct fields.

The keyword he proposes is `pinned`, and pinned places would be declared with `pinned mut`; in this article I'll use `pin` or `pin mut` instead because (1) it looks a lot nicer (2) the concept of a pinned immutable reference isn't very interesting.

With that in mind, a chunk of code that creates a pinned struct with a pinned field and borrows it in a function might look like this:

```rust
struct Foobar {
    pin foo: Foo,
    bar: Bar
}

fn use_foo(foo: &pin Foo) { ... }

let pin mut foobar = Foobar { ... };
use_foobar(&pin foobar.foo);
```

Pinned references could also be declared as method receivers:

```rust
impl Foobar {
    fn do_stuff(&pin self) { ... }
}

foobar.do_stuff();
```

So far so good, all I've done is restate what Boats suggested with a slightly more concise syntax.

But now I'd like to propose two more places that we might want to declare as pinned.


## More pinned places

To get the full value from the concept of pinned places, we should be able to declare pinned function arguments and returns:

```rust
fn take_and_return(pin something: Foobar) -> pin Foobar {
    let result = ...;
    // ...
    result
}
```

Taking a pinned value would be a guarantee from the function to the caller: after I take `value`, I won't ever move it.

Returning a pinned value would be a guarantee from the *caller* to the function: after the result is returned, the caller won't ever move it. The callee is perfectly free to move its result around before returning it.

Why would this be useful?

Well, consider one of the classic use cases for immovable types: interop with C++ strings.

```rust
struct CppString {
    ptr: *mut u8,
    len: usize,
    capacity: usize,
}
```

C++ does something called "small string optimization", where for small enough strings `ptr` may (or may not) point to the area covered by `len` and `usize`. This means a C++ string can't be safely moved in Rust.

The [cxx book's section on `std::string`](https://github.com/dtolnay/cxx/blob/fc603fdc30a3d1bd6b6d81598c53d9681e1fef40/book/src/binding/cxxstring.md) says this:

> Rust code can never obtain a CxxString by value. C++'s string requires a move
> constructor and may hold internal pointers, which is not compatible with Rust's
> move behavior. Instead in Rust code we will only ever look at a CxxString
> through a reference or smart pointer, as in &CxxString or Pin\<&mut CxxString\>
> or UniquePtr\<CxxString\>.
> 
> In order to construct a CxxString on the stack from Rust, you must use the
> [`let_cxx_string!`] macro which will pin the string properly.

[`let_cxx_string!`]: https://docs.rs/cxx/*/cxx/macro.let_cxx_string.html

Pinned function arguments and returns mean you could take and return C++ strings by value, for instance:

```rust
impl CppString {
    fn new(content: &[u8]) -> pin Self {
        let pin mut value = Self {
            ptr: null(),
            len: 0,
            capacity: 0,
        };
        value.ptr = // ...
        // ...
        value
    }
}
```

Of course, this would only work if something like GCE (Guaranteed Copy Elision) was implemented, eg if the language could guarantee that `value` is not moved between its declaration and its return.

Implementing GCE would have a lot of benefits, but it would be an engineering challenge in itself.


## Conclusion

I don't know if any of this will be implemented in the coming years. The types team has made it quite clear that they have a pretty long backlog of features they need to burn through before they can consider adding new syntax.

But in the meantime, it's a nice thing to doodle about. That syntax feels pretty clear and readable, and it would probably a long way toward attenuating the "Rust async is painful" problem.

I'm happy that after years of talking in circles about these problems, we can still eventually make progress.
