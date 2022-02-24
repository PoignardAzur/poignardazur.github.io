---
layout: post
title: Analyzing unsized variables in Rust
---

In [the Rust programming language](https://www.rust-lang.org/), unsized variables are one of those concepts that has been floating around for a long time, but hasn't seen much design work in recent years.

Although there is [an RFC dating back to 2017](https://github.com/rust-lang/rfcs/pull/1909) (5 years old as of the writing of this article), and there's a stub implementation in the nightly compiler, the feature looks stalled.

Little work has been done on it in recent years, the `unsized_locals` feature is [marked as incomplete](https://github.com/rust-lang/rust/blob/f19adc7acc649ad2b18b6e172b683ebadb3d8a92/compiler/rustc_feature/src/active.rs#L525-L527), and as far I can tell it doesn't look like much progress has been done in resolving its fundamental design problems.

Still, I think there's some room for quick improvement, especially with the `unsized_fn_params` feature.

This article is an analysis of unsized variables, touching both past design work and how things could evolve in the near future. While I'm not familiar with compiler internals, I'll still try to focus on things that should be technically feasible to implement.


## What are unsized variables?

"Unsized variables" is a catch-all term I'm using to point to the use of unsized types in locals, function parameters, and function returns. Rust maintainers will usually refer to "unsized locals" and "unsized params" instead, but there's not a lot of formal documentation on the subject.

What are unsized types, you may ask?

In Rust, the type system draws a fundamental distinction between regular types and [unsized or dynamically-sized types](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait). To over-simplify, unsized types are types whose size isn't known at compile time.

For instance:

```rust
fn my_function(n: usize) {
    let array = [123; n];
    // do things with_array
}
```

The above code can't compile on a current compiler, because the compiler wants all local variables (locals for short) to have a known size, so that it can decide on the size and layout of the function's stack frame ahead of time.

Currently, in Rust, there are two kinds of unsized types:

- Variable-length arrays, eg `[u8]`
- Trait objects, eg `dyn MyTrait`

(There are actually a few more, but those are the ones we care about)

Unsized types can only be used indirectly, through references and boxes:

```rust
fn foobar_1(thing: &dyn MyThing) {}     // OK
fn foobar_2(thing: Box<dyn MyThing>) {} // OK
fn foobar_3(thing: MyThing) {}          // ERROR!
```

So, unsized types can't be used the same way as sized types. In particular:

- They can't be declared as local variables.
- They can't be passed as parameters to functions.
- They can't be returned from blocks or functions.


### Why should we care?

Unsized variables have one major use case: **passing trait objects to functions and returning trait objects**. Allowing this would enable many useful features:

- Passing `self` by value in trait-safe methods.
- Passing `dyn FnOnce` to functions (currently requires boxing, since `&mut dyn FnOnce` can't be called).
- Returning `impl MyTrait` from trait-safe methods (aka [RPIT-in-traits](https://github.com/rust-lang/impl-trait-initiative/blob/master/RFCs/rpit-in-traits.md)).
- [Trait-safe async methods](http://smallcultfollowing.com/babysteps//blog/2022/01/07/dyn-async-traits-part-7/).

There are some other potential use cases, like saving allocations by passing arrays instead of vecs, but ecosystem crates like smallvec cover these cases well enough.

The rationales relating to object safety, on the other hand, would allow programming patterns that are currently impossible to become trivial, and are a major motivator for unsized variables.


## RFC #1909

[RFC #1909](https://github.com/rust-lang/rfcs/blob/master/text/1909-unsized-rvalues.md) came out in February 2017, and is pretty straightforward. It proposes that:

- (1) All locals can be unsized, including function arguments.
- (2) Return values of functions must be sized.
- (3) The Right-Hand Side of assigments (eg `x = y;`) must be sized.
- (4) Trait methods can take `self` by value and still be object-safe.

The reasoning for rules (2) and (3) is that the compiler needs to known the size of a variable at the site of its declaration. You can't execute arbitrary code, which is itself going to allocate stack space, before allocating the space for your unsized variable.

To understand why, we need to talk about stack allocations.

### Stack model

For the purposes of this discussion, let's imagine a simplified model for the program's stack. When you declare a new variable, the compiler bumps the stack pointer and increases the stack's size to fit the new variable (we ignore alignment here). Then code writes into that memory space and, once the variable's scope is exited, the compiler bumps the stack pointer again to release the stack's memory.

So, a function like this:

```rust
fn my_function() {
    let x: u16 = 42;
    let y: u32 = 10_000;

    // do things with x and y

    // return
}
```

might look like this at various points in time:

![Stack space schema](/assets/unsized_locals_schema_1.drawio.svg){:class="img-responsive"}

In practice, though, the compiler (and more specifically, the LLVM backend) will compute how much space all the locals are going to need at any given point, and allocate the maximum amount of space when entering the function.

So in principle, it would be more accurate to represent the memory like this:

![Stack space schema](/assets/unsized_locals_schema_2.drawio.svg){:class="img-responsive"}

But we'll use the fragmented representation for clarity.

If a function has arguments, the calling function will either copy the arguments to the beginning of our stack frame, or, for big arguments, copy a pointer to argument's address.

So this code:

```rust
fn other_function(arg1: u8, arg2: u8, arg3: SomeBigStruct, arg4: u8) {
    let x: u16 = 42;
    let y: u32 = 10_000;

    // do things with x and y

    // return
}
```

Will produce this stack:

![Stack space schema](/assets/unsized_locals_schema_3.drawio.svg){:class="img-responsive"}

Creating an unsized local is relatively simple: we just bump the stack by a dynamic amount. For instance, this code:

```rust
fn use_unsized_array(n: usize) {
    let array: [u8] = [123; n];
}

use_unsized_array(5);
```

Will produce this stack:

![Stack space schema](/assets/unsized_locals_schema_4.drawio.svg){:class="img-responsive"}

Passing unsized function parameters is even simpler: arguments are just fat pointers to the unsized values. These values can be on the stack, or boxed (though that case requires some extra compiler logic for dropping).

Where things become more complicated is when the declaration and the initialization of an unsized variable are separated. For instance:

```rust
fn bad_unsized_array() {
    let array: [u8];
    let x: u16 = 42;

    if (some_condition()) {
        array = [u8; some_value()];
    }
}
```

![Stack space schema](/assets/unsized_locals_schema_5.drawio.svg){:class="img-responsive"}

Where should the compiler write the value of `x`? It can't put it immediately after `array`, since its size is still unknown when `x` is being written to.

There are some simple answers (for instance, the compiler can store all static variables first, and *then* the dynamic array), but they don't hold when an unsized value is passed across a return call, or copied to another unsized value.

This last case can be, pretty frequent, for instance:

```rust
fn copy_unsized_array() {
    let result_array = {
        let first_array = [0; some_value()];
        let second_array = [sum(first_array); other_value()];
        second_array
    };
}
```

In that code, the compiler must first allocate memory for `first_array`, then memory for `second_array`, then memory for `result_array`. While `second_array` and `result_array` could in principle be merged into one, `first_array` must still be live while `second_array` is being written to.

What that means is that, when the block returns, memory for `first_array` will still be allocated, because `result_array` is above it in the stack. This breaks our nice clean "memory is allocated when variable is declared and freed when variable is out of scope" abstraction.

Again, there are theoretical ways to deal with that.

In practice, [the documentation for RFC #1909's implementation](https://github.com/rust-lang/rust/blob/bb22eaf39e6e0a904a8f19a2a659620c12f03a24/src/doc/unstable-book/src/language-features/unsized-locals.md) `unsized_locals` currently says:

> Another pitfall is repetitive allocation and temporaries. Currently the compiler simply extends the stack frame every time it encounters an unsized assignment. So for example, the code
>
> ```rust
> #![feature(unsized_locals)]
> 
> fn main() {
>     for _ in 0..10 {
>         let x: Box<[i32]> = Box::new([1, 2, 3, 4, 5]);
>         let _x = *x;
>     }
> }
> ```
>
> [...] will unnecessarily extend the stack frame.

What this tells me is that the implementation uses one `alloca` use for every unsized temporary, alloca being the LLVM primitive for allocating dynamic amounts of stack space. Because alloca doesn't have (AFAIK) a granular way to deallocate stack space, that means repeatedly using it is a sure way to get a stack overflow.

(I checked [on the playground](https://play.rust-lang.org/?version=nightly&mode=debug&edition=2021&gist=efa64cefc7fda78fd0232342900b79fb), and, yup, still a stack overflow.)

The focus of the lang team has moved to a subset of RFC #1909: the `unsized_fn_params` feature. That feature allows passing unsized values as function parameters, which doesn't require using alloca.

From discussions on zulip, I'm told that the `unsized_fn_params` feature may be ready to stabilize. At least, while it still needs to be checked for bugs, it doesn't require new design work.


## Unsized returns

One use-case RFC #1909 doesn't cover even hypothetically is unsized returns.

Two years ago (oh, how time flies), I submitted [RFC #2884](https://github.com/rust-lang/rfcs/pull/2884) to cover this case, based on a previous RFC draft from user notriddle.

The RFC was presented with the following rationales:

- Helping performance by providing an equivalent of sorts to [C++'s placement new](https://stackoverflow.com/a/222578/3511753) (hence the name).
- Help writing to uninitialized memory, in particular when reading IO.
- Returning `impl MyTrait` from trait-safe methods, and trait-safe async.

In retrospect, only the third rationale really holds water. LLVM is pretty good at eliding copies and placement is rarely necessary, [other proposals have come out](https://github.com/rust-lang/rfcs/blob/master/text/2930-read-buf.md) that address Reading to unitialized memory, and examples for these use cases muddled the main point.

The core of the RFC is that functions be allowed to return unsized values; these functions would be "split in two": the first part of the function would return the size of the returned value, and the second part would write the value into a provided buffer.

```rust
fn foobar(arg: Arg) -> [u8] {
    [123; some_value(arg)]
}

// WOULD BECOME

fn __foobar_size(arg: Arg) -> usize {
    some_value(arg)
}

fn __foobar_write(buffer: &mut [u8]){
    fill(buffer, 123);
}
```

On the user size, someone wanting to use unsized functions would need to pass them to functions that would box them, eg:

```rust
let my_foobar = Box::new_with(|| foo_bar(some_arg));
```

The idea being that `Box::new_with` would first call `__foobar_size`, then `malloc`, then `__foobar_write`.

In retrospect, the RFC was probably too ambitious given the current state of the compiler. It relies on Guaranteed Copy Ellision (GCE), which isn't yet implemented, and "separating a function in two", which would be feasible with generators but probably come with a ton of follow-up problems.

A good first step, as mentionned at the time, would be to implement GCE in the compiler. Doing so would probably have minor knock-on benefits for performance.

Another step would be to implement unsized returns, but only for trait methods in some situations. For instance, look at this trait:

```rust
trait GetDebuggableValue {
    fn get_debug(&self) -> impl Debug;
}

fn foobar(arg: &dyn GetDebuggableValue) {
    let debug_value = Box::new_with(|| arg.get_debug());
    // do things with debug_value
}
```

From the point of view of `foobar`, `debug_value` is unsized.

But from the point of view of the trait implementation, the size is known statically. In fact, the return size could even be written in the `GetDebuggableValue` vtable. In that case, the `get_debug()` function wouldn't need to be split into with a generator; `Box::new_with` would simply need to do a vtable lookup before calling `get_debug()`. This might be easier to implement in the compiler.

In any case, one interesting point is that RFC #1909 and RFC #2884 can absolutely be implemented in parallel. They might have synergies if both were implemented, but neither should be a blocker on the implementation of the other.


## Shiny future

These days, the Rust team is [posting a lot of vision documents](https://blog.rust-lang.org/inside-rust/2022/02/03/async-in-2022.html) about what an ideal version of the Rust async ecosystem would look like.

One common thread is that, in the "shiny future", async should be a non-obstrusive feature. It should get out of the way of the user, who should barely have to think more about their code than if they were writing sync code.

We could imagine a similar vision for unsized values in Rust.

Code should be equally easy to write whether values are sized or unsized, *within the limits a reasonable programming model*. So, while some patterns might be impossible, the user shouldn't worry about triggering a stack overflow when writing this code:

```rust
fn loop_with_copies() {
    for _ in 0..1000 {
        let x: Box<LargeValue> = get_large_value();
        let _x = *x;
    }
}
```

whether `LargeValue` is sized or unsized.

I think it might be interesting to write a design document outlining what the "limits a reasonable programming model" should be. I think the current state of unsized locals is limited by what LLVM allows with alloca, but with some design work could find designs that are more convenient for users while still being feasible to implement on rustc's backends (LLVM, GCC, cranelift, etc).

In the meantime, what these features need is attention. If the Rust team is serious about allowing trait-safe async, then work on unsized value needs to pick up a lot of steam.

Discussion on [r/rust](https://www.reddit.com/r/rust/comments/szgpxc/analyzing_unsized_variables_in_rust/).
