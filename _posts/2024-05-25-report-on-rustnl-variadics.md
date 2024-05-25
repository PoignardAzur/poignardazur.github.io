---
layout: post
title: Report on variadic generics discussion at RustNL.
---

This is a summary of a bunch of discussions that took place at [RustNL's Unconference](https://2024.rustnl.org/unconf/), where developers representing various projects in [the Rust programming language](https://www.rust-lang.org/) came together to chat and find common ground.

During the Unconference, I ran a survey among developers of the GUI subgroups (and a few others) to see if their projects could benefit from the addition of **Variadic Generics** to the language.

As it turns out, more than a few did!


## Variadic what?

"Variadic generics", or variadic templates, cover a wide array of features that a language can implement. Usually, "variadic" means that some function or type can have an unlimited number of arguments. "Variadic generics" means those arguments are handled in the type system, not just type-erased.

In Rust, you can think of variadic generics as "being able to implement a trait for tuples with an arbitrary number of fields". You can already implement traits for arrays of any size, but an array's elements are all the same type. Variadic generics would cover tuples with heterogenous fields (eg normal tuples).

With that feature, implementing a trait for tuples might look like this (original code don't steal):

```rust
impl<...Ts: SomeTrait> SomeTrait for (...Ts) {
  fn do_stuff(&self) -> u32 {
    let mut sum = 0;
    for member in ...self {
      // The trait bounds ensure each field has a do_stuff() method
      sum += member.do_stuff();
    }
    sum
  }
}
```

Since this feature currently doesn't exist, crate maintainers will often fake it with macros. For example, here's how [the standard library implements `Hash` for tuples:](https://github.com/rust-lang/rust/blob/54ab4258397030d71cbc5fbf279c0bc1861560aa/library/core/src/hash/mod.rs#L927-L939)

```rust
// The actual code is a bit longer, but you get the idea
macro_rules! impl_hash_tuple {
    ( $($name:ident)+) => (
        #[doc(fake_variadic)]
        #[stable(feature = "rust1", since = "1.0.0")]
        impl<$($name: Hash),+> Hash for ($($name,)+) {
            #[allow(non_snake_case)]
            #[inline]
            fn hash<S: Hasher>(&self, state: &mut S) {
                let ($(ref $name,)+) = *self;
                $($name.hash(state);)+
            }
        }
    );
}

impl_hash_tuple! {}
impl_hash_tuple! { T }
impl_hash_tuple! { T B }
impl_hash_tuple! { T B C }
impl_hash_tuple! { T B C D }
impl_hash_tuple! { T B C D E }
impl_hash_tuple! { T B C D E F }
impl_hash_tuple! { T B C D E F G }
impl_hash_tuple! { T B C D E F G H }
impl_hash_tuple! { T B C D E F G H I }
impl_hash_tuple! { T B C D E F G H I J }
impl_hash_tuple! { T B C D E F G H I J K }
impl_hash_tuple! { T B C D E F G H I J K L }
```

For more details on variadic generics, what proposals have been made about them, and what they might look like, read [Analysing variadics, and how to add them to Rust](https://poignardazur.github.io/2021/01/30/variadic-generics/).


## The survey

The specific question I asked at the Unconference was: **"Are there specific cases in your existing code where your life could get easier if Rust had variadic generics?"**.

I'm only listing positive answers here: I had a bunch of answers along the lines of "Wow, that sounds cool, but I don't really need it for my project" which I didn't write down.

- **Ben Wishovich - Leptos:**

  - [`tc!` for implementing `EventHandler`](https://github.com/leptos-rs/leptos/blob/119c9ea23fac0bec45eb5c19fd4d458a17624cfb/leptos_dom/src/events/typed.rs#L306-L332)
  - [impl_into_view_for_tuples!](https://github.com/leptos-rs/leptos/blob/119c9ea23fac0bec45eb5c19fd4d458a17624cfb/leptos_dom/src/lib.rs#L1098-L1142)

- **Emil Ernerfeldt - Rerun.io**
  - The entire [`range_zip/generated.rs`](https://github.com/rerun-io/rerun/blob/8afcafa01839b085ab81311ae04d74db8b7c5701/crates/re_query/src/range_zip/generated.rs) and [`clamped_zip/generated.rs`](https://github.com/rerun-io/rerun/blob/8afcafa01839b085ab81311ae04d74db8b7c5701/crates/re_query/src/clamped_zip/generated.rs) files.
  - Interestingly, instead of macros, these files use textual code generation to cover the different tuple sizes.
  - Emil told me that this approach generated a lot of boilerplate but was overall fine, and he wasn't sure variadics were needed. Surprisingly, he didn't have a use case in egui.

- **Eddy Bruel - Stitch** (aka Makepad's wasm interpreter)
  - Interestingly, he wrote a [meta-macro for calling tuple macros](https://github.com/makepad/makepad/blob/dd2957438dc2a121dfc35997b244701eb1a481c1/libs/stitch/src/into_host_func.rs#L11-L32).
  - From what I saw, that macro is used thrice in that codebase, all in the same file:
    - [With `impl_wrap!` to implement `IntoHostFunc`](https://github.com/makepad/makepad/blob/dd2957438dc2a121dfc35997b244701eb1a481c1/libs/stitch/src/into_host_func.rs#L46-L99)
    - [With `impl_host_result!`](https://github.com/makepad/makepad/blob/dd2957438dc2a121dfc35997b244701eb1a481c1/libs/stitch/src/into_host_func.rs#L120-L146)
    - [With `impl_host_val_list!`](https://github.com/makepad/makepad/blob/dd2957438dc2a121dfc35997b244701eb1a481c1/libs/stitch/src/into_host_func.rs#L177-L204)

- **Jonathan Kelley, Evan Almloff - Dioxus**
  - [The `impl_dep!` macro in `use_reactive.rs`](https://github.com/DioxusLabs/dioxus/blob/40df088b7dd355c781b827df17d36b2ec40b3469/packages/hooks/src/use_reactive.rs#L64-L71), which implements the `Dependency` trait used by `use_reactive()`.
  - Jonathan was skeptical of variadics, but told me that he thought they would be especially useful if they enabled safe variadic function arguments for the various callbacks passed to hooks.

- **Ed Page - Winnow**
  - [`permutation_trait!`](https://github.com/winnow-rs/winnow/blob/main/src/combinator/branch.rs#L336-L358)
  - From what Ed told me, they used variadic-like macros for two purposes: sequences of rules, and permutations between alternate rules.

- **Raph Levien - Xilem**
  - [`impl_view_tuple!`](https://github.com/linebender/xilem/blob/main/xilem_core/src/sequence.rs#L341-L361)
  - (Okay, I didn't actually have to ask Raph, 'cause I maintain that code.)

- **Alice Cecil - Bevy**
  - Similarly to Stitch, the Bevy project has written [an `all_tuples` meta-macro](https://github.com/bevyengine/bevy/blob/bfc13383e0a0800c1e9c57454ae5454f5891ba8a/crates/bevy_utils/macros/src/lib.rs#L110) for calling tuple-implementing macros.
  - [A quick grep.app search](https://grep.app/search?q=all_tuples&filter[repo][0]=bevyengine/bevy) shows about 22 usages of the macro throughout the project. That's quite a lot!
  - From what Alice told me, variadic generics would be very near the top of the Bevy project's wishlist for Rust features. Her exact words were "We would use this pattern a lot more if we had full support".
  - Alice talked to me at length about the problems with Bevy's current approach (see "Do we really need variadics" section below).


## Takeaways from the survey

I was a bit surprised by the variety of answers I got.

The use cases were pretty diverse: Leptos and Xilem use them to list children in UI trees, Stitch uses them to represent argument lists of WebAssembly functions, Dioxus uses them to list reactive dependencies, Bevy for basically everything, etc.

Some people didn't know what variadics were, and once I explained, told me "Oh yeah, this would make some parts of my code so much easier". Some people thought they might add superfluous complexity to the language. Towards the end of the day, a few people came to me and went "I heard you were running a survey on variadics. Can I add my case? I really want them!".

### Do we really need variadics?

Coming in, I had a suspicion that people who didn't need variadics severely underestimated the level of motivation of people who did need them. RustNL confirmed that suspicion. The contrast between people saying "Why would anyone want that?" and people telling me "Oh yeah, we desperately want this yesterday." was pretty stark.

Alice Cecile made a strong case for them, and thought it was important to communicate to people that the macro workaround was not a satisfying solution:

- It produces terrible error messages.
- The compile times cost is high.
- Code using this pattern is hard to read and review.
- Its documentation is terrible.

Variadic generics would alleviate all of these problems, or outright remove them. For example, the standard library's documentation [already pretends the language implements variadics to document tuple implementations](https://doc.rust-lang.org/std/hash/trait.Hash.html#impl-Hash-for-\(T,\)); the linked entry is much more readable than [bevy's tuple implementations](https://docs.rs/bevy/latest/bevy/ecs/query/trait.QueryFilter.html#impl-QueryFilter-for-\(F0,+F1,+F2,+F3,+F4,+F5,+F6,+F7,+F8,+F9,+F10,+F11\)).

### Next steps

I've talked to a few Rust team people, and they've given me suggestions to move this forward:

- Kobzol suggested that I coordinate with the Rust Survey Team to make get a larger-scale version of the survey above.
- Niko Matsakis said that Variadics might be a good candidate for the [Rust Project Goals](https://blog.rust-lang.org/inside-rust/2024/05/07/announcing-project-goals.html) roadmapping effort. Honestly, I'm a little skeptical of any new bureaucratic structures the Rust project puts in place (historically, they've failed to address the root problem that changes don't happen unless a core contributor champions them), but I can still give it a go.

Aside from these, I think it may finally be time to write a thorough variadic generics RFC. Variadics RFC written so far have been somewhat bespoke and didn't really take into account existing discussions on the subject. I would like to write a RFC taking inspiration from [Jules Bertholet's design sketch](https://hackmd.io/@Jules-Bertholet/HJFy6uzDh) and [my own variadics analysis](https://poignardazur.github.io/2021/01/30/variadic-generics/).

Whatever happens, I'll be posting here about the progress I make. I think the time is right for this feature: project maintainers want it, lang team members are open to the idea, and the possibility space has been thoroughly mapped. I'm seeing some good signs we can finally make progress!

Fingers crossed.

[Discussion on r/rust.](https://www.reddit.com/r/rust/comments/1d0coao/report_on_variadic_generics_discussion_at_rustnl/)
