---
layout: post
title: Post-mortem for work on Xilem in 2024
---

Last year, Google funded me to work on [the Linebender ecosystem](https://linebender.org/), a group of projects aiming to improve user interface  design in [the Rust programming language](https://www.rust-lang.org/). I was one of several people [to get such funding](https://linebender.org/blog/xilem-2024/), and my specific focus was the Xilem crate (and later Masonry).

As 2025 begins, I'd like to take some time to reflect on the work accomplished so far, and how it compares to what we *intended* to deliver.

(This article was written in January, so it's a *little* out of date as I post it in late March, but it's still relevant.)


## The initial roadmap

### Planned work

At the time Google approached me, I sent them a rough roadmap detailing what I thought I could do if I was hired full-time for a year.

It was roughly separated into "Minimum scope", "Better scope", and "Ideal scope".

The **Minimum Scope** included:

- Port Xilem to Masonry.
- Add a list of basic widgets.
- Implement a Widget Inspector.
- Store widgets using ECS.
- Lay out the full widget tree with Taffy.
- Add convenience features for every widget:
    - Basic events: Mouse, Pointer, Click, Keyboard, TextInput, ClipBoard, Focus, Gamepad, all accessible from any widget.
    - Visual customization: border size/style/color, padding, background color/image, shadow, etc.
    - Text attributes: font size, family, style, weight, etc.
- Improve layout DX.
- Release a widely-advertised update.

The **Better Scope** included:

- Rework internals.
    - Rework event/lifecycle handling (see masonry-rs #1).
    - Rework mouse/pointer handling (see masonry-rs #2).
    - Rework IME events (see masonry-rs #3).
    - Implement child widget tracking (see masonry-rs #9).
    - Improve TestHarness (see masonry-rs #12).
    - Improve scroll portal code (see masonry-rs #15).
    - Implement a flexible message-passing system.
- Implement DevTools features.
- Implement animations.
- Write more elaborate examples:
    - Stopwatch.
    - Color picker/converter.
    - Contacts.
    - Kanban board.
    - Node Graph.
    - Panel Docks.
- Write benchmarks.
- Add "Hero Animations" (using the View Transitions web API).
- Implement Drag & Drop.
- Handle keybindings and shortcuts cleanly.
- Consider a 1.0 release.

The **Ideal Scope** included:

- Implement WebDriver protocol.
- Implement DevTools protocol.
- Create a Storybook-inspired editor.
- Implement Hot-reloading.
- Implement Record and replay.
- Benchmark compile times.

To be clear, while the "ideal scope" was clearly meant to be future work, the "minimum" and "better" scopes were both supposed to be completed by the end of 2024, which was wildly optimistic.

I'm not even gonna tell you the time estimates I made. I marked porting to Masonry as something that would be done within fifteen days, which is... not the time it ended up taking.

### Work delivered

Overall, the work we did in 2024 mainly covers four items:

- Porting Xilem to Masonry.
- Storing widgets using an arena.
- Standardizing passes to access the arena directly.
- Reworking internals.

So why is the list of delivered items so much shorter than the planned list? I'd list a few reasons:

- **Planning fallacy:** I allocated time for every item assuming that I'd be at my maximum productivity for every one of them, that each item could be delivered in the course of a focused sprint where I wouldn't work on anything else, and that documentation, public communication, bug-fixing and code reviews would take zero time. When I did perform focused sprints, I was able to implement major features in a very short time, but these sprints aren't sustainable.
- **Personal stuff:** I had a bout of depression in the spring, followed by two surgeries. My productivity was, uh, impacted.
- **Underestimating the need for reworking internals:** The "reworking internals" item was planned as something to do after all the "minimum scope" items. That was a mistake: most of my time this year was spent working on Masonry's internals. In theory it would have been possible to e.g. create new widgets using existing slightly hacky APIs, but in practice nobody wants to write code that will become obsolete.
- **Handwaving design work away:** The "switch to ECS" work had been brewing in my head for years, and it still took months to figure out the design we wanted. You can't say "it will take X days to design Y", because often the design process is about spending weeks or *months* mulling on something in the background. You can't short-circuit that process; I suspect because it needs some concepts and ideas to enter your long-term memory, which takes weeks of sustained reflection to happen.
- **A heavy RFC process:** One of those things that don't exist on paper but take a while in practice is "telling people how you want to do things". We implemented an RFC process which ate a lot of time, documentation effort, and in the end didn't deliver value worth that time and effort.

Overall, I think a fully productive Olivier with none of the personal life problems and who didn't waste any time with the RFC process could have knocked down two or three more items, and would still have fallen short of the initial plan.

A lot of the planned roadmap was blocked on having a styling feature. I've only had a solid design for styling in the last few weeks, after spending months thinking on it. Thinking on it faster or earlier would have been hard while I was working on other major architectural refactors.


## The public roadmap

In January 2024, about two months about I wrote the roadmap above, I wrote [an article](https://linebender.org/blog/xilem-backend-roadmap/) on the Linebender blog giving a much lower-detail plan for the coming year.

On re-read, I notice the article features this little gem:

> It's debatable how much this could have been avoided. As I've pointed out before, the Rust GUI ecosystem is subject to massive yak-shaving: many of us came here because we wanted to build a text editor, and now we're all learning about text rendering, text editing, compositing, accessibility trees, using monoids to implement stuff on the GPU, ECS, and some concepts that I'm absolutely certain Raph made up like Bézier paths and C++.

The yak shaving is still going strong (not going to name projects, because what counts as a yak shave is subjective), though we've started to productize a little, and Vello is now included in at least [one professional product](https://graphite.rs/).

In the case of Xilem specifically, the January 2024 article mentioned four objectives:

- Get Xilem to a usable state.
- Switch to a Masonry backend.
- Avoid custom Widgets.
- Use an ECS-inspired architecture.

### Get Xilem to a usable state

> I want Xilem to get back to being suggested to newbies in the same breath as Iced and SlintUI. In the next few years, I want the entire ecosystem to get to a point where people talk about Rust GUI like they talk about ripgrep or rustls.

We clearly haven't reached that stage yet. We're not even at a stage where Xilem could win back the mindshare that Druid lost over time.

Again, the big blocker here is a styling system. People won't ever have a good time writing a UI with Xilem if they have to stop and create a new Masonry widget every time they want to slightly change the appearance of their app.


### Switch to a Masonry backend.

> As a result, Xilem's native backend is in a poor state:
>
> - There is code commented out.
> - There are entire modules commented out.
> - There is documentation referring to items from Druid that no longer exist.
> - There are TODOs without an associated issue.
>
> Masonry's backend codebase is a healthier starting point. Masonry also comes with some built-in perks, like powerful unit tests and a structured widget graph.

Switching to Masonry was the first major refactor we undertook.

I still think it was the right decision, and the arguments above have been vindicated. Masonry today has no code commented out, has up to date documentation, and has a lot of features that would have been painful to implement in Xilem (tab focus, text selection, pointer capture).

Masonry's mission statement, as a reusable backend for multiple GUI frameworks, hasn't really be achieved yet. There are signs of early interest, and some contributors came to work on Masonry who wouldn't have been interested to work on Xilem, but overall the framework still lacks killer features that would convince other maintainers to adopt it.


### Avoid custom Widgets.

> We should write Xilem's backend under the assumption that end users of the library (including the Xilem frontend) will very rarely create their own widgets. Instead, they will usually compose the primitives given to them the same way they compose DOM elements in the browser.

This is an interesting one.

This idea got strong negative reactions, and even months after the article was published we had people coming on the Zulip server asking how X could possibly work if we didn't allow custom widgets.

Overall, the idea was quietly dropped, and the Widget trait is still exposed, with detailed documentation on how to implement it, and I expect things to stay that way for the foreseeable future.

On the other hand, it does feel like some of our design decisions are moving towards allowing less of *certain* types of customization in widgets. The new pass systems handles recursion directly instead of having `ParentWidget::event()` call `ChildWidget::event()`, some behavior like tab focus and changing pointer icons that previously had to be reimplemented in every widget is now standardized, etc.

The upcoming style and layout redesigns will also likely drift towards more standardization of behavior. We'll still allow a lot of customization (some users *really* need it), but I expect we'll be a lot more deliberate about where that customization happens.


### Use an ECS-inspired architecture.

> I believe Widgets should be owned by the library. If your container has children, then the only thing the container will actually own is keys into a structure (probably a slotmap) where the widget is stored. This makes a lot of things easier, like serialization and debugging, but it has an impact on the entire backend. It's an infrastructure investment.

We made the switch from "Widgets own the childrens" to "Widget own keys" over the early summer, and then spent the rest of the year handling the changes that fell out of that.

I still think it was a massive win. Masonry's pass handling code is more concise, clean and readable than it ever was. Add new features with that architecture has proved to be super easy and fun.

As part of that refactor we created the `tree_arena` crate, which lets us store some items in a tree-like data structure and access them efficiently. The main access pattern it enables is "mutating an item's children while keeping a live reference to the item's value", something we need to do a lot in widget passes.

The initial implementation of the arena used an actual tree, which wasn't great for performance, but could be implemented with safe code only. Contributor KGrewal1 recently added an unsafe implementation storing all items in a single HashMap, which lets us have `O(1)` access to any item in the tree. Ideally, the final implementation would use a slotmap instead.

A task that remains is implementing the "Component" part of ECS. Especially for styling, we want to efficiently associate values of arbitrary types with any widget, and efficiently access them.


## Conclusion

Before writing this article, I felt a little uneasy about the work we achieved in 2024. We promised a lot, and I wanted to examine why we didn't deliver as much before I went on making any more promises.

After writing this, I think the larget factor is that we underestimated how much architecture work we'd end up doing. That makes me feel better, because I think we only need about three major architural changes (styling, layout, widget creation) before Masonry reaches its final-ish design.

I'm super excited about 2025, and I'm hopeful we'll make some major strides in the coming months.
