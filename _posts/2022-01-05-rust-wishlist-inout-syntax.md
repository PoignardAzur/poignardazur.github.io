---
layout: post
title: "Rust 2030 Christmas list: Inout methods"
---

This is the third entry on my Christmas list for Rust 2030.

These articles are me fantasizing about what I'd like [the Rust programming language](https://www.rust-lang.org/) to be like, if we had infinite resources to implement every possible feature. I'm not worrying about "is this important to implement right now" so much as "what should the language look like on the long term".

For my third article, I'd like to talk about mutability duplicates.


## Definition

"Mutability duplicates", for lack of a better term, are functions that you need to write twice, where both versions do the exact same thing, except one of them has different mutability annotation. They're the bane of any language where the type system keeps track of mutability.

In Rust, they might look like this:

```rust
fn get_value(&self) -> &Value {
  self.value
}

fn get_value_mut(&mut self) -> &mut Value {
  self.value
}
```

In C++, they might look like this:

```c++
Value* get_value() {
  return this.value;
}

const Value* get_value() const {
  return this.value;
}
```

C++ developers are annoyed enough about the duplication that elaborate workarounds have started to surface, [like this StackOverflow answer:](https://stackoverflow.com/a/47369227/3511753)

```c++
T const & f() const {
    return something_complicated();
}
T & f() {
    return const_cast<T &>(std::as_const(*this).f());
}
```

Now, Rust developers are usually more easygoing about minor code duplication (my personal view is that if you're only duplicating one or two line, adding abstraction to remove the duplication [isn't worth it](https://caseymuratori.com/blog_0015)), but still. This is annoying.

Mutability duplication creates tons of papercuts: it means you to remember to add `_mut` after every method call, it creates API bloat, vtable bloat, etc.

Is there a better way?


## D's solution: `inout`

[In the D language](https://dlang.org/spec/function.html#inout-functions), `inout` is a sort of "wildcard" of type qualifiers.

For instance, while the `get_value` example in D could be written like this:

```d
Value* get_value() {
  return this.value;
}

const(Value)* get_value() const {
  return this.value;
}
```

The idiomatic way to write it is actually

```d
inout(Value)* get_value() inout {
  return this.value;
}
```

Thus, there is no code duplication. Besides being shorter to write, this has a minor documentation advantage: a library user can read the prototype and know that the function can be called with a mutable or immutable object, and that the code executed will be the exact same either way (so, for example, the mutable version can't lock a mutex).

### How it works

I'm only passingly familiar with the D language, so forgive the imprecise language.

At the call site, `inout` acts like a wildcard, that gets resolved to a particular type qualifier. What type qualifier it gets resolved to depends on the types of the parameters passed to the call site.

For instance, given this code:

```d

inout(int)* foobar(inout int* a, inout int* b);

// ...

foobar(my_a, my_b);
```

The type system takes the types of `my_a` and `my_b`, and finds the least restrictive qualifier that matches both these types. `inout` becomes that qualifer.

For instance, if my_a is `const int*` and my_b is `int*`, `inout` "becomes" `const`, and the function returns `const int*`. If both `my_a` and `my_b` are `int*`, then the function returns `int*`.

(There are [more matches](https://dlang.org/spec/function.html#matching-an-inout-parameter), but this is the simplified version.)

Inside the function's implementation, `inout` is basically a fancy version of D's `const` (which is roughly equivalent to C++'s `const`). There are some subtyping rules, but the main rule is that you can't mutate an `int` if all you have is an `inout* int`.


## Implement `inout` in Rust

With all that in mind, how could Rust implement the `inout` qualifier?

Well, first off, I think the rules should be a little different. D's type system is different from Rust: whereas Rust has bi-directional Hindley–Milner type resolution, D's type resolution is (I think) one-directional: return types are deduced from argument types, not the other way around. Also, D has function overloading and Rust hasn't.

All of that is to say that I'm not sure D's "least restrictive match" resolution would be decidable in Rust, and even if it is, it seems too complex and hack-ish for Rust.

Instead, I would propose a much simpler system: `inout` types are only allowed in methods, and their mutability must be the same as the `self` argument.

This should be relatively easy to implement in the type system (`inout` is basically one more constraint in type resolution), while covering 90% of use cases (almost all mutability duplication is in getter methods).

So the `get_value` example would look like:

```rust
fn get_value(&self) -> &inout Value {
  self.value
}
```

### Transition and breaking changes

There's currently a ton of types in the standard library (and, you know, in the entire ecosystem) that have duplicate `xxx()/xxx_mut()` methods. How would they transition to inout mutability?

The simplest solution would probably be to add the `inout` keyword to the `xxx()` version, and eventually deprecate the `xxx_mut()` version (except for cells and mutexes, where the internal code is indeed different).

This *ought* to be possible, because adding `inout` shouldn't be a breaking change in practice. In theory though, adding mutability to types in existing code might result in subtle behavior changes. `inout` might need to be introduced as having no effect in existing editions, and be activated by a new edition, so that `cargo fix --edition` could detect any actual change.

(Or language maintainers could do a [crater](https://github.com/rust-lang/crater) run to see if any code *actually* changes behavior in the wild, finds that none does, shrug, and implement the change immediately.)


## Conclusion

`inout` is of those innovative D features that I can't wait for Rust to integrate.

It solves a common use-case in a very concise fashion, though it needs to be implemented with care to avoid making language semantics extremely complicated.

I hope it or something like it is added to the language eventually.

[Discussion on r/rust](https://www.reddit.com/r/rust/comments/s0ocgf/rust_2030_christmas_list_inout_methods/)
