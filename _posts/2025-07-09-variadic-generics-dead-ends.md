---
layout: post
title: Variadic Generics ideas that won't work for Rust
---

A long-running interest of mine is how we could add variadic generics to [the Rust programming language](https://www.rust-lang.org/).

The discussion is long-running enough that I've seen some arguments show up again and again in various forms from various people, over a span of a decade.
Since it looks like we're reaching a point where [the Rust maintainers may soon accept work on the feature](https://poignardazur.github.io/2025/06/07/report-on-variadics-rustweek/), I want to take the time to head off a few of those arguments from the get-go.


## Quick refresher on variadics

In programming, "variadic" usually means that some function or type can have an arbitrary number of arguments.
"Variadic generics" means those arguments are handled in the type system, not just type-erased.

In Rust, you can think of variadic generics as "being able to implement a trait for tuples with an arbitrary number of fields":

```rust
impl<...Ts: SomeTrait> SomeTrait for (...Ts) {
    fn do_stuff(&self) -> u32 {
        let mut sum = 0;
        for member in ...self {
            // The trait bounds ensure each member has a do_stuff() method
            sum += member.do_stuff();
        }
        sum
    }
}

let value: u32 = (0, 0.5, "hello", Some("hello"), false).do_stuff();
```

(Not that the syntax is a placeholder, and not the point of this article.)

You could also use it to map types:

```rust
impl<...Ts> UnwrapAll for (for<T in Ts> Option<T>) {
    type Unwrapped = (...Ts);
    fn unwrap_all(self) -> Self::Unwrapped {
        for option in ...self {
            option.unwrap()
        }
    }
}

let my_gift_boxes = (Some(1), Some(true), Some("hello"));
let opened_gifts = my_gift_boxes.unwrap_all();
assert_eq!(opened_gifts, (1, true, "hello"));
```

(Again, *this is placeholder syntax*. Please don't bikeshed the syntax in the comments.)

In some cases, you may not want to use traits at all:

```rust
fn unwrap_all<...Ts>(options: (for<T in Ts> Option<T>)) -> (...Ts) {
    for option in ...options {
        option.unwrap()
    }
}
```


Since this feature currently doesn't exist, crate maintainers will often fake it with macros (for example, it's how [the standard library implements `Hash` for tuples:](https://github.com/rust-lang/rust/blob/54ab4258397030d71cbc5fbf279c0bc1861560aa/library/core/src/hash/mod.rs#L927-L939)).

For more details on variadic generics, what proposals have been made about them, and what they might look like, read [Analysing variadics, and how to add them to Rust](https://poignardazur.github.io/2021/01/30/variadic-generics/).

Today, I'll focus on three proposals that people bring up a lot in associated discussions, and try to explain why these proposals wouldn't work for the Rust language.


## Dead end 1: Just use iterators

In threads about variadics, people unfamiliar with language semantics will often ask "Why not just have the Tuple trait be an iterator?"

To give an example:

```rust
impl<Ts: Tuple> SomeTrait for Ts {
    fn do_stuff(&self) -> u32 {
        let mut sum = 0;
        for member in self.iter() {
            sum += member.do_stuff();
        }
        sum
    }
}
```

The problems with this one should be obvious pretty quickly.

### Dealbreaker: There's no guarantee the members implement your trait

The code above isn't sound: there is no guarantee that arbitrary tuples will have members which implement `SomeTrait`.

Also, the for loop above implicitly assumes that `member` has the same type every time, but `Self` is a tuple, its members could have any type!

You could get around this problem by making `Tuple::iter()` return some kind of type-erased reference (maybe `&dyn Any`), but then you get back to the problem that you can't make the type-erased reference implement `SomeTrait`.

Another workaround would be to add some kind of `TupleFieldsImplement<DynTrait>` trait, so the above code would be written as:

```rust
impl<Ts: Tuple + TupleFieldsImplement<dyn SomeTrait>> SomeTrait for Ts {
    fn do_stuff(&self) -> u32 {
        let mut sum = 0;
        for member in self.iter_fields() {
            // type of `member` is `dyn SomeTrait`
            sum += member.do_stuff();
        }
        sum
    }
}
```

This could *maybe* be implemented without huge changes to the trait solver.
Even then, this is only a partial solution that only works for dyn-compatible traits and doesn't address e.g. associated types (unless we also add syntax-level trait parameters).

### Dealbreaker: Mapping tuples is clunky

The syntax above lets you read from a tuple's fields, but it doesn't let you write a new tuple.
So for example, the Clone impl might look like this:

```rust
impl<Ts: Tuple + FieldsImplementClone> Clone for Ts {
    fn clone(&self) -> Self {
        let mut clone = MaybeUninit::<Self>::uninit();

        for (member, clone_member) in self.iter_fields().zip(clone.iter_fields()) {
            *member = clone_member.clone()
        }
        clone
    }
}
```

In this case there are many obvious ways the code above doesn't work:

- How does the `FieldsImplementClone` trait even work? 
- What does `clone_member.clone()` even do? The `Clone` trait isn't dyn-compatible.
- How does the compiler know that the types of `member` and `clone_member` are the same? If they're `dyn Any`, there's no way to assign a `dyn Any` to another `dyn Any`.

It's always possible to say that tuple iteration wouldn't cover the `Clone` trait.
But then we're talking about a very limited subset of use cases.

**If a proposal doesn't let me implement `Clone` on arbitrary tuples, then that proposal isn't a viable substitute for variadics.**

Other traits we'd have a hard time implementing for similar reasons: `Default`, `PartialEq`, `PartialOrd`, `Serialize`, `Deserialize`, any trait with associated types, etc.


## Dead end 2: Variadic recursion

When variadic generics come up, people will often suggest implementing them the same way C++ does, with recursion.

That suggestion is presented as an elegant simplification: it would need very little new syntax or new concepts, just some new tuple features and trait wizardry to use them.

The idea is that for any function that wants to run over multiple types, you write a version of that function for N+1 types, and a version for a single type.

Because Rust doesn't have function overloading, the two functions would need to be trait implementations:

```rust
impl<T0: SomeTrait, Ts: Tuple + SomeTrait> SomeTrait for (T0, ...Ts) {
    fn do_stuff(&self) -> u32 {
        let (head, ...tail) = self;
        let value = head.do_stuff();
        let sum = tail.do_stuff();
        value + sum
    }
}

impl SomeTrait for () {
    fn do_stuff(&self) -> u32 {
        0
    }
}
```

This code relies on three new features:

- The `Tuple` trait, which enables the other two features.
- The `(T, ...Ts)` syntax that lets our trait impl match tuples with at least one element.
- The `(head, ...tail)` syntax that lets us destructure our tuple and get a tail tuple.

Already we can make some DX observations:

- We replaced an iterative loop with a tail-recursive loop (well, almost tail-recursive). This is intuitive for users of functional languages, but it's not quite idiomatic Rust.
- Our core logic is split across two functions: `(T0, ...Ts)` has the reduction function and `()` has the initial accumulator.
- We're placing a lot of faith in the inliner. If our tuple has 12 items, that's 12 levels of inlining the compiler must perform before it can even hope to optimize that function.
- We're also generating a lot of ELF symbols (one per tuple item) that we hope will get garbage collected away.

Still, it's not too hard to get an intuition for the work the compiler must do: when given a tuple, it dispatches non-zero-len tuples to the first trait impl and the unit type to the second, then recurses.

Let's consider a harder use case:

```rust
let my_gift_boxes = (Some(1), Some(true), Some("hello"));
let opened_gifts = my_gift_boxes.unwrap_all();
assert_eq!(opened_gifts, (1, true, "hello"));
```

We need to write an `UnwrapAll` trait which maps a variadic list of Option types to a variadic list of their unwrapped types.
If we keep the same recursive logic, the code might look like this:

```rust
trait UnwrapAll {
    type Unwrapped: Tuple;
    fn unwrap_all(self) -> Self::Unwrapped;
}

impl<T0, Ts: Tuple + UnwrapAll> UnwrapAll for (Option<T0>, ...Ts) {
    type Unwrapped = (T0, ...Ts::Unwrapped);
    fn unwrap_all(self) -> Self::Unwrapped {
        let (head, ...tail) = self;
        let head = head.unwrap();
        let tail = tail.unwrap_all();
        (head, ...tail)
    }
}

impl UnwrapAll for () {
    type Unwrapped = ();
    fn unwrap_all(self) -> Self::Unwrapped {
        ()
    }
}
```

We need one new feature, tuple concatenation, to write `(T0, ...Ts::Unwrapped)` and `(head, ...tail)`, but otherwise the example is fairly similar.
The observations we made earlier about DX still apply.

In the "gift boxes" example above, the type of `opened_gifts` would resolve to:

- `<(Option<i32>, Option<bool>, Option<&str>) as UnwrapAll>::Unwrapped`, then
- `(i32, ...<(Option<bool>, Option<&str>) as UnwrapAll>::Unwrapped)`, then
- `(i32, ...(bool, ...<(Option<&str>) as UnwrapAll>::Unwrapped))`, then
- `(i32, ...(bool, ...(&str, ...<() as UnwrapAll>::Unwrapped)))`, then
- `(i32, ...(bool, ...(&str, ...())))`, then
- `(i32, bool, &str)`,

Specifically, what I mean is that the compiler would need to do all these resolution steps every time it needs to figure out that `opened_gifts.2` is of type `&str`.
This would be transparent when everything goes well, but it would give some fairly opaque error messages if, say, you accidentally replaced `Some("hello")` with `Ok("hello")`.

Still, if readability problems were the only issues with that syntax, it would hardly be justified for me to call it a dead end.

I claim there are more fundamental problems with that syntax, that I haven't seen any proposal address.

### Dealbreaker: Tuple destructuring works poorly with references

Let's look at implementing the `Clone` trait again.

Our trait impl might look like this:

```rust
impl<T0, Ts: Tuple + Clone> Clone for (T0, ...Ts) {
    fn clone(&self) -> Self {
        let (head, ...tail) = &self;
        let head = head.clone();
        let tail = tail.clone();
        (head, ...tail)
    }
}
```

The `let (head, ...tail) = &self;` line would initialize head as a reference to the first element of `self` and `tail` as a reference to the rest of the elements.

Here's the problem: **Rust does not guarantee that the elements of a tuple are contiguous**.
In other words, there's no guarantee that the layout for `(A, B, C)` contains a valid instance of `(B, C)`.
For instance, an `(i8, i32, i8)` tuple might store its elements as `(i32, i8, i8)` for layout optimization.

We could guarantee tuple order and forbid the above optimization, at a (small?) performance cost for non-variadic code.

Otherwise, we'd need some syntax to map a reference to a tuple to a tuple of references, but I don't see an obvious syntax that fits the recursion style we've been using so far.

### Dealbreaker: Tuple destructuring/concatenation is hard to represent

In these discussions, I try to present various features the way their proponents usually present them.
Very often, they start from the premise that using tuple destructuring and concatenation would be better because it would be simpler to implement: after all, the language already has an (unstable) `Tuple` trait!

It's simply assumed that adding tuple destructuring and concatenation will be minor changes, and then we get variadics for free.

This is not, to put it mildly, an assumption that most language maintainers share.

For instance, going back to the initial example:

```rust
impl<T0: SomeTrait, Ts: Tuple + SomeTrait> SomeTrait for (T0, ...Ts) {
    // ...
}
```

What rules does the trait solver use to match types to this implementation?
How is the `Self` type represented?

Currently, tuples are represented as a list of either known or generic types.
This feature would require us to add another representation, "tuple with a suffix which is itself a generic parameter".

That is a deep-cutting change, that would need to be worked into many parts of the compiler.
These tuples would have their own rules (for instance, you couldn't do `self.3` in the above impl), their own interactions with the trait system, etc.

And this assumes that the feature is implemented in a minimalist, self-contained way.
If patterns like `(T0, T1, ...Ts)` or `(...Ts, Tlast)` or `(...Ts, ...Us, ...Vs)` are allowed, then the feature will need even more compiler support.

### Dealbreaker: Associated types are too opaque

Let's go back to the `UnwrapAll` example above.
Let's say we import the trait from some `unwrap-all` crate.

```rust
// unwrap-all/lib.rs
trait UnwrapAll {
    type Unwrapped: Tuple;
    fn unwrap_all(self) -> Self::Unwrapped;
}

/// ... trait impls ...
```

Let's imagine another crate with a trait that does the opposite of `UnwrapAll`: a trait that takes a variadic list of types and maps them to the same types wrapped in options:

```rust
// wrap-all/lib.rs
trait WrapAll {
    type Wrapped: Tuple;
    fn wrap_all(self) -> Self::Wrapped;
}

impl<T0, Ts: Tuple + WrapAll> WrapAll for (T0, ...Ts) {
    type Wrapped = (Option<T0>, ...Ts::Wrapped);
    fn wrap_all(self) -> Self::Wrapped {
        // ...
    }
}

impl WrapAll for () { /* ... */ }
```

Let's say that a developer imports both the `wrap-all` and `unwrap-all` crates.
Can that developer compose the two traits together?

For instance, is it possible to write this code?

```rust
fn pointless_packaging<Ts: Tuple + WrapAll>(tuple: Ts) -> Ts {
    let wrapped_tuple = tuple.wrap_all();
    let unwrapped_tuple = wrapped_tuple.unwrap_all();
    unwrapped_tuple
}
```

I claim that this code wouldn't compile.
(And indeed, the [variadics-free equivalent](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=f58c0cb7471cce9aa5697ad70a20614a) doesn't compile today.)

The problem is that the compiler has no way to unify the associated type of `WrapAll` with the requirements of `UnwrapAll`.

We as the people reading the code can *see* that the tuple produced by `WrapAll` is always going to be made exclusively of `Option<T>` types, but the compiler only knows that `wrapped_tuple` is of type `<Ts as WrapAll>::Wrapped` and has no way to prove anything about what is in that associated type without looking at impl bodies, which it does not have access to because it cannot prove that a specific impl will be picked.

As soon as we get into non-trivial use cases, using associated types recursively to represent variadics would mean that developers would never be able to compose them.

### Dealbreaker: The DX is just bad

I don't just want to leave this implied: the DX of everything I've shown so far is just awful.

Any time you want to do something with a list of types, you need to declare a new trait, write two implementations for that trait, and split the logic between them.
All the examples I've included so far have been annoying to write and probably annoying for you to read.

Compare with the more expressive syntax again:

```rust
fn unwrap_all<...Ts>(options: (for<T in Ts> Option<T>)) -> (...Ts) {
    for option in ...options {
        option.unwrap()
    }
}
```

I don't claim the syntax above is optimal or the one I'd pick.

But it's *so short*.
It's *so straightforward*.

You don't need to unroll the reduce logic in your head, you don't need to read two trait implementations, you don't even need traits at all.

Having a declarative syntax for variadics allows variadics to be convenient and painless to use in a way that the recursive syntax has a hard time getting close to.

Beyond the other obstacles I mentioned, that's my number one reason to dislike the recursive syntax: **developers who need variadics should not be forced to give up on for loops.**


## Dead end 3: First-class types

At the more extreme end of variadics proposals, people will sometimes say variadics should just be a special case of a more powerful type system, where types can be treated as values like any other.
Zig is often cited as an inspiration, usually with a mindset of "If Zig can do it, why can't we?".

In that case, variadic code might look like this:

```rust
impl<Ts: Vec<Type>> SomeTrait for Tuple<Ts> 
    where { for i in 0..LEN { Ts[i]: SomeTrait } }
{
    fn do_stuff(&self) -> u32 {
        const LEN: usize = <Self as Tuple>::LEN;

        let mut sum = 0;
        for i in 0..LEN {
            // The loop mirrors the loop in the where-bounds,
            // so each member is guaranteed to implement SomeTrait.
            member = self.get::<i>();
            sum += member.do_stuff();
        }
        sum
    }
}
```

To give another example, the unwrapping example might look like this:

```rust
fn unwrap_types(types: Vec<Type>) -> Vec<Type> {
    types
        .iter()
        .map(|ty| {
            match ty {
                EnumType("Option", inner) => inner.generic_arg(0),
                _ => panic("some error message"),
            }
        })
        .collect()
}

fn unwrap_all<Ts: Tuple>(options: unwrap_types(Ts::FIELDS)) -> Ts {
    const LEN: usize = Ts::LEN;

    let outputs: Ts;
    for i in 0..LEN {
        let option = options.get::<i>();
        outputs.set::<i>(option.unwrap());
    }
    outputs
}
```

(Some suspension of disbelief is needed here. That code would need a lot of Rust changes to work.)

This change is usually sold as being simpler, more fundamental than variadics.
Instead of adding a new syntax to the language, we're using an existing syntax and existing concepts (code, traits, normal control flow) and applying them to a new domain (types).

In practice this is a much, much larger change than variadics, and it would arguably turn Rust into a radically different language.

So it's kind of hard to explain concisely why this won't happen: it's like having a discussion about tire friction on the new car model, and having someone say we should make a helicopter instead.
It's kind of hard to know where to even start.

### Dealbreaker: Lots of post-monomorphization errors

Post-monomorphization errors refer to any compiler error in the code of a generic function, that isn’t triggered by compiling the generic function, but is triggered by instantiating it.

A major strength of Rust compared to C++ or D or Zig is that post-monomorphization errors are virtually non-existent. If your generic function compiles when you write it, it will compile when someone else uses it.

**Post-monomorphization errors are bad and first-class types would bring millions of post-monomorphization errors.**

(Man, that name is a mouthful. We need something shorter. Pomono errors?)

Take a chunk from the code above:

```rust
    for i in 0..LEN {
        let option = options.get::<i>();
        outputs.set::<i>(option.unwrap());
    }
```

How does the compiler know that the types of `options.get::<i>()` and `outputs.set::<i>()` match?

The type signature of the function doesn't guarantee it in a way the compiler can easily parse.

We as humans know that the `unwrap_types` function is always going to match `Option<T>` to `T`, and therefore that the type of `options.get::<i>().unwrap()` will be the right type to pass to `outputs.set::<i>()`, but the compiler can't trivially prove that for any caller of `unwrap_all`.

So the only way to make that function work would be to check types at monomorphization time, adding a lot of potential for more obscure, hard-to-debug errors.

### Dealbreaker: Type functions are bad for type inference

In Rust, you can write this code:

```rust
fn foo() {
    let mut my_list = Vec::new();
    my_list.push(10_u32);    
}
```

The compiler infers the type of v to `Vec<u32>` *after* it's been declared, because type inferences mostly go both ways: if you know that `Vec<X>` is `Y` and `Y` is `Vec<u32>`, then you know `X` is u32.

The Zig equivalent to the code above is

```zig
fn foo(allocator: std.mem.Allocator) !void {
    var my_list = std.ArrayList([]const u8).init(allocator);
    defer my_list.deinit();
    try my_list.append("foobar");
}
```

Notice that you have to pass the `[]const u8` type parameter as soon as `my_list` is created.

There are many reasons behind that, but the core of it is that type inference in Zig does not flow backward the way it does in Rust, and one of the reasons for *that* is that Zig's generics rely a lot on arbitrary type-to-type functions which by nature can't be reversed.

Adding arbitrary type functions to Rust would make them less convenient for type inference, in similar ways.

For instance, with the `unwrap_all()` function as defined above, this code wouldn't compile:

```rust
// ERROR: Missing type annotations
let my_gift_boxes = (Some(Default::default()), Some(Default::default()), Some(Default::default()));
let opened_gifts = my_gift_boxes.unwrap_all();
assert_eq!(opened_gifts, (1u32, true, "hello"));
```

### Dealbreaker: Rust maintainers kind of don't want to rewrite the entire language

This is the "no, we're not pivoting to building helicopters" part.

Ask anybody in the lang team, the compiler team, or the types team about first-class types and they will very likely recoil in horror, begin chanting in Latin, and throw holy water at you.

It's a big change.
Huge.
Easily bigger than every feature that was added to Rust since 2015, *taken together*.

Adding true first-class types to Rust would fundamentally change Rust as a language, its priorities, its philosophy, and its target demographics, in a way that can't be justified with "it's simpler than variadics".


## Conclusion

I hope by now my point has become clear: these things are more complicated than they look.

When discussing a complex change to an open-source project, people often have the urge to find a shortcut, a simple solution that avoids the work and drama that the project's community finds itself in as it grapples with the solution space.

We don't want big messy choices, we don't want big changes to the tools we use, so we ask "Could we just get what we want by doing this small tweak instead?".

These takes are especially common in discussions about variadic generics, because it feels like the `Tuple` trait is already there and iterators are already there, and there should just be a way to combine them and get variadics at home, right?

But when you dig into the details, you always find the same problems:

- The proposal doesn't cover common use cases (e.g. the `Clone` trait).
- The small tweak turns out to require a massive change to the compiler's internal architecture and how it sees types or traits.
- Making the proposal work for all use cases turns out to require much of the same work as the more general variadics proposals.

In practice, the core use case for variadic generics is "We want to implement all our traits with tuples of arbitrary size".
If we want to cover this use case, then the minimum set of features we need is:

- A syntax to express "all tuple members implement SomeTrait" as a bound.
- A syntax to iterate over tuple members, to both read, write, and create them, while treating each as a different type.
- Rules to combine the two points above: if you add a bound that all tuple members implement SomeTrait, then when iterating over members each member is assumed to implement SomeTrait.

There are a lot of different syntaxes and semantic choices we can make, but either way, code that uses these three features will look something like this:

```rust
impl<...Ts: SomeTrait> SomeTrait for (...Ts) {
    fn do_stuff(&self) -> u32 {
        let mut sum = 0;
        for member in ...self {
            // The trait bounds ensure each member has a do_stuff() method
            sum += member.do_stuff();
        }
        sum
    }
}

fn unwrap_all<...Ts>(options: (for<T in Ts> Option<T>)) -> (...Ts) {
    for option in ...options {
        option.unwrap()
    }
}
```

If you read about variadic generics and you think, "Oh, I have an idea for doing the same thing as variadics using only existing iterators", and your idea doesn't have the three features I listed, then your idea isn't enough to implement traits on arbitrary tuples.

So please take me at my word when I say that these proposals, and proposals similar to them, are non-starters.
If they were easy, they would have been implemented ten years ago.
The lang team has rejected them before, and will reject them again.

If you're skeptical of that last part, I've also gathered endorsements from oli-obk and Josh Triplett:

> **[Josh Triplett](https://github.com/joshtriplett):** Rust co-evolves with its community. One of our jobs is to notice when many people are experiencing the same complexity, and help them manage that complexity more easily. Variadic generics are a clear win, and this article shows how simple they could be and how much more complex the alternatives would be. \[...\] No matter how we spell it, variadic generics will absolutely have a "do this for each entry in the list" operation, where the body of that has the same code but different types each time.

> **[oli-obk](https://github.com/oli-obk):** While I like the ideas of the rejected proposals, they stop being neat solutions once the details actually have to be hammered out. This article covers the critical concerns I have from a feature implementer's perspective and explains them thoroughly enough that I will use it to focus future discussions by referring to it instead of re-litigating the points ad-hoc.

After years of design discussion, we're finally at a stage where variadic generics are reaching the top of the triage pile.
Serious discussion and initial work is, hopefully, about to start.
I'm hoping this article helps us not get dragged down in litigating the same alternatives over and over again.

Anyway, this was the last big article I wanted to write on the subject.

My next step after this will be writing an RFC.
Stay tuned!

[Discussion on r/rust.](https://www.reddit.com/r/rust/comments/1lvlhpk/variadic_generics_ideas_that_wont_work_for_rust/)
