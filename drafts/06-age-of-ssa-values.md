Let's say you're writing a CPU core (can be registers, stack machines, something more fancy), and you're trying to figure out how optimized it is

The fundamental problem is you're not just measuring time, you're trying to find "time that wasn't necessary to spend".

One interesting metrics could be "age of ssa value". Every cycle the value would increase by one. Moving the value from one register to another would preserve its age. Doing a select of two values would produce a value with the youngest age.

The point would be to detect occasions for prefetching and other optimizations. If a load takes 100 cycles to retire, but the address you have it had >100 cycle of age, that means you missed an occasion to prefetch it early and completely hide the load latency.
