---
layout: post
title: Remark on Rust's 10th anniversary.
---

Last month, [RustWeek 2025](https://rustweek.org/) took place, during which we celebrated the 10th anniversary of [the Rust programming language](https://rust-lang.org).

As it happened, that day was the scheduled date for [the release of Rust 1.87](https://blog.rust-lang.org/2025/05/15/Rust-1.87.0/), as part of its regular six-weeks release trait.

If you're anything like me, and you might be wondering how that works: did the two dates really happen to coincide?

Let's do the math:

- Rust has one release every six weeks.
- `87 * 6 * 7 == 3654`
- `3654 == 365 * 10 + 4`

Which makes exactly 10 years, plus four leap days! So the math checks out: between 2015 and 2025, there have been four bissextile years: 2016, 2020, 2024, and... wait a minute.

So, after looking into the dates of various Rust releases (thank you [releases.rs](https://releases.rs/)) and checking the answers ChatGPT gave me, it looks like while releases were supposed to always come out on Thursdays, Rust 1.0 was late by one day, and came out on May 15, 2015, on a Friday.

Next release came out 41 days later and established the release-on-thursdays schedule.
Other releases broke the once-every-six-weeks pattern: patch releases, and off-by-one errors.

For instance, Rust 1.7 came out on Wednesday, March 2, 2016, 41 days after 1.6 and 43 days before 1.8.

Overall, the release schedule stuck very close to a pattern of "1.N comes out `42 * N` days after May 14, 2015".

Which leads us to Rust 1.87, which came out 3654 days after that date, which, through the magic of off-by-one errors, happens to be precisely 10 years after the release of Rust 1.0.

Anyway, there's no grand lesson here, it was just fun to go down this rabbit hole.

Knowing this useless factoid fills you with determination, probably.
