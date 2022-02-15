---
layout: post
title: "Rust 2030 Christmas list: Better cfg"
---

*\[Note - This is republishing of an article that was previous [posted on github gist](https://gist.github.com/PoignardAzur/8038f5ed7dc8d00e3e9472aca57fb35c), and advertised on reddit in February 2021.\]*

This article is me fantasizing about what I'd like [the Rust programming language](https://www.rust-lang.org/) to be like, if we had infinite resources to implement every possible feature. I'm tentatively calling it my "Christmas list for Rust 2030", mostly because it sounds catchy.

For my first Christmas wish, I'd like to discuss conditional compilation and cfg.

## The problem

There's a virtue of Rust I rarely see discussed. I would formulate it as follows:

**If it compiles on my machine, it probably compiles everywhere.**

I don't know if it was an explicit design goal of the lang team; and it's not a virtue that stems from one specific feature of the language. It's an emergent quality that comes from the trait system, and the way it avoids post-monomorphization errors; and also from the wider ecosystem, and the general availability of multi-platform libraries, such that the idiomatic way to do anything is almost always multi-platform.

But that virtue has one big, ugly exception: [conditional compilation](https://doc.rust-lang.org/reference/conditional-compilation.html).

Consider the following code:

```rust
#[cfg(windows)]
fn foobar() {
    println!("foobar");
}

fn main() {
    #[cfg(windows)]
    // Whoops. Typo.
    fobar();
}
```

This compiles on my machine. It compiles [on the Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=d31943c0c56bd8dfe9c15616052b2a42).

But if you try to compile it on a Windows machine, what you'll get is:

```
error[E0425]: cannot find function `fobar` in this scope
 --> tmp_rust.rs:9:5
  |
2 | fn foobar() {
  | ----------- similarly named function `foobar` defined here
...
9 |     fobar();
  |     ^^^^^ help: a function with a similar name exists: `foobar`
```

In other words, conditional compilation breaks my confidence that, if my crate compiles, then it will always compile for every possible target I intend to support.


### Mutually exclusive features

I've been working on [druid](https://github.com/linebender/druid) recently. It's a GUI framework written entirely in Rust; one of the things it has to deal with is multiple platform-dependent backends.

For instance, on Linux, druid implements two backends: x11 and GTK. These backends are mutually exclusive: druid can only bind to a single one. Cargo doesn't really provide a correct way to do that. [What druid does](https://github.com/linebender/druid/blob/8f4d2c508b90e9588aadc78713f39285a41ed47a/druid-shell/src/platform/mod.rs#L31-L43) is assume that, if one backend is selected, then the other isn't:

```rust
#[cfg(all(feature = "x11", target_os = "linux"))]
mod x11;
#[cfg(all(feature = "x11", target_os = "linux"))]
pub use x11::*;
// ...

#[cfg(all(not(feature = "x11"), target_os = "linux"))]
mod gtk;
#[cfg(all(not(feature = "x11"), target_os = "linux"))]
pub use self::gtk::*;
```

This isn't ideal for a few reasons:

- If druid is compiled with `default-features = false`, it emits some obscure errors in the dependency tree.
- It means compiling druid with `--all-features` implicitly suppresses the gtk backend.
- It creates an invisible peer-dependency.

That last point is subtle. To explain it, let me talk about how cargo handles dependencies.


### Dependency merging in Cargo

This is a little-known, almost undocumented implementation detail of Cargo.

You might know that, when resolving your dependency tree, [Cargo will merge SemVer-compatible dependencies](https://doc.rust-lang.org/cargo/reference/resolver.html). For instance, if you depend on `foobar 1.0.3` and one of your dependencies depends on `foobar 1.1.5`, Cargo will try to substitute both dependencies with the highest compatible version of foobar it can find (eg `foobar 1.4.0`). If another dependency uses `foobar 2.4.0`, then a second version of foobar (eg, `foobar 2.4.1`) will be compiled separately.

Part of this resolution will be to bundle crate features together. For instance, if your crate has the dependency:

```toml
foobar = { version = "1.0.3", features = ["blue"] }
```

and another crate you depend on has the dependency:

```toml
foobar = { version = "1.1.5", features = ["red"] }
```

then following the above logic, Cargo will try to compile `foobar 1.4.0` with the set of features `["blue", "red"]`.

An important consequence of that is that **Cargo features should be exclusively additive**.

Because if you declare a feature that takes away symbols or conflicts with another feature, then you create a situation where your crate might compile fine, but it might produce obscure compilation errors if your end-user installs another unrelated crate.

(This gets even worse when we consider how Cargo might conflate dependencies, dev-dependencies and build-dependencies, though edition 2021 will fix that problem)

In the case of Druid, this is mitigated by the fact that, if both features are activated, Druid just picks the x11 backend instead of producing a compile error; which means the crate degrades somewhat gracefully. This is still counter-intuitive, and prone to subtle breakages.


### Impact on CI and testing

One impact of the above problems is, it becomes a lot harder to be confident about a crate's code when it uses conditional compilation. Again, our ideal is:

**If it compiles on my machine, it probably compiles everywhere.**

Speaking from recent experience, conditional compilation makes it easy to commit code, try every manner of test you can think of, and still have the code fail on CI because of some combination of parameters you didn't try on your machine.

And that's *if* the CI covers everything. Maybe our `foobar` crate has a compiler error that only happens on macOS builds in release mode, and foorbar's CI only checks macOS builds in test mode.

(Release mode can't actually be detected in compilation, but `test` and `debug_assertions` can)

In theory, if `foobar` has N features, to be sure that possible configuration is error-free, you would need to run the CI 2^N times for every supported platform (so at least `windows`, `macos`, `linux` and probably the `wasm32` arch). In practice, everyone just includes a few likely builds and hopes for the best.


## What I want

This is the "Christmas" part of this article. I'm trying to imagine my ideal implementation, not something easy to add to the existing compiler. My general rule is:

**In the default case, I want the compiler to be able to guarantee that my code will compile with any combination of the features and targets I support.**

Going back to the first example:

```rust
#[cfg(windows)]
fn foobar() {
    println!("foobar");
}

fn main() {
    #[cfg(windows)]
    // Whoops. Typo.
    fobar();
}
```

I want the compiler to emit a compile error for the above code even when I am compiling for a Linux target.

In fact, I want the compiler to be able to type-check and borrow-check my code independently of target. Target-specific errors should be the equivalent of post-monomorphization errors: possible in some contexts (especially as const generics develop), but mostly absent.


### Partition symbols by config-set

To make sense of code with conditional compilation conditions, the compiler needs a way to annotate symbols, which I'll call **configset**.

A symbol's configset is the set of all config values in which that symbol exists. Since there are an infite set of possible configs (for instance, there are an infinite number of possible architectures), you should think of it as a predicate, not a finite `HashSet`.

Concepts of [algebra of sets](https://en.wikipedia.org/wiki/Algebra_of_sets) apply to configsets. Eg:

- A configset can be **disjoint** with another configset; eg, `target_family="unix"` and `target_family="windows"` are disjoint.
- A configset can be **a subset** of another configset; eg `target_os="linux"` is a subset of `target_family="unix"`.
- Multiple configsets can be **a partition** of a greater configset; eg `all(feature="my_feature", target_endian="big")` and `all(feature="my_feature", target_endian="little")` are a partition of `feature="my_feature"`.

**Rule n°1:** The compiler performs name lookup, type-checking and borrow-checking for every symbol, even when the current config isn't part of the symbol's configset.

So the `#[cfg(windows)]` example should fail to compile, because rustc tries to look up the name `fobar` and can't find a matching symbol.

**Rule n°2:** An expression can only refer to a symbol if the expression's configset is a subset of the symbol's configset.

So in the following code:

```rust
#[cfg(all(feature = "with_foo", feature = "with_bar"))]
fn foobar() {
    println!("foobar");
}

#[cfg(feature = "with_foo")]
fn foo() {
    // NOPE
    foobar();
}
```

foo isn't allowed to refer to foobar, because if the `with_bar` feature is turned off, then foo will emit a compile error.

The broader goal of these rules would be to change how the compiler reasons about conditionally-compiled code. It's an extension of the way Rust reasons about generics: instead of treating generic code as a bunch of opaque symbols, transforming these symbols into a typed instantiation on demand, and type-checking that instantiation, Rust treats generic code as typed from the get-go, which means it can catch type errors early on.

Similarly, instead of treating conditional compilation as a bunch of AST-level mutations that must be performed so the AST can be turned into IR the compiler can reason about, Rust should treat conditional compilation as IR-level properties of expressions/statements/functions/types/etc.

This means that, instead of compiling the same code N times with different configs each time, CIs and developers could just compile the code once, and trust that the compiler proved that the code would work with any possible config.


### Multiple symbols

Things get more complicated if the compiler needs to manipulate symbols that have multiple interpretations, eg

```rust
#[cfg(feature = "gl_backend")]
pub type GraphicsHandle = open_gl::GlHandle;

#[cfg(feature = "vulkan_backend")]
pub type GraphicsHandle = vk::VkHandle;

#[cfg(feature = "web_backend")]
pub type GraphicsHandle = web::DomNode;
```

(this is, for instance, the pattern that druid follows with its backends; each module exports an `Application` type, a `Menu`, a `Clipboard`, etc)

In some cases, the symbol may not even be of the same type:

```rust
#[cfg(feature = "use_dyn_cerealizer")]
static CEREALIZER: &dyn Cerealize = &IntoWheat;

#[cfg(not(feature = "use_dyn_cerealizer"))]
static CEREALIZER: &impl Cerealize = &IntoRice;

fn eat_cereals() {
    // Which symbol does this refer to?
    CEREALIZER.cerealize();
}
```

It's unclear whether the above code should fail to compile. On the one hand, we can clearly see that the code is valid whether or not the feature is turned on: in both cases, CEREALIZER implements the same trait.

On the other hand, it's unclear how the compiler should reason about it. What if the traits are different? What if one symbol is a function, and the other a constant? What if `eat_cereals` returns CEREALIZER's type?

I think these kinds of rules need to be easy to reason about, so I'll pick simple solutions:

**Rule n°3:** A module can declare multiple symbols with the same name, as long as they have disjoint configsets.

**Rule n°4:** A module can never export multiple symbols with the same name (namespaces notwithstanding), even when these symbols have disjoint configsets.

**Rule n°5:** If a symbol is declared multiple times, an expression can only refer to that symbol if the expression's configset is a subset of one of the declarations.

(so the `eat_cereals` function wouldn't compile, unless you copy-pasted it twice, once with the `use_dyn_cerealizer` feature and once without)


### Forward declarations

Rules 4 and 5 above are a little restrictive. In some cases we *do* need external code to refer to a single symbol (eg our `GraphicsHandle`) that can have multiple possible definitions (eg `GlHandle`, `VkHandle`, `DomNode`).

On the other hand, we also need an easy way to tell the compiler how to think about that single symbol internally.

I claim that the most elegant way to do that is with [Forward declarations](https://en.wikipedia.org/wiki/Forward_declaration).

Forward declarations are the idea that you can tell the compiler that a symbol exists, without actually defining the symbol, and promise that the definition will come later. They're mostly used in C/C++ header files, to refer to symbols (classes, functions, constants) defined later in the file, or in another file; function prototypes are the most common kind of forward declaration.

Rust currently doesn't have forward declarations, though [RFC-2071](https://github.com/rust-lang/rfcs/blob/master/text/2071-impl-trait-existential-types.md) and [RFC-2515](https://github.com/rust-lang/rfcs/blob/master/text/2515-type_alias_impl_trait.md) propose something similar.

Here are some examples of what forward declarations might look like in Rust:

```rust
pub trait PlatformGraphicsHandle { /* ... */ }

// FORWARD DECLARATION
pub type GraphicsHandle: PlatformGraphicsHandle;

// DEFINITION
type GraphicsHandle = open_gl::GlHandle;

// ---

// FORWARD DECLARATION
pub const MY_VALUE: i32;

// DEFINITION
const MY_VALUE: i32 = 0;

// ---

// FORWARD DECLARATION
pub fn foobar() -> i32;

// DEFINITION
fn foobar() -> i32 {
    // ...
}
```

In general, I think forward declaration syntax should mirror trait syntax: anything that can be declared in a trait, and implemented in the trait impl, can be forward-declared, then defined in a module.

**Rule n°6:** A module can export the forward-declaration of a symbol, *if* the module has one or more definitions of that symbol, and the configsets of the definitions are a partition of the configset of the forward declaration.

```rust
mod graphics {
    // ERROR: GraphicsHandle is undefined when feature gl_backend is off.
    pub type GraphicsHandle: PlatformGraphicsHandle;

    #[cfg(feature = "gl_backend")]
    type GraphicsHandle = open_gl::GlHandle;
}

mod cerealization {
    // Yup, all cases are covered
    pub static CEREALIZER: &impl Cerealize;

    #[cfg(feature = "use_dyn_cerealizer")]
    static CEREALIZER: &dyn Cerealize = &IntoWheat;

    #[cfg(not(feature = "use_dyn_cerealizer"))]
    static CEREALIZER: &impl Cerealize = &IntoRice;
}

fn eat_cereals() {
    // No problem.
    cerealization::CEREALIZER.cerealize();
}
```

Note that, when using the imported symbol, you can only use information declared in the forward declaration. For instance, when resolving the traits implemented by `CEREALIZER`, the compiler only considers the fact that `CEREALIZER` implements `Cerealize`. Even if the compiler knows that `use_dyn_cerealizer` is turned off and therefore `CEREALIZER` is of type `&IntoRice`, the compiler isn't actually allowed to *use* that information, because it might not be true with other configs.

(this is similar to how existential types work right now)


### Trait resolution

Trait resolution is a complex beast. By contrast, I want conditional compilation to be easy to reason about; developers shouldn't have to think about convoluted scenarios where traits are resolved a certain way if some options are turned on, and resolved another way otherwise.

**Rule n°7:** Trait resolution doesn't depend on conditional compilation. Traits and implementations can be conditionally declared, but are resolved unconditionally. If an expression or a type expression needs to bind to a trait and the resolved implementation is conditionally-declared, the expression's configset must be a subset of the implementation's configset.

Or, to put it simply: Chalk shouldn't have to worry about configsets, except insofar as they can act as namespaces for symbols.

Multiple conditional bindings of the same trait to the same type aren't allowed, even if the declarations have disjoint configsets.


### Things that become impossible

With the rules as described so far, there are a few types of constructs that would no longer be possible to conditionally-compile:

- Function arguments, generic arguments, etc.
- Trait items (methods, associated types, associated consts, etc).
- Struct fields, tuple fields, enum variants.
- Defining a symbol of a different kind depending on the config.
- Basically anything that isn't a symbol but can get an `#[attribute]`.

(see [playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=3bf927bc57238138537c7ff0eb323a87) for examples of what those look like now)

I would argue that, backward-compatibility aside, most of these constructs aren't worth supporting with conditional compilation. They make the code harder to reason about for a human being, and they can usually be rewritten in a more idiomatic way.

If someone does strongly need one of these use-cases, we could add an escape hatch like `#[ignore_cfg_error]`, or a `#[stronger_cfg]` that would behave like current `#[cfg]` does. Remember, the proposed changes only serve to frontload compile errors that would happen anyway. They can degrade gracefully by producing false negatives, since ultimately the compiler will always make sure that the code it generates *is* valid for the current target.


### Cargo-level features

The rules listed above help us catch conditional compilation errors early, which covers half of the initial problem.

The other half, mutually-exclusive features, need support from Cargo.

Generally speaking, we want to be able to express:

- These features are mutually exclusive (therefore disjoint).
- The features are mutually exclusive, and one must be set (therefore a partition of the universe).
- This feature depends on this other feature.
- This feature is only available for this platform.

Using druid as an example again, the syntax might look like:

```toml
[features]
backend = {
    feature-enum = true,
    must-be-set = true,
    values = ["gtk", "x11"],
    default = "gtk",
    pre-req = 'target_os="linux"',
}
"backend.gtk" = ["gio", "gdk", "gdk-sys", "..."]
"backend.x11" = ["x11rb", "nix", "..."]

im = ["im"]
serde = { values = ["im/serde"], pre-req = "im" }
```

The operative notation is `feature-enum`, which both declares sub-features and declares them to be mutually exclusive.

This, of course, has consequences on the dependency resolution algorithm of Cargo described above, since we're declaring that some dependencies cannot be merged.

One way to address it would be to simply say dependencies with a different enum-featureset should be represented as separate compilation units, the same way semver-incompatible versions are represented now.

This would mean that types exported from the same crate with different values for enum features would be incompatible, unless they originated from a common crate (like types from semver-incompatible versions of the same crate are incompatible, unless they use the [semver trick](https://github.com/dtolnay/semver-trick).

In theory, this could make things more complicated for end-users. In practice, most of the crates with enum features would probably be top-level dependencies or close, that need a backend to be set directly by the root crate, so I expect it wouldn't be that much of a problem.

We could add a notation to require that the root crate be the one to decide the value of a feature; that would probably be convenient, though you'd need to figure out a lot of corner cases and interactions with other features beforehand. I think just doing "dependency splitting" with mutually exclusive features would already cover 90% of use-cases.


## Other corner cases

The feature I described is simplified, and doesn't cover a lot of potential cases:

- `extern` symbols.
- Default trait implementations.
- Inherent associated types.
- Macro resolution.
- Macros in general.
- `build.rs` scripts.
- Const generics.
- Post-monomorphization errors.

At first I started to write analysis about each of them, and then I remembered this is a Christmas list and not an RFC, so I get to say "Santa Claus will figure it out".

So these corner cases and any other you might think of are left as an exercise for the reader.


## Decidability

Note that a lot of the rules I described are of the form "This is only possible if the compiler can prove that this config predicate is a subset/superset of this other config predicate".

I don't really have a theoretical background here, but I'm pretty sure this is a form of the [boolean satisfiability problem](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem), which is decidable.

The SAT problem is also NP-complete, which means its theoretical complexity can be pretty high.

But I *think* most realistic use-cases can be solved with very performant algorithms? I really don't know the field enough to give a more detailed description here.


## Benefits

I've already explained the main reason I think this is a strongly desirable feature, but I think there's a lot of side-benefits as well:

- **Better incremental compilation:** Right now, different targets (eg, dev and test) must be recompiled from scratch. Delayed conditional compilation would let them share a lot of work.
- **Faster CI:** For similar reasons. Right now, to test different platforms with different features, CI has to compile the same crate multiple times, sometimes with different machines. Even multi-platform tests could be performed on a single machine with a compatibility layer like Wine.
- **Distribute MIR builds from registry:** A popular request is to have Cargo fetch crates with a pre-compiled MIR build, and run codegen directly on that build, skipping most compilation steps. While cargo-registry has already said they wouldn't implement that, a private registry could implement it (though it would need to address security concerns; reproducible builds would probably be a pre-requisite).
- **Simpler tests:** Right now there's no single command that lets you test everything in your crate. With better conditional compilation, we could rely on the fact that `cargo check --all-targets` tests everything, even things that are disabled by conditional compilation. (actually making sure unit tests pass with every possible config might be harder)

Anyway, this the first item of my "ideal Rust" wishlist. I kind of like this format; writing a formal proposal can be interesting, but exhausting. Writing with no expectation that the result be formally approved or backwards-compatible or even remotely feasible to implement in the existing compiler feels pretty refreshing.

Discussion on [r/rust](https://www.reddit.com/r/rust/comments/lkgoyk/rust_2030_christmas_list_better_cfg/).
