---
layout: post
title: Report on platform-compliance for cargo directories
---

When you use [the Rust programming language](https://www.rust-lang.org/)
toolchain, usually through a cargo command, it needs a place to store a bunch of
config files, caches, and the cargo binary itself. By default, that place will
be your operating system's user directory, which I'm going to refer to as
`$HOME` or `~`, where it will put a `.cargo` folder.

This is a very common approach for Unix projects, but not the preferred approach
of any of the platforms Rust is available on. Each platform has its own folders
that it expects apps to store different types of data in, with various degrees
of compliance.

Platform compliance is a subject that pops up every now and then, usually with
some frustration that Rust still doesn't have it. Here's an overview of the main
threads I've found on the subject:

- [Original PR on github (2014)](https://github.com/rust-lang/cargo/pull/148)
- [Original cargo issue on github (2015)](https://github.com/rust-lang/cargo/issues/1734)
- [Second implementation PR on github (2015)](https://github.com/rust-lang/cargo/pull/2127)
- [Sibling issue open in Rustup (2016)](https://github.com/rust-lang/rustup/issues/247)
- [RFC opened to pin down semantics before merging a PR (2016)](https://github.com/rust-lang/rfcs/pull/1615)
  (this is where the bulk of the discussion is).
- [People on reddit commenting on the decision to postpone the RFC (2016)](https://www.reddit.com/r/rust/comments/636qbr/rfc_fcp_close_no_platformspecific_directories_for/)
- [Thread about a proposed workaround (2016)](https://www.reddit.com/r/rust/comments/66g5by/tricking_rust_into_following_the_xdg_base/)
- [Third implementation PR (2018)](https://github.com/rust-lang/cargo/pull/5183)
- [Fourth implementation PR (2020)](https://github.com/rust-lang/cargo/pull/8063)
- [Fith implementation PR (2021)][https://github.com/rust-lang/cargo/pull/9178]
- [Second RFC attempt (2021)](https://github.com/spacekookie/rfcs/pull/1) (note
  that user spacekookie is also the one that wrote these two PRs above)

I am leaving out various people complaining about Rust's XDG non-compliance on
[reddit](https://www.reddit.com/r/rust/comments/lrqsbh/comment/googdw0/?utm_source=reddit&utm_medium=web2x&context=3),
[Hacker News](https://news.ycombinator.com/item?id=35285858) and
[Github](https://github.com/rust-lang/rustup/issues/3256) (and I probably missed
quite a few threads). As you can see, many of these threads have high vote
counts: people wanted this in 2014, and they still want it now.

As the saying goes, "the best time to plant a tree is back in 2014 when the
language was unstable and breaking changes weren't much of a concern"; and
indeed, going through these threads, you can feel the frustration of a bunch of
users asking for the change to be implemented as fast as possible before the
ecosystem ossifies, being ignored, and years later being told that it's too
costly to implement now that the ecosystem has ossified.

That frustration can get pretty close to resentment and hostility. Indeed,
speedrunning through past discussions, it can feel like platform-compliance
skeptics' position is a textbook application of the OSS's _Simple Sabotage Field
Manual_: "insist on doing everything through channels, never permit short-cuts
to be taken in order to expedite decisions", "haggle over precise wordings of
communications, minutes, resolutions", "refer back to matters decided upon at
the last meeting and attempt to re-open the question of the advisability of that
decision".

Now, there is an important point I want to make: the previous paragraph is not
me accusing developers who favor the status quo of deliberately making the
process harder. This might be a reasonable belief in a corporate environment,
but Rust is a high-trust open-source community, where we have the privilege of
being able to assume good faith.

Rather, I think that developers resistant to platform-compliant config files
have concerns; some of which I feel are valid, some which are basically noise;
and that despite the eagerness of the community for a fix, and the attempts of
multiple members to implement ones, nobody was able to propose a solution that
truly addressed those concerns.

Part of this is bureaucratic entropy (if skeptics oppose the same minor concerns
over and over again, eg because they are not aware someone else already raised
them, discussion inevitably gets bogged down), part of this is that the major
concerns are _actually pretty complex_.

And since the second best time to plant a tree is right now, I'd like to lay out
those concerns, and exactly what it would take to implement platform-compliance
for cargo in 2023 (and whatever year you're reading this).

## Okay, what are you even talking about?

If you're not familiar with the subject matter, this introduction might have
been a bit unclear to you. To get a better idea of what the fuss is about, let's
get into low-level details.

When you run

```sh
cargo build
```

into your terminal, cargo needs a bunch of configuration values to figure out
what it wants to do; things like "do I pass additional flags to the compiler?",
"which linker do I use?", "what is the default optimization level?", etc. It
will look for config files in the current directory, every parent directory, and
in a special path, `~/.cargo/config`.

Moreover, cargo also wants to keep a cache of previously built crates,
downloaded source files, credential tokens, etc. All of these files will be
stored in paths like `~/.cargo/git`, `~/.cargo/credentials`, etc.

And cargo also keeps both the rust toolchain (`cargo` itself, `rustfmt`,
`rustc`, etc) and all the globally-installed binaries it builds in
`~/.cargo/bin`. That directory is usually added to your `$PATH` on first
install, so after running `cargo install somerustprogram`, you can run
`somerustprogram` directly from your terminal like any other command.

(Also, rustup has its own config files in `~/.rustup`, which we won't cover
here; the general principle is the same.)

Currently, you can change the `~/.cargo` part by setting the environment
variable `$CARGO_HOME`. If you set `CARGO_HOME=/foo/bar`, then your global
config file will be in `/foo/bar/config`, your credentials file will be in
`/foo/bar/credentials`, etc.

However, there is no way to "split" the cargo home. There is no way to say: I
want config files to be in one folder, temporary files to be in another, and
binaries to be in yet another.

Except this is exactly what some users want! On Linux in particular, there is an
increasingly popular standard called
[XDG Base Directory](https://wiki.archlinux.org/title/XDG_Base_Directory):

- `XDG_CONFIG_HOME`: Where user-specific configurations should be written
  (analogous to `/etc`). Should default to `$HOME/.config`.
- `XDG_CACHE_HOME`: Where user-specific non-essential (cached) data should be
  written (analogous to `/var/cache`). Should default to `$HOME/.cache`.
- `XDG_DATA_HOME`: Where user-specific data files should be written (analogous
  to `/usr/share`). Should default to `$HOME/.local/share`.
- `XDG_STATE_HOME`: Where user-specific state files should be written (analogous
  to `/var/lib`). Should default to `$HOME/.local/state`.
- (Non-standard, but popular) `XDG_BIN_HOME`: Where user-specific executables
  should be written (similar to `/usr/bin`). Default to `$HOME/.local/bin`.

(Mac, Windows, and other OSes have their own conventions, which I'm less
familiar with.)

It's controversial how much benefit these conventions really bring. Some people
really don't care, some people swear by them.

Some benefits people have cited:

- Reducing the home directory clutter, making it easier to find relevant files.
- Having all config files centralized in an easily backed-up `~/.config/` folder
  with no clutter.
- Having all cache files centralized in an easy-to-destroy `~/.cache` folder.
- Having all local binaries in a single folder which is "guaranteed" (your
  distribution may disagree) to be included in the default `$PATH`.
- Following standard paths makes it easier to cooperate with sandboxing
  frameworks like Flatpak (though it doesn't seem very tidy in practice).
- More abstract benefits, like giving analysis and backup tools a better
  understanding of your folder structure.
- Windows doesn't use XDG paths, but has its own standard paths, and rarely uses
  dotfiles. Having dotfiles used for Windows may be confusing and annoying for
  users.

Speaking as someone who frequently has to re-install my environment, the first
two points speak to me: I really wish app developers stopped having their own
special top-level folder for all data that I must then track down and manually
copy when I'm trying to port my configuration to a new laptop / PopOS install /
whatever. At one point I thought NixOS would be the solution, but, eh,
[it didn't pan out](https://poignardazur.github.io/2021/09/08/nixos-post-mortem/).

Also, the supposed benefits I listed strongly depend on network effects: people
will only write more tools based on XDG assumptions if developers write
xdg-compliant apps; developers will be more motivated to write xdg-compliant
apps if there is an ecosystem benefit to doing that. This can lead to
frustrating situations where maintainers of a few major projects stall and point
at each other to say "If X didn't bother to do it, why should I?". That said, in
2023, I'd say
[support has progressed enough](https://wiki.archlinux.org/title/XDG_Base_Directory#Support)
that it's clear the ecosystem is heading towards widespread support (if at a
glacial pace).

## So what is blocking us?

Backward compatibility.

It's a thornier problem than you may think.

First, before the maintainers make any widespread switch, they must make sure
that it won't break ecosystem tools. Say we change the cargo config file to be
stored in `~/.config/cargo/config`. If tools like
[cargo-geiger](https://github.com/rust-secure-code/cargo-geiger),
[bacon](https://github.com/Canop/bacon), or
[tarpaulin](https://github.com/xd009642/tarpaulin/) were using the hardcoded
legacy path to look for cargo config data, they'd break as soon as you'd change
your directory structure.

Second, Rust developers often use the `rustup` installer to switch between rust
versions. If they switch back to a previous version of cargo which only knows to
look for `~/.cargo`, it won't read the config files in your new emplacement.

The second point is, as far as I'm aware, the one that all RFCs and PRs so far
have failed to address. It's _complicated_, the kind of problem that requires a
multi-step rollout to solve, where maintainers make sure to proactively notify
tool authors, stay on alert for breakage reports, be quick to provide hotfixes,
etc. Given that many maintainers are skeptical of the benefits I've listed
above, they have understandingly not felt keen to accept this workload.

A third point, less complicated but still time-consuming, is that a lot of tests
in cargo and rustup are written under the assumption that config files can be
found under `~/.cargo`. These tests will need to be fixed before any
wide-reaching change is implemented. I'm not actually sure what needs to be
fixed;
[the discussion in the RFC thread mentions `cargo clean` and hardcoded paths](https://github.com/rust-lang/rfcs/pull/1615#issuecomment-803558170).

## What should we do?

So, assuming we do want platform-compliant cargo files, which I do, what should
we do?

Step zero would be to write an RFC outlining everything I'm about to say in a
semi-formal format. [RFC #1615](https://github.com/rust-lang/rfcs/pull/1615) is
a good starting point, but has missing steps.

### Set up a new directory structure

First, we have to establish three directories where cargo files will be stored:

```
CARGO_CACHE_DIR
CARGO_CONFIG_DIR
CARGO_BIN_DIR
```

To quote RFC #1615:

> These will be used to split the current `.cargo` (`CARGO_HOME`) directory up:
> The cached packages (`.cargo/git`, `.cargo/registry`) will go into
> `CARGO_CACHE_DIR`, binaries (`.cargo/bin`) installed by Cargo will go into
> `CARGO_BIN_DIR` and the config (`.cargo/config`) will go into
> `CARGO_CONFIG_DIR`.

We will also define three functions in the `cargo` library:

```
cargo::cache_dir()
cargo::config_dir()
cargo::bin_dir()
```

During the initial implementation, `cache_dir()` and `config_dir()` will return
`$HOME/.cargo` and `bin_dir()` will return `$HOME/.cargo/bin`, unless their
respective environment variable is set (we're deliberately ignoring
`$CARGO_HOME` for now). That way, the default behavior should stay the same at
first, while giving users the option to have a platform-compliant directory
structure.

After a transition period, and depending on a lookup algorithm (see next
section), these functions may instead return platform-dependent values:

```
WINDOWS:
cache:    AppData\Local\Temp\Cargo
config:   AppData\Roaming\Cargo
binaries: AppData\Local\Programs\Cargo

LINUX, BSD:
cache:    .cache/cargo
config:   .config/cargo
binaries: .local/bin

MAC OS:
cache:    Library/Caches/org.rust-lang.Cargo
config:   Library/Application Support/org.rust-lang.Cargo
binaries: /usr/local/bin
```

(Actual values will depend on some runtime stuff: API calls on Windows and
MacOS, `XDG_***_HOME` environment variables on Linux, etc.)

### Agree on a canonical lookup algorithm

Given its environment, how should cargo pick a path for its files? In other
words, what should eg `cargo::config_dir()` return?

This is a question with no straightforward answer.
[RFC #1615 says](https://github.com/tbu-/rust-rfcs/blob/0215ab6cb5857cd31d791e07e56c7bf238e174a1/text/1615-cargo-platform-standards.md#detailed-design):

> In order to maintain backward compatibility, the old directory locations will
> be checked if the new ones don't exist. In detail, this means:
>
> - If any of the new variables `CARGO_BIN_DIR`, `CARGO_CACHE_DIR`,
  > `CARGO_CONFIG_DIR` are set and nonempty, use the new directory structure.
> - Else, if there is an override for the legacy Cargo directory, using
  > `CARGO_HOME`, the directories for cache, configuration and executables are
  > placed inside this directory.
> - Otherwise, if the Cargo-specfic platform-specific directories exist, use
  > them. What constitutes a Cargo-specific directory is laid out below, for
  > each platform.
> - If that's not the case, check whether the legacy directory exists (`.cargo`)
  > and use it in that case.
> - If everything else fails, create the platform-specific directories and use
  > them.
>
> This makes Cargo use platform-specific directories for new installs while
> retaining compatibility for the old directory layout. It also allows one to
> keep all Cargo related data in one place if one wishes to.

The problems with the above algorithm are:

- It doesn't specify what to do if some but not all of `CARGO_BIN_DIR`,
  `CARGO_CACHE_DIR`, `CARGO_CONFIG_DIR` are set.
- It doesn't specify what to do when one of the above flags is set, but the
  matching file doesn't exist.
- It assumes that `CARGO_***_DIR` or `CARGO_HOME` being set is significant and
  means "override the default"; except when using cargo (or basically any rust
  binary) through rustup, `CARGO_HOME` is _always_ set. Rustup sets it to its
  default value of `$HOME/.cargo` if it's not set before calling your rust
  binary. (Looking through the code responsible for this, it's not clear to me
  how feasible it would be to not do that.)
- For that matter, it doesn't account for rustup directories, though they have
  fewer special cases.

The problem of rustup in particular is tricky. If you run:

```sh
cargo build
```

from a distribution where Rust is installed by your package manager, then
chances are your command indeed will run the `cargo` executable. However, if you
run the same command on a system where Rust was installed with rustup, then
there's a good chance
[the `cargo` binary is a hardlink to the `rustup` binary](https://github.com/rust-lang/rustup/blob/86ed53c6e3c77234d8c4c5ea410ab3624036c4ee/src/bin/rustup-init.rs#L3-L8)
(likely stored in `~/.cargo/bin`), which will do some light preprocessing,
figure out what toolchain you want to use, and call the appropriate version of
the _actual_ `cargo` binary. This is how rust commands handle these
`+sometoolchain` flags you can give them.

For our problem, the implication is that cargo may run directly, or through a
rustup proxy which adds a default value to `$CARGO_HOME` (and presumably
`$CARGO_CACHE_DIR`, `$CARGO_CONFIG_DIR`, etc). This means our lookup algorithm
needs to work for both scenarios.

**Here is the algorithm I would propose:**

- If `$CARGO_HOME` is set:
  - For each of `$CARGO_CACHE_DIR`, `$CARGO_CONFIG_DIR`, `$CARGO_BIN_DIR`:
    - If the env variable is set, use that value.
    - Else use `$CARGO_HOME` (or `$CARGO_HOME/bin`).
- If `$CARGO_HOME` isn't set:
  - Set `$CARGO_HOME` to `$HOME/.cargo`.
  - For each of `$CARGO_CACHE_DIR`, `$CARGO_CONFIG_DIR`, `$CARGO_BIN_DIR`:
    - If the env variable is set, use that value.
    - Else if the platform-specific directory (eg `$HOME/.cache/cargo`) exists,
      use that directory.
    - Else use `$HOME/.cargo`.
- When running rustup, apply the above steps, then pass the environment
  variables that have been set to the wrapped binary.
- There are some subtleties in practice where rustup wants to be able to mock
  environment variables for unit tests, which we won't cover here.

Note that this algorithm doesn't specify whether the directories are created in
`.cargo` or in platform-compliant directories. Indeed, the default outcome of
using that algorithm as stated, is _not_ using platform-specific directories:
the outcome if config files don't exist is to use `$HOME/.cargo`; but config
files are usually created only when Rustup or cargo first tries to write them
(for instance, `registry/` is created after your first `cargo build` starts
downloading dependencies). But to know where to write them, it needs to follow
the above algorithm, which depends on whether or not these files exist. Hence,
by default, it would assume files need to be created in `.cargo`. This is
deliberate.

(Existing proposals ignore this subtlety; they assume that all config files are
created once at install time, whereas cargo potentially recreates them on each
run, which adds some corner cases.)

A possible change would be to use platform-compliant directories as the ultimate
fallback instead of `.cargo`, though I would recommend against that at first, to
make the transition easier.

Instead, rustup could create platform-specific directories (eg
`$HOME/.cache/cargo`, `$HOME/.config/cargo`) on fresh installs; we could also
create a `rustup migrate-home` command that creates these directories and
migrates existing config files. **This migration should be super optional**, not
eg something that's applied on `rustup update`.

Also, we can assume that package manager installs would create the
platform-specific dirs by default.

### Compatibility with earlier Rust versions

The basic point of rustup is to be able to very easily switch between rust
versions.

This means users expect to be able to run `cargo +v1.50 build` and, assuming
they've installed a toolchain named `v1.50`, to run the matching cargo binary;
since this cargo binary will not have support for platform-specific directories,
rustup will need to mock it somehow.

The most obvious solution would be symlinks: when rustup installs a toolchain
from a previous version, it will look in its `$CARGO_HOME` directory and add
various links pointing to platform-specific files and folders (eg
`$CARGO_HOME/registry` would point to `$CARGO_CACHE_DIR/registry`), assuming
they exist.

(There are some platform subtleties here; for instance, rustup prefers using
hardlinks on Windows IIRC)

The potential existence of these links would add some complexity to the rustup
uninstall process, since it would need to remove both them and the actual files
separately; but as long as these symlinks are only touched on toolchain install
and uninstall, they shouldn't add too much brittleness to the process. In
particular, the lookup algorithm I described above isn't affected by the
existence of symlinks.

### Rustup directories

So far we've discussed cargo env variables passed to `cargo`, and cargo env
variables passed to `rustup`, but rustup also has its own env variables to
manage!

Rustup currently uses `RUSTUP_HOME` and defaults to `$HOME/.rustup`. It would
need its own `RUSTUP_CACHE_DIR` and `RUSTUP_CONFIG_DIR` env variables.
Presumably they would be set in a way consistent with `CARGO_HOME` and
`CARGO_***_DIR`, but the code should not assume that this is the case.

The algorithm would be the same as the one I described for cargo. Rustup is
overall a simpler case, because some of the corner cases (downgrading to
previous versions, custom tools fetching hardcoded file locations) are less
likely to happen.

## Some unresolved questions

### Do we really want CARGO_BIN_DIR?

It's not clear to me that having a separate directory for binaries has a strong
benefit. Separating config files from cache data is useful for backups and
storage, but separating binary files is only useful for adding them to the
`$PATH`. Except rustup can already add the binary folder to the `$PATH`
variable, and it's not clear how many platforms have a standard
binaries-go-there folder. It's not part of the main XDG standard, for instance.

And it adds some non-trivial difficulties: using, say, `~/.local/bin` for cargo
binaries would mean that binaries would be mingled with binaries from other
programs. Rustup couldn't remove all the contents of the directory like it
currently does with `~/.cargo/bin`.

Also, the symlinking trick I described above gets a lot more complicated if
rustup has to guess which binary is or isn't owned by cargo.

Given the above constraints, it might make sense to consider the `bin/`
directory a sub-part of the cache directory.

### Split cache and data?

To get some of the benefits of platform compliance I mentioned earlier in the
post, you have to split your application's data directory, which stores files
you need to keep around, and your cache directory, which stores files that can
be removed at any point and will be re-generated.

I'm not sure how feasible it would be to do this for cargo. The directory
structure doesn't really separate files that can be regenerated on demand and
files that would need to be downloaded if deleted.

We could come up with a new directory structure; but the transition between old
and new versions would get even _more_ complex. We should probably keep it
simple at first.

## Preparing the transition

To avoid ecosystem-wide breakage, we would have to do a survey of existing
cargo-based tools and check how they handle config discovery. Some projects I
got from a quick search:

- cargo
- clippy, rustfmt, rust-analyzer
- cargo-audit
- cargo-auditable
- cargo-crev
- cargo-deny
- cargo-edit
- cargo-expand
- cargo-xbuild
- bacon
- cargo-geiger
- cargo-dinghy

Generally, we'd also want to do a github-wide search for `CARGO_HOME` and
`RUSTUP_HOME`, look through the reverse dependencies of
[the `home` crate](https://crates.io/crates/home/), and probably a few I forgot.
Yeah, that's a lot.

I expect that most of these projects don't use cargo configs, and won't need to
be changed; but due diligence requires that we check most of them before we
perform a change that has the potential to break the entire ecosystem.

For all these projects, we need to look for hard-coded cargo paths, and replace
them with `cargo::bin_dir()`, `cargo::cache_dir()` and `cargo::config_dir()`.
(Well in practice, we'd probably add these methods to the `home` crate, but you
get the idea.)

Note though, that we could implement a lot of the changes I've described before
doing most of that due diligence, as long as we warn users that `CARGO_***_DIR`
and `RUSTUP_***_DIR` environment variables are experimental and might cause some
breakage. The default behavior would stay the same as now.

Before merging any changes, though, what we would need is _tests_, and lots of
them. I don't know what the testing situation looks like in rustup right now,
but for a change that broad we'd probably want integration tests that simulate a
full environment on multiple platforms, so probably containers or something.

There are a lot of corner cases we'd want to sand off before we'd be comfortable
merging that change.

## Conclusion

Is it worth all that effort to implement platform compliance in cargo and
rustup?

I'm a lot less confident than I was when I started writing this article. The
amount of work seems daunting, and I have a lot more empathy for the maintainers
who looked at it and went "nope, not touching that". I'm certainly not rushing
to implement it myself.

That said, I do still feel the same frustration when I do `ls ~` and see the
little `.cargo` folder taunting me, polluting my home. I do think it would be
better in the long term if someone had the motivation to do all that work, cross
the Ts and dot the Is, and actually planted that tree.

Just, you know. Be aware that there's a lot of work before we can actually get
its fruits.

[Discussion on r/rust](https://www.reddit.com/r/rust/comments/13plkf0/report_on_platformcompliance_for_cargo_directories/)
