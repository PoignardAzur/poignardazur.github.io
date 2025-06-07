---
layout: post
title: Report on variadic generics discussions at RustWeek 2025.
---

This is a follow-up to last year's [Report on variadic generics discussion at RustNL](https://poignardazur.github.io/2024/05/25/report-on-rustnl-variadics/).

Last month, I went to [RustWeek 2025](https://rustweek.org/), an event gathering Rust maintainers, professional users and enthusiasts, from the same organizers as last year's RustNL.
This time the event took place in Utrecht, Netherlands, and much like last year there were two days of "Unconference", where people discussed various Rust-related issues.

Once again, I took advantage of the occasion to bring up variadic generics.

## Quick refresher on variadics

In programming, "variadic" usually means that some function or type can have an unlimited number of arguments.
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

(0, 0.5, "hello", Some("hello"), false).do_stuff();
```

Since this feature currently doesn't exist, crate maintainers will often fake it with macros (for example, it's how [the standard library implements `Hash` for tuples:](https://github.com/rust-lang/rust/blob/54ab4258397030d71cbc5fbf279c0bc1861560aa/library/core/src/hash/mod.rs#L927-L939)).

For more details on variadic generics, what proposals have been made about them, and what they might look like, read [Analysing variadics, and how to add them to Rust](https://poignardazur.github.io/2021/01/30/variadic-generics/) and [Jules Bertholet's design sketch](https://hackmd.io/@Jules-Bertholet/HJFy6uzDh).


## Discussions during RustWeek

I took advantage of the unconference to sound out a few Rust maintainers, much more informally than I did last year.

Some things that came out:

- Whereas last year the consensus ([in discussions after RustNL](https://rust-lang.zulipchat.com/#narrow/channel/213817-t-lang/topic/Report.20on.20variadic.20generics.20discussion.20at.20RustNL.2E/near/440652162)) was that the lang team didn't have the bandwidth to even *look* at variadics, now that the new trait solver is partially stabilized, team members have been more open to starting discussions about them.
- [Oli Scherer](https://github.com/oli-obk) is currently working on [a reflection proposal](https://github.com/rust-lang/rust-project-goals/pull/311). That proposal solves similar use-cases as variadics and Oli thought they might replace them, but I think I've successfully made the case that reflection wouldn't remove the need for variadics completely. If nothing else, it wouldn't cover the "implement `MyTrait` for all tuples whose fields implement `MyTrait`" use case.
- Oli was initially skeptical about the whole variadics concept. They were generally worried that the concept would be way too complex to implement, especially if we leaned towards "C++ style" variadic recursion. I think I managed to make the case that a simpler version of variadics was possible, which wouldn't be a nightmare to implement.
- [Josh Triplett](https://github.com/joshtriplett) is still cautiously enthusiastic about variadics. He's raised the possibility that improved declarative macros could cover similar use cases, but again, I've argued that only variadics cover the general cases.


## Do we really want variadics?

Whenever the subject comes up, some people are very skeptical.
Do we really need that feature?
Won't it add bloat to the language?

I'd like to quote a few paragraphs from my article last year:

> Coming in, I had a suspicion that people who didn't need variadics severely underestimated the level of motivation of people who did need them. RustNL confirmed that suspicion. The contrast between people saying "Why would anyone want that?" and people telling me "Oh yeah, we desperately want this yesterday." was pretty stark.

> The use cases were pretty diverse: Leptos and Xilem use them to list children in UI trees, Stitch uses them to represent argument lists of WebAssembly functions, Dioxus uses them to list reactive dependencies, Bevy for basically everything, etc.
>
> Some people didn't know what variadics were, and once I explained, told me "Oh yeah, this would make some parts of my code so much easier". Some people thought they might add superfluous complexity to the language. Towards the end of the day, a few people came to me and went "I heard you were running a survey on variadics. Can I add my case? I really want them!".

> Alice Cecile made a strong case for them, and thought it was important to communicate to people that the macro workaround was not a satisfying solution:
> 
> - It produces terrible error messages.
> - The compile times cost is high.
> - Code using this pattern is hard to read and review.
> - Its documentation is terrible.


## Next steps

### Submitting proposals

Last year, Niko suggested that I submit variadics as a [Rust Project Goal](https://blog.rust-lang.org/inside-rust/2024/05/07/announcing-project-goals.html).

I was skeptical at the time, but now that I've seen the Project Goal process play out for a year, I'm a little warmer to the idea.

We're also close to the point where I'd want write an MCP or an RFC.

### More writing

A problem with advocating for variadic generics is that the discussions about it get *a lot* of bikeshedding.

One thing that happens a lot is that people argue for one of two features:

- Tail-recursion variadics. This is the "C++ style" variadics I mentioned above; the idea is that your iteration primitive is to do `let (head, ...tail) = values; do_thing(head); recurse(tail);`.
- Arbitrary type-level functions. This means Zig-style functions which manipulate types like any other value.

Once those ideas get brought up, discussions very easily get lost into arguments and counter-arguments, and then peter out.

So as a prelude to any RFC, MCP or other project, I'd like to write an article along the lines of "What variadic generics shouldn't be" where I would make the case, in detail, that these proposals do not work.

Another frequent concern is that, even if an initial proposal keeps things simple, future proposals will want to add ever more use-cases for variadic generics, and keep blowing up the language complexity.

So another article I might want to write is one listing use-cases for non-simple variadic generics, and in which directions they might push the language.

The argument I'd like to make, ultimately, is that it's possible to come up with a syntax for variadics that's both readable, feasible to implement, and forward-compatible with fancy future ameliorations.


## Conclusion

Advocating for variadics has been a long journey, and we're still not seeing the end of the tunnel.

Still, the discussions at RustWeek gave me a fair amount of hope for the future.

I think variadics are a feature worth having, and I'm more optimistic than ever that we'll get them.

You know.

Eventually.

[Discussion on r/rust.](https://www.reddit.com/r/rust/comments/1l5ige9/report_on_variadic_generics_discussions_at/)
