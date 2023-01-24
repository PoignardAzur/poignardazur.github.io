---
layout: post
title: "Rust 2030 Christmas list: Subcrate dependencies"
---

This is the fourth entry on my Christmas list for Rust 2030.

These articles are me fantasizing about what I'd like [the Rust programming language](https://www.rust-lang.org/) to be like, if we had infinite resources to implement every possible feature. I'm not worrying about "is this important to implement right now" so much as "what should the language look like in the long term".

In this article, we're gonna talk about everyone's favourite subject: build times.


## The problem

Here's a spicy hot take: compile times in Rust still need to be improved.

But don't take my word for it! According to the [2021 Rust developer survey](https://blog.rust-lang.org/2022/02/15/Rust-Survey-2021.html), 61% of developers thought they need to be improved.

Now, that was a year ago, and Rust maintainers have continued to improve the compiler in ways both big and small since them, both with small incremental changes and big leaps like enabling whole-program PGO for the compiler.

And yet, improving the compiler is a slow process, and in the meantime, developers would still like to improve their build times *now*, not after 10 years of progress.

One of the most often-suggested ways to improve build times is to split your crate into smaller units. The compiler is a mostly single-threaded beast, you see, while Cargo can use as many cores as your machine has. If you have a four-cores machine, compiling a single crate will only use a single core, while compiling four crates will use every core. Your build will go faster if your crate is split.

For a clean build, this doesn't matter too much, because the compiler will spend most of its time building dependencies, and *those* can be processed mostly in parallel whether or not your crate can (though downstream users of your crate will appreciate the added parallelism for *their* clean builds). But for an incremental build, splitting your crate adds parallelism.

