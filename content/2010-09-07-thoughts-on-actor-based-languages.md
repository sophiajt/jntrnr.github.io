+++
title = "Thoughts on actor-based languages"
[taxonomies]
tags = [ "languages" ]
+++

(or "Minnow, a post-mortem")

I've been fortunate to work on both [Minnow](https://github.com/jntrnr/minnow-language/tree/alpha3)(a shared-nothing actor-based language) and [Chapel](https://chapel-lang.org/)(concurrent global-view language). I wanted to give a few thoughts comparing the two approaches.

First, when we talk about concurrency or parallelism, without going into the pedantic definitions of each, we should answer the question "why not stay serial?"  There are a lot of reasons to do so, if we can.  Serial code is going to be easier to write, easier to debug, and easier to reason about.  Generally, going parallel is just going to complicate our lives.

I see two main reasons why going parallel would be required:

 - **Reliability** - Does the system have to tolerate failures of its parts?  
 - **Performance** - Is serial too slow for your application?

In [Erlang](http://www.erlang.org/), reliability is king.  Its shared-nothing actor-model style allows actors to fail and be restarted, which allows for more advanced features like hot code swapping.  

For areas where performance is king, C and Fortran still reign.  In high-performance scientific computing, it's common to remove abstractions and program as "close to the metal" as possible.  Though this doesn't preclude reliability, that reliability must either be done by the programmer or by the underlying architecture.  It is, as you can imagine, very fast.  

When I created Minnow, I set out to create an actor-based language that could perform as fast as C but potentially gain the benefit of reliability from the actor abstraction.  Focusing solely on single node, shared-memory machine, I was able to optimize:

 - Message sends, down to a few atomic operations
 - Number of actors possible, using light-weight threads and continuations
 - Run-time overhead, even in the presence of fairness counters for cooperative multi-tasking, by minimizing extraneous work in loops and function calls 

In short, Minnow was quite fast.  

Was it fast enough?  As it turns out, the answer is *no*.  Though Minnow avoiding copying where possible, it did not allow the user to see a large matrix and operate on pieces of it. Doing so would go against the notion of shared-nothing, we might even say that shared-nothing runs counter to the goal of performance.  Algorithms where we could bifurcate data into chunks, for example, wouldn't suffer, but in general we can no longer use contiguous memory in a natural way.

The modern batch of concurrent languages (Chapel, X10, Co-Array Fortran, and others) return to looking at memory in a contiguous way.  In fact, using [partitioned global address space](https://en.wikipedia.org/wiki/Partitioned_global_address_space) (or PGAS), the notion of contiguous memory is extended to encompass the memories of all the nodes in a cluster.  

The question of where reliability should live, and if we should push it into the language and sacrifice performance, appears to be solved issue in the high-performance community.  It remains to be seen if the larger programmer community will embrace this approach.

