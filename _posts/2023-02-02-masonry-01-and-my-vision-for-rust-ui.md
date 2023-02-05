---
layout: post
title: Announcing Masonry 0.1, and my vision for Rust UI
---

When I see the landscape of native GUI in 2022, I feel like something is missing.

I don't just mean Rust UI. My frustrations with UI frameworks started long before I'd even heard of Rust.

## The origin story: Qt and fear

The [Qt framework](https://www.qt.io/) is a C++ toolkit for writing GUI apps.

In 2019, I spent a year working on a Qt project for an energy company, a diagram editor meant to be used by electrical engineers. I was brought in as a consultant late in the project, to fix bugs and add small features before the product shipped. I had already worked on Qt projects before, but they were mostly amateur projects, toy examples and the like. This was the first GUI project of industrial scale that I worked on, and it was fascinating and/or infuriating to see that my frustrations with Qt still applied.

Working on an existing Qt codebase was *slow*.

As a programmer, when you want to tweak a feature in an existing project, you have to follow a bit of a process:

- First, you find what part of the code is responsible for the feature. If you didn't code it yourself, this is by far the part that takes the most time.
- Then you have to read and understand that code, and maybe figure out what other bits of code depend on it.
- Then you have to change the feature to do what you want, and cross your fingers and hope you didn't break some completely unrelated part of the code you weren't aware of.

The Rust community **has a popular concept of "fearless X"**. Fearless concurrency, fearless refactoring, etc.

Ideally, whenever you make a change to the codebase, you want to be confident about which part of the code you need to change, and you want to be confident your change is not going to break the rest of the program.

You want that not only because adding a regression sucks, but because *being confident makes you work faster*. Double-checking the code you write takes time and energy. Having an unclear threat model and jumping at shadows saps your motivation.

Work on a large-scale Qt project was the opposite of fearless. Even with IDE tooling, I had a very loose understanding of what each function was responsible for and where its possible callers were. This was in part because of Qt's signals-and-slots architecture and in part because the codebase's overuse of OOP made the call graph look like a plate of spaghetti.

If something didn't work as planned, figuring out why was a struggle. I had to build a model in my head of the code I was running; based on that model, make an educated guess at what was the thing that was going wrong; and finally, make a very small code change and wait 30 seconds for the project to rebuild so I could test that change.

There was information (eg the values of some variables) known to the computer but not to me, and to surface that information I needed to recompile the project with printf calls / restart in Visual Studio's debugger.

## A problem already solved

These are all problems I had had before.

But by 2019, I had already worked with JS frameworks and learned to use browser devtools and such, so I was all the more frustrated that I *knew* it was possible to do better.

*I shouldn't have to spend 20 minutes grepping through the code trying to guess which class displays this widget! The framework **knows** what widget this is, it's printing it on my screen! I should be able to just select the widget by mouse and get at the very least its name, and ideally the stack trace of the call that created it and a bunch of other info.*

This is a kind of very deep frustration that I often come back to. The computer **knows** what I'm trying to find out. I shouldn't have to do detective work because it hasn't been programmed to tell me! Not being able to surface that information feels like having a medical device run a 20.000$ diagnostic on a patient, then immediately throw the result away because the doctor entered her email address incorrectly. The problem isn't a lack of capability or information, the problem is a lack of versatility in the way we access that information.

Browser devtools aren't perfect, but they add an amazing amount of versatility. The DOM inspector can tell us for any given onscreen pixel which DOM node produces this pixel, what CSS styles are applied to the node, what the node looks like if we add or remove styles, etc. The JS console (in both Chrome and Firefox) is the most amazing tool for visualizing program data I've ever seen.

![Firefox devtools screenshot](/assets/firefox_devtools.png){:class="img-responsive"}

I don't really see the current Rust frameworks helping with the frustration I mentioned. SlintUI, Iced, Druid, egui, etc, all have some great goals: simplicity, responsiveness, good performance, low overhead, accessibility and internationalization support, etc. But none of them support, say, writing unit tests for widgets.

(That's not quite true; Druid has the Harness type, and SlintUI has a nice-looking test harness which I think is internal-only. At the very least, none of them *advertise* supporting unit tests.)

None of them seem to be driven by that same intuition of "Debugging a GUI app shouldn't be painful" that I've had for years.

That's not to say I'm a better developer than Raph Levien or the authors of all these libraries. From what I've seen, the frameworks I've mentioned are high-quality and I strongly approve of the goals they're striving for.

But I do notice that none of them seem to have quite the same vision I have. So, allow me to try to share that vision, and then show what I've been doing to work towards it.


## My vision: Fearless GUI

Rust GUI should be fearless, for the reasons I've described. And fearless GUI is GUI that has other virtues:

- Iterative
- Fixable
- Testable
- Inspectable
- Replayable

We're a long way away from *everything* I'm about to describe. This is a vision document, not a roadmap for 2023. That said, I do think everything I mention here is technically achievable with the manpower the Rust community can muster.


### Iterative GUI

![Iterative GUI](/assets/masonry_goals_iterative.svg){:class="img-responsive"}

This is the most obvious part, but GUI development should have short iteration cycles.

You shouldn't spend hours thinking about what you're going to do next. You should have your hands in the interface mud, making small changes and constantly getting visual feedback that slowly guides you toward your desired goal.

That means build times should be short. How short? Well, the gold standard here is seamless hot reloading: a rebuild so fast it happens interactively, and changing the code is just another way of interacting with the app. Anyone who has worked with a framework with fast hot reloading can attest: working with those is an absolute joy, one it's hard to live without once you know of it, on par with the difference between writing Rust and *writing friggin assembler*.

I don't know how we can achieve this ideal. In this case, our bottleneck is the rest of the ecosystem: we need a faster compiler, better incremental compilation, incremental linking, etc. I do hope that Rust supports hot-reloading as a first-class feature one day.

We might even need to go beyond Rust for this: we might want to use scripting languages like JS or Python or some completely new Rust-centered scripting language. SlintUI seems ahead of the curve here, with its GUI description quasi-language.

That said, short build times aren't the only thing that makes for an iterative process. Good information is important, hence the next sections.


### Fixable GUI

![Fixable GUI](/assets/masonry_goals_fixable.svg){:class="img-responsive"}

Imagine the situation: you're debugging your app. There's a bug in the menu, where none of the buttons are displayed. Your Git-fu is strong enough that you know to immediately bisect the repository, and after 10 minutes of rebuilding and testing, you find the offending commit, which refactors the layout algorithm. Unfortunately it's a pretty large commit, so you still don't know where the bug comes from, and you have to debug it manually.

First you check that the paint method of your button is called. It is indeed, these buttons do exist. You know that the paint method has an early exit in case the button is offscreen. You add a println/breakpoint after that early exit and confirm that it's not systematically taken. You check the transparency of your widgets. Maybe you accidentally set the alpha to zero? Nope, they're fully visible. You print out the positions of your widget on paint. Because paint is called 60 times per second, your terminal is quickly filled with identical log messages, but you confirm that the positions make sense.

Then you remember that the commit changed the layout calculations, by adding the origin of the parent widget to each child widget. Of course! The widget positions are wrong when the offset is added! You check the offset positions and, nope, you get sensible results that don't explain the bug.

So you check again that the paint method is called (it is) and you check that the transparency parameter isn't overridden somewhere (it isn't).

After two more hours of banging your head against the keyboard, you realized that a faulty calculation in your list gave all the widgets a size of NaN which means your painter tried to paint NaN-sized rects and Nan-sized text and predictably failed. No, the painter primitives don't emit a warning when you send them NaN values, why do you ask?

This fictional example illustrates a general idea: **in computer graphics, there are a thousand ways to not display what you want**, and if you bump into one, you get very little information to eliminate the other nine-hundred-ninety-nine.

Fixable GUI is GUI that doesn't suffer from this problem. It's *pit of success* GUI.

It's GUI that performs defensive programming (in debug mode) to alert you if you set invalid values at any stage, and tells you how to correct your mistakes.

It's GUI that gives you the primitives you need to quickly diagnose *what your code is doing*. A trivial way to short-circuit my fictional example above would be a "dump trace to tmp file" button that would re-render the entire UI and write a `tracing` log of every call. Then you would only have to grep it for `MyButtonWidget::paint`, and you would quickly see that the paint method is indeed called, but is given NaN values.

But more than that, Fixable GUI is GUI that gives you *affordances* to fix your problems. Knowing in hindsight what set of tools would have solved the problem is easy, guessing in advance is hard. Maybe you're thinking "*Of course* you should use `tracing`", but was that your first suggestion when reading the example? Do you have tracing in your own app?

Fixable GUI is GUI that gives you the tools to fix your problems *even if you don't know what the tools are*.

That means warnings when the framework detects probable mistakes, and powerful introspection abilities that help you even when you're groping blindly. For instance, a Browser-DevTools-style widget hierarchy inspector could let you confirm that the buttons are indeed present, show where they're displayed, and show that their size is `(NaN, NaN)`.


### Testable GUI

![Testable GUI](/assets/masonry_goals_testable.svg){:class="img-responsive"}

When I was working on that big Qt project, we were lucky enough to have a suite of integration tests.

We had a special test runner that would read config files in a special DSL telling them "click on this button; now confirm this text appears; now click on this box and type 'Hello' and enter", etc. The runner would take control of your mouse and keyboard input, so you couldn't do anything with that machine until the tests were done. We couldn't run them locally or on a server, so one engineer had to download the master commit every day on a dedicated machine, run the tests there, then upload the results. Sometimes we'd get a report along the line of "Test X is broken today. This is probably because of commit Y that Bob pushed yesterday".

This was, as far as I'm aware, the state of the art in testing Qt applications.

(It's hard to be confident about this kind of statement. In this case though, the company had hired engineers from the Qt org for a few weeks of consulting, so if there was a more efficient way to do this, I assume the team would at least be aware of it, which I know they weren't. Also, I still occasionally read Qt docs and I've still found nothing on writing unit tests for widgets.)

Anyway, Qt is far from alone in this. GTK doesn't support testing either; most Rust GUI frameworks don't; only the browser ecosystem has some support for headless testing, thanks to NPM packages.

And that's bad! It's bad that nobody except JS frameworks supports testing!

People have a perception that testing is less important in GUI code and that your tests should be in business logic, but the truth is that UI code can break just as much as business code. UI tests aren't less important, they're just harder to write.

Testing is essential if you want Fearless GUI, especially at scale. Testing lets you make sure the changes you added didn't create a regression in some unrelated part of the code. It helps you *not worry* about breaking things, which helps you write faster.

**Rust GUI should expand developers' notions of how GUIs can be tested.** At the very minimum, Testable GUI includes headless widget rendering, to cover low-level widget code. It includes both "structural tests" (such as "this list has the expected number of rows") to check for logic regressions and "screenshot tests" (rendering the UI to a buffer and saving it to a version-controlled snapshot) to check for visual regressions.

Testable GUI includes fuzzing, to check if there is any combination of user inputs that can crash the app. GUI developers traditionally rely on QA testers to find these crashes, and on user reports after that. That's insane! Automated fuzzing can find bugs faster, more reliably, and for cheaper than humans can. And you can run the fuzzer after every commit, not just once at the end of the production process.

We could even imagine more advanced use cases for automated testing, such as detecting if the app renders correctly for all internationalization configs and accessibility settings (to flag cases where, say, part of a label is no longer readable past a certain zoom level because it overflows its container) or checking that a published tutorial still matches the app's UI. Testing isn't just a set of features, it's a mindset that expands as your tools become more functional, their results more reproducible.

Last but not least, Testable GUI has to enable benchmarking and performance testing. GUI frameworks should provide the same kind of metadata as a game engine does out of the box: average FPS, number of dropped frames, 99% latency, etc. Benchmarks must test a variety of cases (very large lists, very deep widget trees, huge assets, etc) and check all of the metrics I cited, not just the FPS average.

Testable GUI is GUI that is proactive and even *aggressive* in looking for performance regression scenarios and doesn't wait for users and application developers to find them.


### Inspectable GUI

![Inspectable GUI](/assets/masonry_goals_inspectable.svg){:class="img-responsive"}

A GUI application isn't just a random bunch of meaningless bytes that could have any values. The application's data is *structured*, and Inspectable GUI is GUI that surfaces that structure.

The state of the art for inspectability is the browser (aka Firefox and Chromium). And I don't just mean browser devtools, I mean browsers in general. They give you *so many* affordances:

- Text search.
- Screen reader support.
- Firefox reader mode, which takes the page's main content and presents it in a readable form without the cruft.
- Going back to the page you were on before.
- Reloading the current page.
- Selecting and copying any text on the page.

Sadly, JS frameworks do everything they can to negate all these advantages and give you their inferior poorly-integrated alternative, but still, the browser gives both users and developers *a lot* of power out of the box, something GUI frameworks should strive to emulate.

Devtools give even more power to developers; I've already mentioned the Inspector and the Console, but there are also Network tools to inspect packets, options to emulate a phone environment or a bad connection, profilers, debuggers, etc...

I think the **minimum** the Rust ecosystem should do for GUI is to implement all these state of the art affordances.

Like, that would be okay. It would be a B minus.

An A plus would be implementing [Bret Victor's learnable programming](http://worrydream.com/LearnableProgramming/).

![Screenshot from learnable programming](/assets/learnable_programming.jpg){:class="img-responsive"}

I want to be able to log rectangle values in my UI code and have the UI highlight the place that the rectangle represents so I can visualize it. I want all data structures / widgets / layout concepts like Flex to have visual aids that are displayed when you select them. I want to dump the state of my widget tree, open it in the console, and inspect that widget tree as if it were my current page, search strings in it, etc.

I want an equivalent to [Pernosco](https://pernos.co/) for GUI, where I can click any pixel on the screen and get a breadcrumbs trail I can follow to figure out exactly which part of the code is responsible for that pixel having the value it has.

This is an area where I think Rust has the potential to be truly groundbreaking. What I'd like to see is a set of tools that improve the developer experience so much that they push back our *perception* of what developer tools can help us do.


### Replayable GUI

![Replayable GUI](/assets/masonry_goals_replayable.svg){:class="img-responsive"}

You know that thing where Overwatch developers needed a killcam, so [they built a replay system into their ECS framework](https://www.youtube.com/watch?v=W4oZq4tn57w), which they could then use for game highlights and esport broadcast and debugging?

> (5:30) We have a hotkey internally that dumps the last 30 seconds of replay data to disk. \[That data\] helped us really isolate and fix a bevy of bugs that would have been much more difficult to recreate without a replay. Beyond that, replays have provided us a great way to repeatedly test performance over time: developers can tweak code, measure the performance impact and iterate without having to start a playtest.

Yeah, I want the same thing for GUI.

Replayable GUI has a strong separation between deterministic and platform-dependent code. It can produce screenshots and videos and perform record-replay by serializing user inputs and other syscall results to a file on disk.

**This replay feature should be a first-class citizen**, not a bonus that you learn about in some advanced tutorial. Loading a replay should be easy and Just Work. Recording a replay should be a single keyboard shortcut away in every application.

You should be able to send replays over email and on file-sharing services to people who don't share your build. There should be tools to convert your replay to a video.

When running a replay, you should have access to all your framework's devtools: you should be able to run your debugger on the code (both forward and backward, of course), inspect widgets, print values, etc.

Replays should be integrated with tests. The framework's testing harness should have a method to dump a replay, so that if your headless test fails, you can get a visual representation of what happened and understand where it went wrong.

And replays should be extremely memory-efficient so that developers can enable recording by default in their apps. If the app crashes, the developer shouldn't think "I need to trigger this condition again while recording", they should think "Oh, the crash message is saying that a record was auto-saved to this temporary file, I can load it now and figure out what happened".


### This is going to take a while

I am absolutely not saying any of this is easy to do.

What I'm trying to describe is an *ideal*, something to navigate towards. Again, I don't think existing Rust GUI frameworks are bad. It's just that none of them seem to be seeing what I'm seeing, none of them quite seem to be seeing this immense gap that the Rust ecosystem should be rushing to fill.

With all that said, here's my very small, tentative attempt to start filling that gap.


## Introducing Masonry 0.1

[Masonry](https://crates.io/crates/masonry) is a framework that aims to provide the foundation for Rust GUI libraries.

It started out as a fork of Druid that emerged from discussions I had with Raph Levien and Colin Rofls about what it would look like to turn Druid into a foundational library.

To put it simply, Masonry gives you a platform to create windows (using Glazier as a backend) each with a tree of widgets. Each widget has to implement the Widget trait that Masonry provides.

Other than that, the framework is not opinionated about what your user-facing abstraction will be: you can implement immediate-mode GUI, the Elm architecture, functional reactive GUI, etc, on top of Masonry.

The todo-list example of Masonry looks like this:

```rust
use masonry::widget::{
    Button, CrossAxisAlignment, Flex, Label, Portal, SizedBox, TextBox, WidgetMut,
};
use masonry::{
    Action, AppDelegate, AppLauncher, Color, DelegateCtx, Env, WidgetId, WindowDescription,
    WindowId,
};

struct Delegate {
    next_task: String,
}

impl AppDelegate for Delegate {
    fn on_action(
        &mut self,
        ctx: &mut DelegateCtx,
        action: Action,
    ) {
        match action {
            Action::ButtonPressed | Action::TextEntered(_) => {
                let mut root: WidgetMut<Portal<Flex>> = ctx.get_root();
                if self.next_task != "" {
                    let mut flex = root.child_mut();
                    flex.add_child(Label::new(self.next_task.clone()));
                }
            }
            Action::TextChanged(new_text) => {
                self.next_task = new_text.clone();
            }
            _ => {}
        }
    }
}

fn main() {
    // The main button and text box with some space below,
    // all inside a scrollable area.
    let root_widget = Portal::new(
        Flex::column()
            .with_child(
                Flex::row()
                    .with_child(TextBox::new(""))
                    .with_child(Button::new("Add task")),
            )
            .with_spacer(VERTICAL_WIDGET_SPACING),
    )
    .constrain_horizontal(true);

    let main_window = WindowDescription::new(root_widget)
        .title("To-do list")
        .window_size((400.0, 400.0));

    AppLauncher::with_window(main_window)
        .with_delegate(Delegate {
            next_task: String::new(),
        })
        .launch()
        .expect("Failed to launch application");
}
```

Note that, compared to crates like Druid or Iced, Masonry takes a fairly low-level approach to GUI: there is no complex reconciliation logic or dataflow going on behind the scenes. When the example above wants to add a child to the flex container, it calls `flex.add_child(some_child)`.

This simplicity makes Masonry somewhat painful if you want to use it to build actual GUI applications. The hope is that, by being low-level and straightforward, developers can easily build GUI frameworks on top of it.

(Well, in theory. The first stress-test will be porting [Panoramix](https://github.com/PoignardAzur/panoramix), a React-style GUI crate, to Masonry.)


### The Widget trait

Masonry has a built-in library of widgets with the basic ones: buttons, checkboxes, flex containers, etc.

If you want to add your own, you have to implement the Widget trait:

```rust
impl Widget for HelloButton {
    fn on_event(&mut self, ctx: &mut EventCtx, event: &Event, _env: &Env) {
        match event {
            Event::MouseDown(_) => {
                println!("Hello world!");
            }
            _ => (),
        }
    }

    fn layout(&mut self, ctx: &mut LayoutCtx, bc: &BoxConstraints, env: &Env) -> Size {
        const DEFAULT_BUTTON_SIZE: Size = Size::new(50, 30);

        let actual_size = bc.constrain(DEFAULT_BUTTON_SIZE);
        actual_size
    }

    // [...] - Some methods not shown here

    fn paint(&mut self, ctx: &mut PaintCtx, env: &Env) {
        let stroke_width = 1.0;
        let rounded_rect = ctx.size().to_rect().to_rounded_rect(4.0);

        ctx.stroke(rounded_rect, &COLOR::WHITE, stroke_width);
        ctx.fill(rounded_rect, &Color::GRAY);
    }

    fn children(&self) -> Vec<WidgetRef<'_, dyn Widget>> {
        Vec::new()
    }
}
```

If you're familiar with Druid, you may recognize some methods: Masonry's Widget trait is very close to Druid's. The biggest difference is that container widgets in Masonry must return a list of their children, which the framework uses for reflection and testing.


### Unit tests

Masonry is designed to make unit tests easy to write, as if the test function were a mouse-and-keyboard user. Tests look like this:

```rust
#[test]
fn some_test_with_a_button() {
    let [button_id] = widget_ids();
    let widget = Button::new("Hello").with_id(button_id);

    let mut harness = TestHarness::create(widget);

    // Make a snapshot test of the visual contents of the window
    assert_render_snapshot!(harness, "hello");

    harness.edit_root_widget(|mut button, _| {
        let mut button = button.downcast::<Button>().unwrap();
        button.set_text("World");
    });

    // Make new snapshot test now that the window has changed
    assert_render_snapshot!(harness, "world");

    // References to widget automatically implement Debug, and
    // will print their part of the widget hierarchy.
    println!("Window contents: {:?}", harness.root_widget());

    // You can also use insta to snapshot-test the widget hierarchy
    assert_debug_snapshot!(harness.root_widget());

    // Clicking on a button will produce a "ButtonPressed" action.
    harness.mouse_click_on(button_id);
    assert_eq!(
        harness.pop_action(),
        Some((Action::ButtonPressed, button_id))
    );
}
```

The test harness still needs some improvements, but it's by far the most polished part of Masonry. In my personal experience, writing unit tests for Masonry is a breeze, and it has legitimately helped me catch bugs in my widget implementations.


### Devtools

Earlier in this article, I said that a Rust GUI framework should have all the affordances of a browser.

Masonry isn't even remotely at that point.

I initially wanted to include a rudimentary Widget Inspector as a proof of concept, which would let you click an arbitrary widget and get its position in the widget tree, styling information, other metadata, etc. But implementing that inspector took longer than I expected, and by that point, I had already delayed the first release way too much (see next section), so I decided to release Masonry without it.

This speaks to the limitations of Masonry as a GUI framework. While the information I need to display in the editor (the widget hierarchy and metadata) is readily accessible, actually making a UI to display it is still the difficult part. Masonry just isn't a good library to make GUIs with, partly by design (the goal is to be a backend, not a user-facing tool), and partly because of accidental limitations. As the framework evolves, these limitations will need to get addressed.


## Next steps?

At the beginning of this post, I've laid out my vision for what a GUI framework should be: iterative, fixable, testable, inspectable, and replayable.

Well, so far Masonry aims for "Fixable" and "Testable".

![Masonry currently achieves parts of "Fixable" and "Testable"](/assets/masonry_goals_currently.svg){:class="img-responsive"}

Masonry is very much in its 0.1 version. A lot of internal abstractions are subject to change, and you can take a peek at [the issue backlog](https://github.com/PoignardAzur/masonry-rs/issues) to get an idea of how many aspects need a refactor.

That said, I think the core idea is solid. I definitely wouldn't use Masonry for a project that needs to hit production this year, but in a year or two, that might become a reasonable choice.

### Integration

One of the initial priorities will be to port my crate Panoramix to use Masonry as a backend (it currently uses Druid). I hope to validate that the abstractions I built are legitimately useful for framework designers and do make development faster and easier. So far, my experience using Masonry to write tests and built-in widgets left me optimistic, but now I need to sit down and dogfood the project for real.

The project is also getting to a stage where I would like to get buy-in from other Rust GUI maintainers: I want to persuade GUI developers to try to port their framework. To make that an attractive proposition, I need to develop the "Devtools" side of Masonry, so that developers get strong benefits from using it as a platform.

The first candidate besides Panoramix would be Xilem. I've already had some discussions with both Raph Levien and the broader Xilem community, who have given wary but overall positive feedback to the idea.

There is a lot of work I need to do before the Xilem community will seriously consider that port. A big part of that work will be [switching Masonry's dependencies](https://github.com/PoignardAzur/masonry-rs/issues/24) to the more up-to-date crates that Xilem uses. Other things will need to be refactored and polished.

### Call for help

I'm going to be candid here: I may be in over my head with this project.

Masonry has been one of these "Zenon's paradox" types of projects, where every time you think you're getting closer to your goal, the remaining tasks become bigger and split into sub-tasks. Finish one of the sub-tasks, the others start growing and splitting as well.

I want to encourage any developers who've read this far and feel enthusiastic about the project to contribute. Masonry is only in its infancy. To be perfectly clear, it's even kind of awful, a downgrade from Druid in some aspects (not all, mind you!), and it will need more man-hours than a single person can provide to achieve the goals I've grouped under the "Fearless GUI" umbrella.

And I *do* want to achieve these goals. Despite how big the task is, it doesn't feel unfeasible. There's no reason a GUI framework would be an impossible task, when the Rust community has managed to produce compilers, OSes, game engines, and ports of the entire coreutils collection. I think Masonry is the first step towards a goal that is both achievable and a step forward for the entire ecosystem.

The Rust community feels like it's in a pretty good position to advance the state of the art! We have both industry veterans and idealistic newcomers, all extremely passionate, talented, ambitious enough to want to create something from the ground up, and humble enough to tread lightly and proactively learn about the mistakes of their predecessors.

So I do think Rust is going to revolutionize native GUI one day, and I'd like to think Masonry will be a step towards that.

In the meantime, I've [marked issues](https://github.com/PoignardAzur/masonry-rs/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22) where I expect a single contributor could make a difference in their free time. I'm really hoping that people take interest in the project and that it picks up more speed than it would as a one-man side-project.

Even if you're not sure how to contribute, if you're feeling enthusiastic about the vision I've shared, please come to the [Xilem zulip](https://xi.zulipchat.com/#narrow/stream/317477-masonry) to talk to people and exchange ideas.

Any contribution will be appreciated! I can't stress this enough: at Masonry's current scale, **even a single person popping by to say that they like the project is a huge win**. Issues, pull requests and suggestions help even more.

Thank you for reading this far, and ~~happy holidays~~ have a good month of February!

[Discussion on r/rust.](https://www.reddit.com/r/rust/comments/10uhw4l/announcing_masonry_01_and_my_vision_for_rust_ui/)