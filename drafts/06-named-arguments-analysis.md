---
layout: post
title: Analysising named and optional arguments and how to add them to Rust
---

TODO
This is an analysis of how variadic generics could be added to [the Rust programming language](https://www.rust-lang.org/). It's not a proposal so much as a summary of existing work, and a toolbox for creating an eventual proposal.

xkcd standard meme

Summary:

## Introduction
### Prior work
#### RFCs
## The facts
## My opinion
## What should we do
(organizationally)

Developer delight
https://blog.rust-lang.org/inside-rust/2022/02/22/compiler-team-ambitions-2022.html

Main problem: everybody wants to be a friggin language designer

### Prior work

#### RFCs

https://github.com/rust-lang/rust/issues/6973
https://github.com/rust-lang/rfcs/pull/152
https://github.com/rust-lang/rfcs/issues/323
https://github.com/rust-lang/rfcs/pull/257
https://github.com/rust-lang/rfcs/pull/343
https://internals.rust-lang.org/t/pre-rfc-keyword-arguments/1453
https://internals.rust-lang.org/t/pre-rfc-named-arguments/3831
https://internals.rust-lang.org/t/named-default-arguments-a-review-proposal-and-macro-implementation/8396
https://internals.rust-lang.org/t/pre-rfc-named-arguments/12730
https://internals.rust-lang.org/t/yet-another-named-arguments-prototype/12937
https://internals.rust-lang.org/t/pre-rfc-named-arguments/16413

- https://github.com/fredpointzero/rfcs/blob/variadic_tuples/text/0000-variadic-tuples.md
- https://github.com/alexanderlinne/rfcs/blob/variadic_generics/text/0000-variadic-generics.md
- https://github.com/memoryleak47/variadic-generics/blob/master/0000-variadic-generics.md
- https://github.com/rust-lang/rfcs/issues/376
- https://github.com/rust-lang/lang-team/issues/61

For optional arguments:

- https://internals.rust-lang.org/t/pre-rfc-make-function-arguments-for-option-typed-arguments/3741
- https://internals.rust-lang.org/t/feature-request-optional-arguments-struct-fields-etc/15242

Record type:
https://internals.rust-lang.org/t/record-types/18258/11


#### Other languages

https://peps.python.org/pep-3102/
https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions/#Function-Argument-Labels-and-Parameter-Names

Rosetta stone: https://rosettacode.org/wiki/Named_parameters

Make a Swift study: does Swift code use named variables


#### Precedent in the language





(scottmcm)
> Basically, my core thing here is that if something is nominal, I expect the names to be the thing that matters. Not that positions have to matter too.
>
> (Like it would be really weird if we said "no, you must match those enum variants in the same order as their definition".)

> Let each user decide how their functions should look. Don't roll into Go-ideology "we don't have it because you don't need it and we know better".

> I do not agree whatsoever that my_fun({ foo, bar, baz }); is ugly. The syntactic overhead is minimal. Moreover, and unlike named arguments, you can build up each bit in let statements and then use the field shorthand mechanism to avoid having to write foo(name: value, ...).

Potential syntax:

> demo(1, 2, d: 5, c: 3);
> demo(1, 2, { d: 5, c: 3 });
>
> I prefer the first syntax at call sites because including the braces looks especially ugly when you only use keyword arguments:
>
> duplicate({str: foo, count: bar});
>
> It's not gonna be pretty in declarations either esp. if supporting keyword arguments for all arguments became the norm, but they're less frequent than call sites at least.

> Named arguments in Rust increase the complexity of the language. So before adding a feature we need to think well if it pulls its weight.
>
> On the other hand, I think the largest part of the “complexity” of a language comes from features that have corner cases and unexpected, invisible, unintuitive or unsafe interactions with other language features.
>
> When you have features that fit together well, have no weird interactions with other language features, are safe and handy to use, and have a reasonably intuitive syntax, you learn them quickly even if your language has lot of them.


```
It shouldn’t be that difficult to understand.

Every single new feature has tremendous long-term costs. It has to implemented, then maintained at least until the next breaking version (which might be years away, if ever). It has to be documented. It has to be explained. It has to be considered when designing and writing code, yet another moving part to keep in mind. It’s yet another piece of syntax for new users to see and think “what the heck is that?” and then have to try and google syntax (which never works well). It’s hundreds of the same Stack Overflow questions over and over again because so many people don’t bother to search, or don’t know what they’re searching for in the first place, or how to word it.

It’s yet another grammar construct that can’t reasonably be used for something else. It’s another thing that doesn’t interact with the type system in a clear fashion, possibly leading to even more complicated and hairy type signatures once you get out of the realm of obvious examples and into the edge case minutiae. It’s another complication for tools that aim to work with Rust code (which we already have a relative paucity of). It also doesn’t really compose with other features; it’s just a dead-end corner of the language that does this one thing.

And to top it all off? You don’t need it. There is no credible argument you could make that this feature is actually essential for the language. The problems it seeks to solve can already be handled in other, currently implemented ways.

There are already so many languages that will grab every feature they can get their hands on. I’d argue that most languages made in the last decade or so do that. I think it’s a great thing that Rust is being developed by people willing to say “You know what? No.” People willing to not accept the simple and obvious solution, but to wait for something fundamentally better.

Rust is conservative in a world filled with “implement all the things!” languages. I’m glad for Rust in part because it isn’t Python, and isn’t C++. It doesn’t need to pack on every last feature it could theoretically have. That’s how you end up with a bloated, corpulent language where people have to start defining subsets just to stay sane.

To say you’re baffled by the resistance suggests you aren’t considering how people like me feel about Rust. Rejecting your opponents as insane is a great way to completely sabotage any possibility of reasonable discourse.

You want to know the kicker? I want named arguments.

But, at the same time, I’m thankful that there are people in this community who will pick at every little unsettled detail, force new proposals to go above and beyond to prove their value, and to withstand relentless scrutiny.

Not always happy about it, mind you. I’m still annoyed by an RFC I wrote being rejected (meaning Rust didn’t end up avoiding the problem it was written to head off), and I’m really sad my current RFC is probably going to be rejected even though I think it’s a good solution to a fairly frustrating problem the language has right now… but if that’s what it takes to keep Rust healthy and lean, so be it.
```


> It’s this that’s the problem. It only helps some callsites some of the time, but it introduces ambiguity in how to write and how to read all callsites everywhere, and makes the grammar and reasoning about arguments strictly more complex, across the language and all libraries.


> As a long-time (~12ys) Objective-C programmer and nowadays (almost) exclusive Swift & Rust programmer I’ve found myself to be more and more annoyed with mandatory(!) named parameters in Swift these days even though I used to be a huge proponent of named parameters in Objective-C.
>
> Why? Because it penalizes properly named variables.
>
> Whenever one names a variable, say, radius and passes it to a function taking an argument radius one ends up with
>
> draw_circle(location: location, radius: radius, color: color)
>
> Ugh. One would almost be better off with badly named variables:
>
> draw_circle(location: l, radius: r, color: c)


https://github.com/rust-lang/rfcs/pull/2964#issuecomment-671545486
```
We discussed this in the @rust-lang/lang triage meeting today. Here's the consensus from that meeting:

We're not entirely opposed to the concept of better argument handling, and we've followed several of the proposals regarding things like structural types, anonymous types, better ways to do builders, and various other approaches.

However, we feel that this absolutely needs to be discussed in terms of the problem space, not by starting with a specific solution proposal that solves one preliminary part of the problem. And while we don't want to let the perfect be the enemy of the good, we do think that taking an incremental step requires knowing what the roadmap looks like towards the eventual full solution.

Between that and our roadmap this year being more about finishing in-progress things rather than taking on major new features, we're going to close this RFC. When we're ready to work on this, we'd want to see a proposal in the form of an MCP; however, we're not looking for that MCP this year.
```

## Design questions

- Conflating named parameters and optional parameters?
- Syntax?
- Can there be named-then-unnamed params?
 - If not, doesn't that break "make invalid state unrepresentable"?
- Argument order at call site
- Default struct field values
 -> Overlap in use-cases?
- Anonymous structs
- Interaction with closure types
- Overloading?
- Are explicit arguments mandatory?
- Always allow intra-crate named arguments?
- pub syntax -> pub(crate) redundant?

## Counter-arguments

- Encourages anti-patterns (too many arguments)

> optional arguments hide the fact that you can pass more arguments to the function

- Can already be emulated with builder patterns and newtypes


## Examples in the wild

https://github.com/rust-lang/rfcs/issues/323#issuecomment-426918479

```
compile_opts.filter = ops::CompileFilter::new(
    library: LibRule::Default,
    binaries: FilterRule::All,
    tests: FilterRule::All,
    examples: FilterRule::none(),
    benches: FilterRule::none(),
);
```

https://github.com/rust-lang/rfcs/pull/2964#issuecomment-661826489

## Macro implementation attempts

https://internals.rust-lang.org/t/yet-another-named-arguments-prototype/12937
https://internals.rust-lang.org/t/possibly-one-small-step-towards-named-arguments/12857
https://github.com/samsieber/rubber-duck/blob/master/README.md