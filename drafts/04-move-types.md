From https://www.reddit.com/r/rust/comments/l4roqk/comment/gkttr7q/?utm_source=reddit&utm_medium=web2x&context=3

---

Is that all?

Okay, here's my pitch then:

Add a `Move` effect in an opt-out way (that is, types would be `Move` by default like they are now). Only functions that take `?Move` types have to worry about different semantics.

The major problem is that this creates fragmentation, because `?Move` types can only be used with functions that have been coded to accommodate them. So for instance, you can't use iterator functions, because your `Iterator::map` and `Iterator::filter` and so on take `Move` types. You can add a `?Move` annotation to these functions, but then you're adding another annotation to the entire ecosystem that means "this doesn't do anything special", all to serve a niche use-case.

Instead, to solve that problem, I propose that for any type `MyType: !Move`, shared references to `MyType` can be bound to shared references of generic `T` types, even if `T` is considered `Move`, **only if** there is no way to produce a new `T` from the bounds (so `T: Clone` isn't allowed).

This is safe because you can't move out of a shared reference (except for cells, but we can just forbid unmoveable cell types); and the "no way to clone T" rule means that you can't produce a new `MyType` that you then move. So from the point of view of the called function, `MyType` may as well be `Move`.

Some notes:

- For simplicity, the "no way to produce a new `T` from the bounds" rule could be simplified to "only object-safe trait bounds".
- I'm not sure how well the above works with covariance rules for references. My gut says "it works well enough".
- You probably have other things to worry about, like the trait bound having an associated const of type `MyType` that could then be moved from; or how multiple trait bounds interact together.
- `!Move` types probably would have other rules; for instance, making `Pin<mut&>` references to them would be automatically safe, but that's secondary.
- Existing unsafe code would remain valid, because it would only be called on either `Move` types or shared references to `!Move` types. Code that wants to handle mutable `!Move` types (and actually move them around and all) would have to add an effect annotation, which indicates that they did consider the different semantics.