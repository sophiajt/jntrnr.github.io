---
title: Announcing Rust Language Server Alpha Release 
---

Today, we're announcing the first alpha release of the [Rust Language Server (aka RLS)](https://github.com/jonathandturner/rls).  With this alpha release, this is the first time we're encouraging early adopters to try on real projects and [send us feedback](https://github.com/jonathandturner/rls/issues).  The RLS has now reached a level of maturity where it should be able to run against most Cargo-based Rust projects.

But what exactly is the Rust Language Server?

# In a nutshell

The Rust Language Server is a way of providing editors and IDEs with a range of functionality.  Rather than leaving each editor plugin to have to parse and understand the types in your program and provide you with capabilities like refactoring, the RLS centralizes all this logic and provides it to the editor via a standard [language server protocol](https://github.com/Microsoft/language-server-protocol).

In this alpha release, this allows editors and IDEs to share the following capabilities:

* **auto-completion** - allowing you to complete symbols and press '.' for member lists
* **goto definition** - jump to the definition of a given symbol
* **goto symbol** - jump to the definition of a symbol you know the name of
* **find all references** - show all the locations a given symbol is referenced
* **rename/refactor** - rename all instances of a given symbol to a new name
* **types on hover** - get the type of a symbol
* **show errors** - as the user types, get live analysis showing errors as they happen

The alpha release of the RLS has been run successfully on Linux, Mac, and Windows.

# How to get started

**Step 0**: Make sure you have a recent [nightly rustc/cargo](https://www.rust-lang.org/en-US/other-installers.html), **git**, **python**, **node**, and **cmake** installed

**Step 1**: Check out the RLS:

```
git clone https://github.com/jonathandturner/rls
```

**Step 2**: Set up your editor (here we show [VS Code plugin](https://github.com/jonathandturner/rls_vscode))

```
git clone https://github.com/jonathandturner/rls_vscode.git
cd rls_vscode
npm update
```

**Step 3**: Set the RLS_ROOT environment variable to point to where you checked out the RLS:

```
export RLS_ROOT=/Source/rls
```

At this point, you can open the project in VS Code, and when you run the project, it will open a new instance of VS Code with the RLS plugin enabled.  The plugin will build RLS for you.

**Note:** the initial analysis of a project can take quite a long time, as metadata is built for all the project dependencies.

**Note 2:** analysis currently ignores test code, so you won't get IDE support for it.  We have a fix for this in the works.

# How does it work

The current version of the RLS is built from a combination of two tools: [racer](https://github.com/phildawes/racer) and the [Rust compiler](https://github.com/rust-lang/rust/).

Racer allows us to get quick-and-dirty completions, allow you to get completion results in sub-second times.  The trade-off this makes is that the results are not as accurate.

For tasks that require higher accuracy, like safe refactoring or code navigation, the RLS uses the Rust compiler directly.  Because this type of analysis, while more accurate, takes longer to complete, the RLS will tell the editor when the analysis has successfully completed.

# Help, it doesn't work!

The RLS is closely tied to particular versions of rustc.  If you're a rustup user, the DLLs it needs are managed by rustup.  In this case, you'll get an error like this if you run the rls by hand:

```
jturner-23759:rls jturner$ ~/.cargo/bin/rls 
dyld: Library not loaded: @rpath/librustc_driver-6eb85298.dylib
  Referenced from: /Users/jturner/.cargo/bin/rls
  Reason: image not found
Abort trap: 6
```

If you see an error like this, you can run the rls by instead running the command `cargo run` in the rls directory.  In the future, as we improve the rls integration with the main Rust tools, this will no longer be an issue.

# What's next

The RLS is still early in its life and needs to go through a stabilization period before it's ready for 1.0.  This alpha release marks the first release in this series.

The next release will be a [second alpha](https://github.com/jonathandturner/rls/milestone/3) that includes another set of bugfixes and polish.  This will be followed by a [beta](https://github.com/jonathandturner/rls/milestone/4).  The beta will be the first that will be distributed as a binary and will be much closer to final 1.0 experience in terms of ease-of-install and ease-of-use.

# I want to help

Great!  There are a few ways you can jump in.

**Test the RLS** - use the RLS on your projects and tell us how it feels.  This will help us shake it out and make sure it's ready for production.

**Contribute to RLS:** - we'd like to add more refactorings, more Rust-centric code navigation (eg find all impls for a type), and possibly more advanced features like lifetime visualization and macro debugging.

**Contribute an editor plugin** - there are already at least [six plugins](http://langserver.org/) that follow the Language Server Protocol that the RLS uses, each in its own state of completion.  If you have experience with one of those editors, or if you have a favorite editor you don't see on the list, you can help us by improving support.  While the RLS will continue to grow and mature, it's really the plugin where the "rubber meets the road".  It's the plugin that helps the developer get the most out of what the RLS provides.

**Hack on the compiler** - ultimately, we want to drive everything in the RLS straight from the compiler.  This means being able to have sub-second response times for interactive completions as well.  The Rust compiler team has already started [a series of refactoring](https://github.com/rust-lang/rust/pull/37400) to make the transition to an interactive mode possible.  If you have a background in compilers, IDEs, or static analysis tools and would like to help with this work, you can contact the compiler team at #rustc on irc.mozilla.org or through the [Rust compiler project on GitHub](https://github.com/rust-lang/rust/).