(Yes, the "codegen" part of compilation is actually multi-threaded; I'm going to skip the details for brevity.)

Splitting your crates also improves caching: when deciding whether to recompile a crate, the compiler looks at all the input files and sees if any was recently modified. If at least one was, the entire crate is rebuilt, even if most of the files were unchanged. However, if you split your files among two crates, and then touch a single file, only one of the two crates will be rebuilt.

### The downsides of splitting your crate

So if splitting your crate is such a performance win, why do so few developers do it?

There are a lot of possible reasons. The [Tokio](https://tokio.rs/) developers infamously [switched back from using split crates to a single unified crate](https://github.com/tokio-rs/tokio/issues/1318). A lot of different reasons were cited:

- Having a large number of crates led to a larger maintainer burden.
- Updates in particular were harder, and needed to maintain correct dependencies between sub-crates.
- Users were confused by which subcrate was meant to be a public API and which was meant to be internal-only.
- Perversely, the large number of subcrates made users feel that the project was bloated, because they saw a large number of dependencies being loaded (even though the amount of coded being downloaded and compiled was the same in practice).

From a discussion I had with Tokio maintainer Alice Ryhl:

> Not only was it not worth the trouble - it was worse than the alternative. More work for maintainers. More confusing for users. Every time you want to release something, you have to make multiple releases which is annoying. Even now I have 8 crates on my list that I need to make releases for, and it's going to take me a while to get through that: tokio, tokio-util, tokio-stream, tokio-macros, tokio-test, tokio-metrics, slab, bytes.

Publishing a new version of a crate graph is a lot more painful than publishing a new version of a single crate.

Say you have a workspace with three crates `A, B, C` , with A depending on B and C. If you want to push a new version, you first have to change the version number of B and C, and publish them both. Then you need to change the manifest of A, which holds dependencies on B and C, and change each dependency's number; the manifest should require exact version numbers, to make sure that users of `A 1.3` don't end up using `B 1.7` on accident.

Each of these changes must be done in a specific order; cargo will complain if you try to publish a crate in a repository with uncommitted changes, so you will need to either commit between each change or suppress the error. Alternatively, you can automate the process with a custom script (though until recently, `cargo publish` returned before the crate was actually available on crates.io, so your script needed to include some sleeps).

It's a tedious and somewhat error-prone process.

And in addition, it makes your crate slightly less accessible! Users will see multiple crates on crates.io and docs.rs and may be confused as to which crate they're supposed to use in their projects; if one of your crate exports interfaces that are meant for internal use, users may still end up having access to them if they pull an intermediary crate, etc. Splitting your crate in chunks exposes internal divisions where you might want to present a unified whole.


## A proposed solution: Inline crates

Rust developer Yoshua Wuyts has [recently published an article](https://blog.yoshuawuyts.com/inline-crates/) suggesting a solution to this problem: inline crates.

> To give an example of what this would look like; I'm thinking we adopt a very similar syntax to modules:
>
> ```rust
> mod foo {}   // define an in-line module
> crate foo {} // define an in-line crate
>
> mod bin;     // import a module from a file
> crate bin;   // import a crate from a file
> ```

The hope there is that, by adding a syntax to make it as easy to add an inline crate as to add a new module, people will use these inline crates more often, and get the benefits of parallelization and incrementality.

However, when the article came out, rustc maintainers were overall opposed to the idea.

> Lang had some conversations about this: https://github.com/rust-lang/lang-team/issues/139
>
> IIRC it basically ended up as "might be nice, but seems really hard and maybe less useful that it originally seemed".

The root of the problem is that inline crate would change how crates are discovered by cargo. Right now, cargo follows an overall simple algorithm: it parses your crate's `Cargo.toml`, gets the dependency list, fetches *these* dependencies' `Cargo.toml`, and so on. Also, the root project's `Cargo.lock` pins the dependencies' numbers if present.

In that process, cargo doesn't need to read a single line of Rust code. It doesn't need to pack a Rust parser, it doesn't need to figure out `cfg` semantics, and it overall only needs to perform a single file access per crate, not one per source file.

Having inline crates would complicate that process immensely. For every crate, cargo would need to read the crate's `lib.rs` file, then iterate over `mod foobar` declarations to discover the source files. Cargo would need to perform that work for every file, whether or not any of them include subcrate declarations, to handle the case where they *do* have them.

Performance aside, this behavior is not what Cargo does right now. Implementing it would require major architectural changes that the Cargo team is absolutely not willing to implement, for obvious reasons.

So, the question is: how could we implement easy crate splitting *without* the drawbacks of inline crates?


## Introducing subcrate dependencies

Here is my proposal: we introduce a new concept called **subcrate dependencies**. These are special dependencies that refer to a filesystem path, usually in the same repository. The subcrates they point to have lightweight `Cargo.toml` files with reduced information, with the missing information being inherited from the parent crate.

```toml
# airplance/Cargo.toml
[package]
name = "airplane"
edition = "christmas-2030"

[dependencies]
regex = { subcrate = true, path = "reactor/" }
```

```toml
# airplane/reactor/Cargo.toml
[package]
name = "reactor"
edition = "christmas-2030"
subcrate = "true"
```

Now here is the important part: when the developer runs `cargo publish`, *both the parent crate and all its subcrate dependencies* are published in lockstep, as an atomic transaction (if one publication fails, all are canceled).

The subcrate will be namespaced on its parent. The subcrate's version will be the same as its parent's. In our example above, for instance, if we push `airplane@1.3.4`, we will also be pushing `airplane#reactor@1.3.4`. Though the reactor crate will be uploaded on crates.io, it won't actually be accessible except when building the airplane crate. In fact, the reactor crate will be invisible to users and will be an implementation detail of the airplane crate.

Subcrates do not appear in cargo lockfiles. `airplane@1.3.4` will always depend on `airplane#reactor@1.3.4` and never `1.3.5` or any other version.

This should drastically simplify the publishing process for crate maintainers. Instead of having to go through a whole dance of publishing crates in the right order and updating dependencies to match up, the maintainer can just set up the child crates as subcrate dependencies, and update the entire tree with a single `cargo publish` command.

And while this would still require some changes in the cargo and crates.io infrastructure, these would be relatively minor changes, exploiting what's already there. The feature would add a new type of dependency with special rules, but the general "discover all dependencies, then build them" process would stay the same.

And this wouldn't only be useful for splitting crates for performance gains. This would also help crates that inherently *need* to be separate, the big example being proc-macro crates.

### Subcrates manifests

Subcrate manifests should be lightweight, and only include the bare minimum of information.

Because subcrates are invisible on crates.io, we don't need fields such as `authors`, `description`, `homepage`, etc. Some fields such as `edition`, `rust-version`, `exclude` and `include` do provide semantic information, but we might still want to elide them most of the time.

It's unclear what mechanism we'd use here. Workspaces already provide [field inheritance](https://doc.rust-lang.org/cargo/reference/workspaces.html#the-package-table) for the fields mentioned above:

```toml
# [PROJECT_DIR]/Cargo.toml
[workspace]
members = ["bar"]

[workspace.package]
edition = "2021"
authors = ["Nice Folks"]
```

```toml
# [PROJECT_DIR]/bar/Cargo.toml
[package]
name = "bar"
edition.workspace = true # equivalent to edition = "2021"
authors.workspace = true # equivalent to authors = ["Nice Folks"]
```

We could simply encourage developers to have subcrates be part of a common workspace with the parent crate, and have them use the workspace inheritance mechanism for sharing fields. Or we could have an additional subcrate inheritance mechanism, at the risk of making inheritance less teachable harder to reason about.

We also need to figure out dependency inheritance. Again, this is something workspaces can already provide:

```toml
# [PROJECT_DIR]/Cargo.toml
[workspace]
members = ["bar"]

[workspace.dependencies]
cc = "1.0.73"
rand = "0.8.5"
regex = { version = "1.6.0", default-features = false, features = ["std"] }
```

```toml
# [PROJECT_DIR]/bar/Cargo.toml
[package]
name = "bar"
version = "0.2.0"

[dependencies]
regex = { workspace = true, features = ["unicode"] }

[build-dependencies]
cc.workspace = true

[dev-dependencies]
rand.workspace = true
```

I don't know how adequate that feature would be for subcrates: it still means that, for every dependency you want to share, you need to specify it once in the workspace, once in the parent crate manifest and once in the subcrate manifest. We might want something more concise for cases where we want to share all or almost all dependencies with a subcrate.

One simple feature would be to replace `mydep.workspace = true` with `mydep.parent = true`, with overall similar effects. This wouldn't impact teachability too much, because the attribute would mimic existing semantics.

The end goal of the subcrate feature is that moving a module into a subcrate should require very little churn. Having to re-specify dependencies might go counter to that goal. I don't know how much of a problem that would be in practice.

We could add some sort of cargo command to create a subcrate, which would mechanically generate the `dependency.parent` fields in the manifest, keeping the setup overhead low.


### Other concerns

Here are a few more design questions I can think of off the top of my head:

- **How would this affect semver?** Semver and type-compatibility shouldn't be affected. The fact that there are multiple subcrates will be an implementation detail hidden from mainstream users.
- **But then, how will Cargo select the right subcrate?** Cargo will still select a single crate per major level, as per [its current algorithm](https://doc.rust-lang.org/cargo/reference/resolver.html). Whether that crate is actually a single crate or a bundle of subcrates won't be visible to downstream users. Because each subcrate is pinned to the parent's version, there is no possibility of two semver-incompatible crates using a common subcrate.
- **Will subcrates be specified in the Cargo.lock file?** No.
- **How does this affect documentation?** The same way dependencies currently do. Right now, when compiling documentation for a crate, rustdoc pretends that its re-exported items are part of the crate. We would reuse that same feature.
- **How does this affect integration tests?** We want integration tests of the subcrates, if any, to have access to the parent crate. Also, we might want running `cargo test` on the parent to run the child's tests. Otherwise, no impact.
- **How does this affect examples?** Same as integration tests.
- **How does this affect unit tests?** We might want a subcrate's unit tests to be able to import items from the parent crate, to minimize the churn of splitting a module out. Not sure how architecturally painful that would be.


## Conclusion

Splitting your crate into smaller crates is recommended so often it's almost a platitude, but actually doing it causes a lot of churn that most maintainers aren't willing to deal with.

This article presented a possible approach to reduce that churn, in a way that requires as few structural changes to the Rust toolchain as possible.

[Discussion on r/rust](https://www.reddit.com/r/rust/comments/10k4q0c/rust_2030_christmas_list_subcrate_dependencies/)
