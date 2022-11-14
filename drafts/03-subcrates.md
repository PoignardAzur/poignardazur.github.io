TODO - Interview multi-crate developers
(tokio?)
(matklad?)

https://www.reddit.com/r/rust/comments/ydcpgk/inline_crates/
https://blog.yoshuawuyts.com/inline-crates/

https://matklad.github.io/2021/08/22/large-rust-workspaces.html

https://matklad.github.io/2021/09/04/fast-rust-builds.html

> The first advice you get when complaining about compile times in Rust is: “split the code into crates”. It is not that easy — if you ended up with a graph like the first one, you are not wining much. It is important to architect the applications to look like the second picture — a common vocabulary crate, a number of independent features, and a leaf crate to tie everything together. The most important property of a crate is which crates it doesn’t (transitively) depend on.
>
> Another important consideration is the number of final artifacts (most typically binaries). Rust is statically linked, so, if two different binaries use the same library, each binary contains a separately linked copy of the library. If you have n binaries and m libraries, and each binary uses each library, then the amount of work to do during the linking is m * n. For this reason, it’s better to minimize the number of artifacts. One common technique here is BusyBox-style Swiss Army knife executables. The idea is that you can hardlink the same executable as several files with different names. The program then can look at the zeroth command line argument to learn the name it was invoked with, and use it effectively as a name of a subcommand. One cargo-specific gotcha here is that, by default, each file in ./examples or ./tests folder creates a new executable.