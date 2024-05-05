This is going to be pretty unstructured. I'm trying to point at a general area and see if anybody concurs.

"DSL", or "domain syntax languages", are languages made for a more specific type of task than your traditional programming languages. DSLs can be for config files, for serializing data, or for expressing more patterns more concisely, but that's not really what I want talk about.

What I'm interested in here is DSLs that represent *code*, as in, computations. Either *imperative* code (eg A happens, then B, then C) or functional code written like imperative code (eg let A = foo, let B = bar, return A + B).

I think there's a lot of potential for the latter in multiple domains:

- Hardware logic design.
- GPU shader languages.
- Tensor graphs for ML.
- Incremental computation.
- (maybe high-level GUI?)

Basically anything that takes a graph of computations with a certain structure and transforms it into a graph with a different structure.

I think Rust has a lot of features that would translate well to these domains:

- A rich type system (tuples, enums, generic types, etc).
- Rich control flow (match, if let, for loops, let-else, etc).
- Generics, including const generics.
- The trait system.
- Easy composability with traits and modules.
- Constrained mutability.
- Great error messages.

The last one is the one I'm most interested in. It's the major reason a lot of people love Rust: it warns you early if you do something wrong, and it often warns you in a way that's actionable.

While it's possible to make a DSL for any / all of the domains I've cited above, all the ways I'm aware of tend to not degrade gracefully. Usually the error messages they give you aren't actionable *unless* you know the internals of the DSL enough to understand what the compiler is complaining about (see Appendix).

## Example - Using Rust for tensor graphs

```rust
fn attention(k: Tensor<Dim>, Tensor<Dim>, tokens: &[Tensor<Dim>]) {
  for token in tokens {
  
  }
}
```

Composability:

I want to be able to import fragments of the DSL from other fragments. I want to be able to compose them together easily.

I want hot reloading.


## Appendix - Types of DSL

There are many ways to make a DSL:
- Attribute macros
- Inline macros
- Operator overloading
- Locking data read/writes behind an API (eg React useXXX())
- Lots of closures


---

(from previous post)

The magic formula that makes Rust amazing to work with has many ingredients. Many of these ingredients feed into each other, eg:

- A very clean import system
- A very straightforward package manager.
- A standard way to run a project immediately after cloning it.
- A standard way to run tests.
- A standard way to write tests.
- First-class error handling.
- Pattern-matching.
- if let, let else, for loops.
- Rules around mutability which make it really clear what a given part of a code can or can't do.
- A trait system that makes it easy to describe complex abstractions checked before monomorphization.

I'm trying to get into HDLs (Hardware Design Languages, languages that describe a virtual circuit) and I miss a lot of the features I just listed.

---

Hardware synthesis uses *a lot* of eDSLs.

Eg: Chisel, MyHDL, SystemC

(Also Bluespec is a not-quite-eDSL of Haskell)