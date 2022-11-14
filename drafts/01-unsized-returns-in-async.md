---
layout: post
title: Async traits would benefit from unsized returns.
---

https://rust-lang.zulipchat.com/#narrow/stream/213817-t-lang/topic/RPIT.20in.20dyn.20Traits.20-.20Unsized.20returns
https://rust-lang.zulipchat.com/#narrow/stream/213817-t-lang/topic/return.20position.20impl.20Trait.20in.20dyn.20Trait/near/276384740

This is an analysis of a new feature that could be added to [the Rust programming language](https://www.rust-lang.org/). The analysis assumes that you're at least familiar with Rust and with async functions.

Most of this post is a summary of a [conversation I had on Zulip](https://rust-lang.zulipchat.com/#narrow/stream/213817-t-lang/topic/RPIT.20in.20dyn.20Traits.20-.20Unsized.20returns) with lang team members earlier this year.


## The design problem: async trait methods.

In Rust, async functions are lowered to functions returning existential traits. Eg this:

```rust
async fn get_value() -> i32 {
    // ...
}
```

is lowered to:

```rust
fn get_value() -> impl Future<Output = i32> {
    // ...
}
```

Which roughly means "figure out what type this functions returns, as long as it implements the Future trait". The async machinery does some more wizardry inside the function body to create a state machine based on `.await` instructions, and an implementation of the Future trait for that state machine.

Ideally, we'd like to be able to use the same syntax inside of traits, and functions using those traits:

```rust
trait AsyncIterator {
    type Item;
    async fn next(&mut self) -> Option<Self::Item>;
}

async fn count(iter: &mut impl AsyncIterator<Item = i32>) -> usize {
    let mut c = 0;
    while let Some(_) = iter.next().await {
        c += 1;
    }
    c
}
```

Even more ideally, we'd want to be able to use async methods in dyn traits:

```rust
async fn count(iter: &mut dyn AsyncIterator<Item = i32>) -> usize {
    let mut c = 0;
    while let Some(_) = iter.next().await {
        c += 1;
    }
    c
}
```

Being able to use async in traits is a common ask. The fact that using async cuts off an entire section of Rust's features is a complaint that's often mentioned when people discuss whether Rust is a pleasant language to get into.

So, ideally, we'd really want to be able to just slap async onto trait methods and have it be as convenient to use as any other method as long as you `.await` it.

### First obstacle: RPITIT

"RPIT" stands for "Return-Position Impl Trait", the syntax for returning existential types. "RPITIT" is a barbaric acronym for "Return-Position Impl Trait In Trait", or in other words, writing `fn foobar() -> impl MyTrait` in a trait method.

It's a necessary feature to implement async traits. For instance, the trait described earlier:

```rust
trait AsyncIterator {
    type Item;
    async fn next(&mut self) -> Option<Self::Item>;
}

impl AsyncIterator for MyIter {
    type Item = u32;
    
    async fn next(&mut self) -> Option<Self::Item> {
        // ...
    }
}
```

would be lowered to this syntax:

```rust
trait AsyncIterator {
    type Item;

    fn next(&mut self) -> impl Future<Output = Self::Item>;
}

impl AsyncIterator for MyAsyncIter {
    type Item = u32;
    
    fn next(&mut self) -> impl Future<Option<Self::Item>> {
        // ...
    }
}
```

This syntax in turn would need to be lowered somehow to make sense of the `-> impl Future` in the trait method.

Long story short, the plan is to lower it to something like this:

```rust
trait AsyncIterator {
    type Item;

    type Next__<'me>: Future<Output = Self::Item> + 'me;
    fn next(&mut self) -> Self::Next__<'_>;
}

// Type alias impl trait:
type MyAsyncIter_Next__<'me> = impl Future<Output = u32> + 'me;

impl AsyncIterator for MyAsyncIter {
    type Item = u32;
    
    type Next__<'me>: Future<Output = Self::Item> + 'me;
    fn next(&mut self) -> Self::Next__<'_> {
        // ...
    }
}
```

This would require two currently unstable features: Generic Associated Types to express correct lifetimes, and Type-Alias Impl Trait to tell the compiler "infer the type returned by next, and give it a name we can refer to in our trait implementation".

Fortunately, both of these features are in the process of being stabilized (GATs have been stabilized on the main branch already), so this isn't a major blocker.


### Second obstacle: RPIT in dyn traits

Consider our example async trait desugaring again:

```rust
trait AsyncIterator {
    type Item;

    type Next__<'me>: Future<Output = Self::Item> + 'me;
    fn next(&mut self) -> Self::Next__<'_>;
}
```

Is this trait dyn-safe?

It doesn't seem so. Associated types need to be specified in dyn traits, so to use the trait above we would need to write:

```rust
async fn count(iter: &mut dyn AsyncIterator<Item = i32, for<'a> Next__<'a> = ???>) -> usize {
    let mut c = 0;
    while let Some(_) = iter.next().await {
        c += 1;
    }
    c
}
```

... but the whole point of dyn traits is that they will have different implementations, so you can't statically know which state machine iter will use under the hood.

What we *would* like is to use `&mut dyn AsyncIterator<Item = i32>` and store the future return by `iter.next()` is a dynamic type.

So, in a dyn trait,

```rust
fn next(&mut self) -> impl Future<Output = Self::Item>;
```

would become:

```rust
fn next(&mut self) -> dyn Future<Output = Self::Item>;
```

And here, we come to the crux of the problem: `dyn Future<...>` doesn't have a statically-known size. So how can our `count()` function allocate enough function to store the result of `iter.next()`, when it doesn't know the size of that result at compile-time?





What we want is simple. Imagine this trait, for “async iterators”:

We would like you to be able to write a trait like that, and to implement it in the obvious way:

struct SleepyRange {
    start: u32,
    stop: u32,
}

impl AsyncIter for SleepyRange {
    type Item = u32;
    
    async fn next(&mut self) -> Option<Self::Item> {
        tokio::sleep(1000).await; // just to await something :)
        let s = self.start;
        if s < self.stop {
            self.start = s + 1;
            Some(s)
        } else {
            None
        }
    }
}

You should then be able to have a Box<dyn AsyncIter<Item = u32>> and use that in exactly the way you would use a Box<dyn Iterator<Item = u32>> (but with an await after each call to next, of course):

let b: Box<dyn AsyncIter<Item = u32>> = ...;
let i = b.next().await;





20:35

I'd like to make a separate thread to make my case for using RFC #2884, aka unsized returns, for RPIT-in-dyn-trait and by extension dyn async traits.

Niko's rationale for the current RPIT-in-dyn-trait design is to enable library writers to write:

```rust
async fn count(iter: &mut dyn AsyncIterator) -> usize {
    let mut c = 0;
    while let Some(_) = iter.next().await {
        c += 1;
    }
    c
}
```

and have that code be used in no_std envs.

The big constraints are:

    Library writers must write their code "naturally"; they will write code as generic as possible, and not worry about implementation details.
    Embedded developers must be able to use libraries without allocating anything. They may not even have an allocator.
    These two constraints must be compatible. Eg embedded developers shouldn't have to ask library writers to add an allocator-less version.

To satisfy those, the design adds a new "adapter" concept. Adapter are like magic HKTs that can take a trait MyTrait, and produce a wrapper type MyTraitWrapper that, for every RPIT method of MyTrait, implements a wrapper method with basically the RPIT's vtable, plus a custom destructor. The main use-case of that adapter concept is the #[dyner::inline_adapter] attribute, which, applied to AsyncIterator, gives us InlineAsyncIterator, which is an iterator that stores the return value of the next() call in a preallocated slot.

Olivier FAURE:

Josh's big objection to this is that this pattern is basically encouraging lots of hidden allocations that aren't clear when looking at either the call site or the callee code. Eg:

```rust
impl AsyncIterator for MyRange {
    type Item = i32;
    async fn next(&mut self) -> Option<Self::Item> {
        sleep(30).await;
        self.range.next()
    }
}

async fn do_thing(x: &mut dyn AsyncIterator) -> usize {
    let mut c = 0;
    while let Some(_) = iter.next().await {
        c += 1;
    }
    c
}

fn do_thing_with_my_range() {
    do_thing(&mut MyRange(0..10));
}
```

Looking at the code of all three functions, there's no sign that any of the code allocates anywhere, even though it creates boxes for every call to .next().

Even if we edited do_thing_with_my_range to add an explicit boxing adapter, .next() is the call that allocates, and yet there's no indication at either the call site or in next()'s implementation that memory is allocated.

Josh points out that this is bad, because allocator calls are non-trivial things, and developers may want strong guarantees that they can control where the allocator is called. Allocations can panic, can be expensive cache-wise, etc. Some people get really nervous when the compiler starts adding hidden allocations.
Olivier FAURE:

My own objection to the proposed design is that the whole "adapter" machinery seems very complex, and yet it only covers very specific use-cases.

The "inline adapter" only works for &mut self methods that return a type with a borrow on self. This is the case for AsyncIterator::next(), but it may not be the case for other trait methods, eg:

```rust
trait MyTrait {
    fn get_printable(&self, value: i32) -> impl Display;
}

fn call_three_times(obj: &dyn MyTrait) {
    let a = obj.get_printable(0);
    let b = obj.get_printable(1);
    let c = obj.get_printable(2);
    println!("{a} {b} {c}");
}
```

My understanding is that the implemented workaround is using a cell, and the second call to get_printable panics, but... let's be honest, this is a horrifying hack. If we're not fine with inserting silent allocations, I can't imagine we're fine with inserting random panics because the allocation strategy of your caller didn't like that you called the same &self method twice.
Olivier FAURE:

Instead, I propose that we ask developers who want their async code to be maximally portable to write

```rust
fn count(iter: &mut impl AsyncIterator) -> usize {
    // ...
}
```

We can add InlineAsyncIterator as a special type, implemented with existing concepts (and some unsafe code), and skip most of the dynx / adapter machinery.

Embedded developers who want no allocation and to avoid generic bloat can then write:

```rust
fn invoke_count(mut x: impl AsyncIterator) -> usize {
    count(&mut InlineDynAsyncIterator::new(x))
}
```

For non-async RPIT methods, we can use the syntax in RFC 2884:

```rust
trait MyTrait {
    fn get_printable(&self, value: i32) -> impl Display;
}

fn call_three_times(obj: &dyn MyTrait) {
    let a = Box::new_with(|| obj.get_printable(0));
    let b = Box::new_with(|| obj.get_printable(1));
    let c = Box::new_with(|| obj.get_printable(2));
    println!("{a} {b} {c}");
}
```

Doing it that way, the main advantage is we can use different allocation strategies, and it's obvious which one we're using:

```rust
fn call_three_times(obj: &dyn MyTrait) {
    let a = SmallBox::new_with(|| obj.get_printable(0));
    let b = SmallBox::new_with(|| obj.get_printable(1));
    let c = SmallBox::new_with(|| obj.get_printable(2));
    println!("{a} {b} {c}");
}
```

```rust
fn call_three_times(obj: &dyn MyTrait) {
    let mut arena = MyGrowingArena::new(SOME_VALUE);
    let a = arena.new_with(|| obj.get_printable(0));
    let b = arena.new_with(|| obj.get_printable(1));
    let c = arena.new_with(|| obj.get_printable(2));
    println!("{a} {b} {c}");
}
```



Olivier FAURE: Josh Triplett said:

    Right, count is easier. What should call_three_times do to let its caller decide how to handle the return values?

It can do this:

```rust
fn call_three_times(obj: &impl MyTrait) {
    let a = obj.get_printable(0);
    let b = obj.get_printable(1);
    let c = obj.get_printable(2);
    println!("{a} {b} {c}");
}
```


Olivier FAURE: Eeeh... kinda. Right now the language conflates "is dyn" with "is Sized".

Josh Triplett: It does seem like it'd solve the problem, since then the caller can decide how to turn a dyn into an impl while handling the return values.

Josh Triplett: (Though it doesn't give many options to the caller in doing so.)

Olivier FAURE: I mean... that makes sense? In non-async, non-RPIT code you usually don't worry about how the functions you're calling allocate memory for their internals.

Olivier FAURE: Like, if you're calling find_path_with_astar(my_graph), you usually don't expect to control whether the A* implementation is using Vecs or VecDequeues.

Olivier FAURE: The broader point I'm making is that, given this trait declaration:

```rust
trait MyTrait {
    fn get_printable(&self, value: i32) -> impl Display;
}
```

there isn't any feasible way to pass &dyn MyTrait to a function, and have that function use it, while guaranteeing that no dynamic allocation will ever be made.

Olivier FAURE: You can make it so that there are few allocations, which is often good enough in embedded. You can use an arena. You can force the code that's making the allocation to use Box::new_with or whatever to make it clear that allocations are happening.

Olivier FAURE: But the fn method() -> impl Display; API and the fn call_three_times(obj: &dyn MyTrait) API, taken together, make it so that the program has no way to determine statically how much memory will be needed ahead of time.