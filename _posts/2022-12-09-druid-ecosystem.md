---
layout: post
title: Status of the Druid ecosystem in December 2022
---

[Druid](https://github.com/linebender/druid) is an experimental GUI framework for [the Rust programming language](https://rust-lang.org).

Rust is a... pretty unique language, to say the least. Because of the constraints it adds to your program (no aliased mutable references, data structures don't have parent pointers, etc), writing GUI in this language is a major challenge. Common design patterns (eg the observer, where widgets will run an arbitrary callback when their value changes) become very hard to use, by design.

The Rust ecosystem is still trying to create a GUI framework that works within these limitations, but it's been an uphill battle. There are many projects attempting to solve these problems, but the biggest ones I'm aware of are [Iced](https://github.com/iced-rs/iced), [Slint](https://github.com/slint-ui/slint), and Druid.

Druid initially started as a subproject of Raph Levien's [Xi editor](https://github.com/xi-editor/xi-editor), an attempt to write a high-performance, low-latency IntelliJ-style text editor.

Raph [was unsatisfied with the GUI tools at his disposal](https://raphlinus.github.io/xi/2020/06/27/xi-retrospective.html), and started spending more time on the `xi-win` subproject, which eventually became Druid.

Fast-forward six years later, and an entire ecosystem has grown around Xi and Druid. That ecosystem is fast-moving: the same perfectionism that led Raph and others to abandon Xi for Druid led the community to progressively abandon Druid for other, lower-level projects.

The yak-shaving is very real here, and I'll be the last person to throw stones: I did the same thing when I set aside [Panoramix](https://github.com/PoignardAzur/panoramix) (a framework I built on top of Druid) to work on [Masonry](https://github.com/PoignardAzur/masonry-rs) (a framework I'm building to replace Druid).

Part of the reason we're all starting new projects left and right is this perfectionism I mentioned: when you're working in a green field such as Rust, there's a strong desire to "get it right" the first time. A lot of Rust crates beat the state-of-the-art because they were designed from the ground up, to do one thing and do it well, and don't have to carry the technical debt of previous projects. I think I speak for most of the Druid community when I say this is something we'd like to achieve for GUI as well.

But this tendency to jump from project to project does make the ecosystem pretty confusing to newcomers, like a shared comic book universe after three or four continuity reboots.

This article aims to add some clarity to the current state of affairs. It's a list of Druid-adjacent projects sorted (more or less) chronologically, with a brief summary of each project, what its current status is, and whether updates can be expected.

**Update:** As of February 2023, a lot of this is already outdated. However, the title of the article goes "in December 2022", so I'm not going to rewrite everything. I've added updates when projects have changed or been renamed.

## Projects

### Xi editor

**Link:** <https://github.com/xi-editor/xi-editor>

**Summary:** An attempt to build a high quality text editor, using modern software engineering techniques. It was initially built for macOS, using Cocoa for the user interface.

There's also [a list of development articles](https://xi-editor.io/docs/frontend-notes.html) discussing editor-related design elements.

**Status:** Discontinued. It has a spiritual successor named [Lapce](https://github.com/lapce/lapce), based on the same technologies.


### Piet

**Link:** <https://github.com/linebender/piet>

**Summary:** A cross-platform API for PostScript-style 2D graphics.

This crate implements an API for stateful drawing over multiple backends, with methods like `stroke`, `fill`, `clip`, `draw_text`, etc.

**Status:** Actively maintained.


### Skribo

**Link:** <https://github.com/linebender/skribo>

**Summary:** A library providing low-level text formatting services; intended to be a thin layer around [harfbuzz](https://harfbuzz.github.io/), and as such to provide low-level text shaping.

**Status:** Discontinued.


### Kurbo

**Link:** <https://github.com/linebender/kurbo>

**Summary:** Data structures and algorithms for computing curves and vector paths. It's mainly used for rendering, but is general enough for other applications.

**Status:** Actively maintained.


### Druid

**Link:** <https://github.com/linebender/druid>

**Summary:** An experimental Rust-native GUI toolkit.

Druid is an opinionated GUI framework that lets developers build their UI in a declarative way with high-level abstractions, and then handles the dataflow and lower-level details to show a window to the user.

Druid uses Piet and Kurbo internally for rendering.

**Status:** In limbo. Druid's maintainers have been unsatisfied with the crate's API and main abstractions for years, and progress has slowed as a result. The current plan is to put out a final release before the end of the year, with bugfixes and documentation fixes, and then leave the project in maintenance mode.

Druid will eventually be superseeded by Xilem and/or Masonry.

(There were also two experimental spinoff projects, [crochet](https://github.com/raphlinus/crochet) and [lasagna](https://github.com/linebender/druid/tree/lasagna/lasagna), which I'm not including in this list. Both of those laid ground for what eventually became Xilem and Masonry.)

**Update:** Druid's 0.8 release has come out, and Druid is now officially in maintenance mode.


### Runebender

**Link:** <https://github.com/linebender/runebender>

**Summary:** A font editor built with Druid.

This project was sponsored by Google Fonts who paid Colin Rofls to develop it, thereby making it one of the main drivers for Druid development.

**Status:** Postponed. Colin is currently working on Oxidize.


### Panoramix

**Link:** <https://github.com/PoignardAzur/panoramix>

**Summary:** A prototype implementation of reactive GUI built on top of Druid.

The goal of the project is to build a react-like framework (one where the GUI is mostly built from "components", functions returning trees of lightweight widgets) in the Rust language.

**Status:** Postponed. I'm currently working on Masonry, and hoping to ultimately refactor Panoramix to run on top of Masonry instead of Druid.


### Masonry

**Link:** <https://github.com/PoignardAzur/masonry-rs>

**Summary:** A low-level implementation of a GUI framework in Rust.

The goal of Masonry is to separate the higher-level app-developer-facing API from the lower-level handling of widget logic. Masonry is not intended to be used by app developers; instead, it's meant to be included in other GUI frameworks (like Panoramix, or maybe Xilem or Iced) and to handle the widget tree for them.

**Status:** In development.

**Update:** [Masonry 0.1 has been released.](https://poignardazur.github.io/2023/02/02/masonry-01-and-my-vision-for-rust-ui/)


### Xilem

**Link:** <https://github.com/linebender/xilem/>

**Summary:** An experimental architecture for Rust GUI.

Xilem takes inspiration from Panoramix and other languages' frameworks like Flutter, SwiftUI and Elm.

Ideally, Xilem is meant to eventually replace Druid as the main GUI framework being developed by the community.

**Status:** In development.


### Piet-GPU

**Link:** <https://github.com/linebender/piet-gpu>

**Summary:** Prototype for a compute-centric 2D GPU renderer.

The goal is to try to use advanced GPU features (subgroups, descriptor arrays) to improve hardware acceleration of 2D rendering in a portable way. See [the vision doc](https://github.com/linebender/piet-gpu/blob/main/doc/vision.md) for details.

Its unclear what this project would mean for the baseline Piet project. There has been some talk of eventually replacing Piet with Piet-GPU in Druid/Xilem once Piet-GPU would be mature. That said, my understanding is that the two projects have drifted apart quite a bit, and Piet-GPU's API is now very different from Piet's.

**Status:** In development.

**Update:** Piet-GPU has been renamed and is now called "Vello".


### Glazier

**Link:** <https://github.com/linebender/glazier>

**Summary:** An operating system integration layer for GUI toolkits.

Glazier is similar to [winit](https://github.com/rust-windowing/winit): its job is to open windows and handle their events, while the question of which pixels get rendered where is left to other crates. Glazier aims to handle more features and corner-cases than winit.

Glazier is an offshoot of Druid's druid-shell subcrate. Whereas druid-shell is coupled with the Piet rendering API, glazier intends to be render-agnostic.

**Status:** In development, still unstable.


### AccessKit

**Link:** <https://github.com/AccessKit/accesskit>

**Summary:** A foundational crate that defines the types of data required to make a UI accessible to screen readers and other assistive technologies.

While there are many GUI toolkits popping up in the Rust language, virtually none of them handle assistive technologies. AccessKit aims to provide an API to facilitate handling accessibility interactions in those toolkits.

**Status:** In development. (AccessKit support [was recently merged in egui](https://github.com/emilk/egui/commit/e1f348e4b24c2fa83d25c6a7ddfd9b38b85de161))


### Oxidize

**Link:** <https://github.com/googlefonts/oxidize>

**Summary:** Low-level font processing, mostly focused on parsing font files.

**Status:** In development.


### Swash

**Link:** <https://github.com/dfrg/swash>

**Summary:** Cross-platform font introspection, complex text shaping and glyph rendering. 

It covers work that is traditionally performed by [FreeType](https://freetype.org/) (rasterization) and [HarfBuzz](https://harfbuzz.github.io/) (shaping).

**Status:** In development.


### Parley

**Link:** <https://github.com/dfrg/parley>

**Summary:** Rich text layout engine built on the swash crate.

Whereas Swash handles characters and words, Parley handles paragraph-level layout.

(An implementation of the piet API using Swash is in progress.)

**Status:** In development.


### Lapce

**Link:** <https://github.com/lapce/lapce>

**Summary:** Cross-platform text editor built with Druid and taking inspiration from Xi.

Like all modern editors, it supports features such as code completion, diagnostics and code actions. It has plugins, modal editing, remote development, plugins, and a built-in terminal.

**Status:** In development.

(The author intends to switch to Xilem as soon as the framework is mature enough.)


## Conclusion

Whew!

That was a lot of projects! It might be a bit hard to get them all straight, and to understand how one project relates to another, so I drew a chart of what, ideally, the xilem-adjacent tech stack might look like down the line:

![Tech stack of Rust GUI crates](/assets/druid_gui_stack.svg){:class="img-responsive"}

(Note that this in an *informal* illustration. Just because a tool is in this diagram doesn't mean its author agrees with me about the tool's future and how it relates to others in the stack.)

Hopefully this article gave you a better understanding of the current state of this little corner of the Rust GUI ecosystem.

I know it may appear like there's a lot of churn, but I'll repeat what I said in the intro: Rust developers are perfectionnists. Rust GUI is a relatively untamed field, and we're all striving to find the right abstractions and composition boundaries to make tools that are truly powerful and reusable between projects.

Doing so is a long process that takes a lot of trial and error, but we're still hammering away at it!

If this has motivated you to help work on one of the libraries mentioned above, we have [an active community on Zulip](https://xi.zulipchat.com) that can answer all your questions and will be delighted to hear from you. Good coding!

[Discussion on r/rust.](https://www.reddit.com/r/rust/comments/zh0k2k/status_of_the_druid_ecosystem_in_december_2022/)
