---
layout: post
title: Variadic generics, again
---

**Tl;dr: The Rust community should talk about variadic generics again, and motivated people should get the ball rolling.**

Discussions about adding variadics to [the Rust programming language](https://www.rust-lang.org/) have been around for a while. In 2021 I wrote [an analysis on the subject](https://poignardazur.github.io/2021/01/30/variadic-generics/), and it was already drawing on years of history.

As a quick reminder, variadic generics (aka variadic templates, or variadic tuples), are a feature that would enable traits, functions and data structures to be generic over a variable number of types.

```rust
fn make_tuple_sing<...T: Sing>(t: (...T)) {
    for member in ...t {
        member.sing();
    }
}

let kpop_band = (KPopStar::new(), KPopStar::new());
let rock_band = (RockStar::new(), RockStar::new(), RockStar::new(), RockStar::new());
let mixed_band = (KPopStar::new(), RockStar::new(), KPopStar::new());

make_tuple_sing(kpop_band);
make_tuple_sing(rock_band);
make_tuple_sing(mixed_band);
```

There's been a little less hype around the subject lately, as maintainers focused on other aspects of the language and worked hard to bring them other the finish line, notably const generics, generic associated types (GATs), async traits, internal infrastructure improvements, etc.

It seems a lot of people implicitly or explicitly agreed that the time for variadic generics hadn't come: proposals were too half-baked, and there were other more pressing priorities, like shipping MVPs for the features above.

**I'm confident the time has come, and we should start laying the groundwork now.**

The rest of this post will explain my reasoning, and what I think that groundwork would look like.


## Changing environment

The Rust community's mindset in 2023 is not the same as it was in 2021.

Bluntly put, people are more eager for the language to change. Not just end users, mind you, but maintainers and team members as well. There's a general sense that we've spent a lot of years stuck in decision paralysis, bikeshedding proposed improvements when we could have reaped the core benefits much faster if we'd let go of our perfectionism a bit, and focused on MVP implementations.

To quote [Without Boats' recent post](https://without.boats/blog/a-four-year-plan/):

> For those who don’t know, there was a big debate whether the await operator in Rust should be a prefix operator (as it is in other languages) or a postfix operator (as it ultimately was). This attracted an inordinate amount of attention - over 1000 comments. The way it played out was that almost everyone on the language team had reached a consensus that the operator should be postfix, but I was the lone hold out.
>
> At this point, it was clear that no new argument was going to appear, and no one was going to change their mind. I allowed this state of affairs to linger for several months. I regret this decision of mine. It was clear that there was no way to ship except for me to yield to the majority, and yet I didn’t for some time. In doing so, I allowed the situation to spiral with more and more “community feedback” reiterating the same points that had already been made, burning everyone out but especially me.

The Rust project seems to be moving away from this dynamic a bit. Features like generators and `where Foo::bar(): Send` syntax are being considered and worked on without a lengthy bureaucratic process. People are focusing more on small easy wins and less on big all-encompassing design visions.

At the same time, it seems like Rust's backlog has maybe cleared a bit? Looking at the [Roadmap for 2024 article](https://blog.rust-lang.org/inside-rust/2022/04/04/lang-roadmap-2024.html), a lot of features have been implemented or have made major strides:

- let else syntax.
- async traits.
- Type-Alias impl Trait.
- Generic Associated Types.

With these features stable or soon to be stable, I claim variadic generics are the next extension to Rust's type system we should consider.


## Why variadics?

Some people don't think variadic generics would be that useful to Rust developers.

A common rebuttal I've heard is that you can already implement variadic-equivalent code using macros and cons-lists of tuples. Eg, from [a snippet](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=a23aa4974c19216ed5bbd10cd8f674d1) posted in a previous discussion:

```rust
#[tokio::main]
async fn main() {
    // ...

    assert_eq!(
        var!("hello", "world", 101),
        var!(fut_a, fut_b, fut_c).join().await,
    );
}

// var!(a, b, c) expands to (a, (b, (c, ())))
#[macro_export]
macro_rules! var {
    () => ( () );
    ($a:ident $(, $b:ident)* $(,)?)  => ( ($a, $crate::var!($($b),*)) );
    ($a:expr  $(, $b:expr )* $(,)?)  => ( ($a, $crate::var!($($b),*)) );
}

pub trait FutureVariadic {
    type Join: future::Future;
    fn join(self) -> Self::Join;
}
impl FutureVariadic for () {
    type Join = future::Ready<()>;
    fn join(self) -> Self::Join {
        future::ready(())
    }
}
impl<F: future::Future, Rest: FutureVariadic> FutureVariadic for (F, Rest) {
    type Join = future::Join<F, Rest::Join>;
    fn join(self) -> Self::Join {
        future::join(self.0, self.1.join())
    }
}
```

In practice, I've heard of a lot of people wishing they could use variadics, but I've never heard of anyone using the "simple" design above, probably because of its verbosity (you need to write one trait and two trait implementations for every trait you want to use inside a variadic). And it only covers the basic "implement trait for tuple" use-case.

(By the way, I've noticed before that a lot of people like proposing simplified versions of variadics that would avoid most implementation complexity by noticing that variadics are equivalent to cons-lists of types. Often these proposals are of the "look how clever my design is" type; they usually have very little consideration for how convenient the feature would be to use, or how readable the code would be.)

I think variadics aren't necessarily useful for application developers, or people working on libraries with mostly concrete types (eg web libraries). But for people working on libraries with lots of type manipulation, variadic generics have obvious benefits. They let you go from manipulating types to *collections* of types.

A notorious example is [Bevy's `Query` type](https://docs.rs/bevy/latest/bevy/prelude/struct.Query.html). This type lets you request all entities in your world which own components with certain types, so you can then operate on these components using that type information:

```rust
// This system will run over every entity with both a Position and a Velocity
fn movement_system(query: Query<(&mut Position, &Velocity)>) {
    for (mut position, velocity) in query.iter_mut() {
        // ...
    }
}
```

This kind of code, more than the toy examples of `Future::join` or `Iterator::zip`, is where variadics shine. It's for these interfaces where you want to accept arbitray *collections* of types, and you want to use them like any other type in your generic function.

Right now, when you want to do some seriously type manipulation, you have to rely on band-aids. Deriving traits is based on proc-macros. Bevy uses [this monster of an implementation](https://docs.rs/bevy/latest/bevy/ecs/query/trait.WorldQuery.html#foreign-impls) to cover most reasonable macro sizes.

While variadic generics might not be useful for everyone, for a subset of library writers they would open a huge space of possibilities that's currently awkward to work with.


## Where to start?

I've already written an [analysis of variadics in Rust](https://poignardazur.github.io/2021/01/30/variadic-generics/), and my opinion hasn't really changed since.

The highlights for me are:

- **Focus on the compile error story.** In particular, the design should avoid anything that leads to post-monomorphization errors.
- **Avoid first-class types and types cons-lists.** These lead to a recursive style of variadic code, and everyone who has written C++ variadics can attest those are a major pain.
- **Start with an MVP.** While we can start discussion about advanced features now, I think most of the discussion should focus on getting something useful out the door as soon as possible, especially in areas where short-term decisions don't constrain the long-term design space.

The obvious place to start for an MVP is trait implementations, like the const generics MVP. In fact, I think we should even define our MVP scope as "something that can implement tuple debug":

```rust
impl<...Ts> Debug for (...Ts) where for<T in Ts> T: Debug
{
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        let mut f = f.debug_tuple("");
        static for field in self {
            f.field(field);
        }
        f.finish()
    }
}
```

The above requires:

- Variadic generic parameters, restricted to trait impl blocks, restricted to one variadic param, restricted to implementations where `Self` is `(...Ts)`.
- Variadic trait bounds that apply to the variadic param, as in `for<T in Ts> T: Debug`.
- `static for` loops that iterate over self (without returning values, for now). The loop body will need to be type-checked in a way that assumes the only thing we know about each field is that it implements `Debug`.

Those features wouldn't be enough to implement, say, Clone, but they're the minimum set of features to get *any* variadics at all. Already there's a lot to bikeshed: what's the declaration syntax? The loop syntax? where do you put the `...`? Should you use a magic trait instead, eg `Ts: Tuple`?

And really, the first thing we need to agree on: do we even want imperative-style variadics like above, or do we want C++-like recursive variadics? The latter would look like this:

```rust
trait DebugFields {
    fn apply(&self, f: &mut DebugTuple<'_>);
}

impl DebugFields for () {
    fn apply(&self, _f: &mut DebugTuple<'_>) {}
}

impl<Head: Debug, Tail: Tuple + Debug> DebugFields for (Head, ...Tail)
{
    fn apply(&self, f: &mut DebugTuple<'_>) {
        let (head, ...tail) = self;
        f.field(head);
        tail.apply(f);
    }
}

impl<Ts: Tuple + Debug> Debug for Ts
{
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        let mut f = f.debug_tuple("");
        self.apply(f);
        f.finish()
    }
}
```

I think before we spend any time debating the merits of more complex proposals, the very first step will be to hash out which of the above two visions we want (or both, or neither), and then *move on*. Part of the reason design work is stalled is that every proposal that comes out ends up litigating this debate over and over again, with the occasional "You guys are dumb, just copy Zig's type system" thrown in.

Assuming we can settle that debate, the next step would be to add features one by one, much like with const generics:

- Adding multiple type arguments,
- Mapping variadic types to other types,
- For-loops over multiple variadics,
- For-loops that return values,
- Variadic const generics,
- etc...

Overall, I hope the end result of this process will look like [Jules Bertholet's Variadics design sketch](https://hackmd.io/@x5qEAKM5Q36wKV_Wftw96g/HJFy6uzDh). It makes a lot of syntax choices that make intuitive sense, and feel like they would play well with compiler internals (and I've mostly borrowed from it for the code snippets above).

That doesn't mean we should adopt the design as-is (and historically "adopting" an entire design from a complex RFC has always been more wishful thinking than what actually ended up happening) but that we should keep it in mind as our rough goal while we debate "stepping stone" features.


## Call to action

Alright, this post so far has been full of theorycrafting.

But in practice, what should we do?

I think we're at the stage where we can seriously consider writing a RFC and get some actual level of attention from language contributors. That RFC should be minimal, opinionated, and include bulletproof justification for its choices.

Again, I think a good target for an MVP would be getting this code to work:

```rust

impl<???> Debug for Ts where Ts: IS_A_TUPLE_WITH_DEBUG_FIELDS
{
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        ???
    }
}
```

A working implementation would probably be helpful to get lang team members onboard.

Honestly, this is something where I really wish I had enough time and familiarity with the compiler to do it myself. Lacking both, I'm calling out to any extremely motivated contributors: if you've ever wanted to get generics variadics in Rust, now is the time to start.

For everyone else, I'd like for people to talk about variadics more. I'd like people to produce some in-depth discussion, that goes beyond publishing yet another from-the-ground-up proposal, into actually dissecting differences between proposals, and what consequences each choice would present.

I feel like we're entering a new era for the Rust language, where long-awaited features are getting more attention, and people are exploring new ideas over endlessly re-litigating bikeshed debates.

I'm hoping the coming months will prove me right.

[Discussion on r/rust.](https://www.reddit.com/r/rust/comments/17qp7v4/variadic_generics_again/)
