+++
title = "Intermediate Rust series"
[taxonomies]
tags = [ "rust" ]
+++

Hi all!

A few days ago I [posted on Twitter](https://twitter.com/jntrnr/status/1358592787852140550?s=20) asking for topics that people wanted to see covered for intermediate Rust content. The response was awesome and it's inspired me to make a series where I go through each topic and do my best to cover it as a video and/or blog post.

Here's the current list of topics. If you see something missing, you can drop an issue on [this repo](https://github.com/jntrnr/jntrnr.github.io) or ping me [on Twitter](https://twitter.com/jntrnr) and I'll try to add it.

## Rust features
* Associated types
* Working with lifetimes
* `Pin` and `Pin<Box<Stream>>`
* Macros, proc macros, custom attributes
* Concurrency
* Async/await
* Threads/tasks
* MPSC theory
* Futures
* Send/Sync
* Unsafe
* FFI, shared mem across FFI, closures/function pointers over FFI
* FFI to C and C++
* C interop, opaque structures, repl(c), working with null, alloc/free, bitflag export to C
* What tools to use for C++
* `std::sync` (Arc, Rc, RefCell, etc)
* Raw pointers
* Impl trait, dyn trait, and trait objects
* Fn traits
* Field sensitivity, field sensitivity + smart ptrs, field sensitivity and closures
* Why do we cast from `Box<Struct>` to `Box<dyn Trait>`
* How to use `[patch]` in Cargo.toml
* `std::mem`, swap, replace, take

## Crates
* Tonic
* Tokio
* Deeper dive into existing frameworks/libraries
* Channels/async/par_iter (concurrency vs parallelism)

## Projects
* Basic SQL database
* Interpreters and parsers
* Tokio + io_uring for CLI apps
* Filesystem + FUSE
* Seda or actor based project
* Wasm + wasm bindgen

## Patterns and practices
* Like http://railstutorial.org but for Rust
* Functional composition
* Async functional composition
* Dependency injection (and alternatives)
* Design patterns
* Lessons from Rust in production
* How to teach Rust to teams
* Testing patterns and strategies
* https://rust-lang.github.io/api-guidelines/checklist.html
* Zero copy processing best practices
* Error handling. One big error enum or something extensible? Anyhow?
* Debugging Rust
* Modelling classic data structures in Rust
* Logging
* Shared data structures in long-running apps
* Profiling
* Improving compile times

## Rust background
* How do nightly features stabilize?

# Looking ahead

As you can see, there's a *lot* of area to cover. My plan is to make steady progress on the list above, covering topic by topic until we cover as many of the topics as possible. People were also upvoting tweets with likes, so I may juggle this list around so that I'm sure to cover the hot topics sooner.

Excited to dig into this!

If you'd like to follow along to enjoy the content as it comes out, you can follow me [on Twitter](https://twitter.com/jntrnr), you can follow my [YouTube channel](https://www.youtube.com/user/giard321) where I post my technical content, and follow me on [Twitch](https://www.twitch.tv/jntrnr) for when I stream a topic.
