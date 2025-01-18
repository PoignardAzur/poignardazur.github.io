---
layout: post
title: Failures of git merge
---

Intro: show git merge working correctly

False negatives:
- Imports
- Indentation changes
- Argument lists

False positives:
- Symbol renames


```rust
log!("the id is {}", foo.id);

if do_stuff {
    // do stuff
}
```

One side changes `foo.id` to `foo.get_id()`

The other moves the log to `do_stuff()`

---

A B C X Y Z

One side adds D E F the other adds U V W
