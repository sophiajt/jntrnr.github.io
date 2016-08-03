---
layout: post
title: "Helping with the Rust Errors"
author: Jonathan Turner
---

I recently did some work with Niko Matsakis on a new compiler error format. You can try them out
by setting `RUST_NEW_ERROR_FORMAT=true`. While I put together a
blog post talking about the design for the main blog, I also wanted to open it up for people to come
help us move to the new errors. I've split up this post based on background for testers, doc
writers, first-time compiler hackers, and experienced compiler hackers.

# I'm good at testing!

Perfect! We've love your help to come up with tests that show where an error message is confusing or
just plain wrong. If you'd like a little help getting started, there are a bunch of unit tests for
the compiler that all fail intentionally.  You can find these in the rust-lang repo under the
`src/test/compile_fail` directory. You'll notice we've already started a
[list of errors](https://github.com/rust-lang/rust/issues/35233), but that list is incomplete. We'd
like to make sure every error code that needs to be updated makes it onto that list so that others
can help update them.

There are a *bunch* of good examples in the `compile_fail` directory that explore various error
codes. If you look around there
you can also search for error messages and see if there is already a test. Running this test will
help you see if the error has been updated to the new format.

If you can find a test case that needs to be updated, file an issue like one of those listed in the
"list of errors" link and cc @jonathandturner. I can help get the error updated to the new format
and tag the error so that others can find it.

# I love writing docs!

Good, we need that too. There are a couple places where doc writers can help out, especially if
if you already know a little Rust.

The first place is helping us write clearer error messages. Even now, and especailly as we move to
the new format, some errors are confusing and sometimes wrong. By helping write clearer error
wording, and by experimenting with new ways to describe the error that is occuring, you can help us
further improve the errors for the next developer.

If you look at the [list of errors](https://github.com/rust-lang/rust/issues/35233) we have so far
you can see what we're going for. The likely need here is that errors need new labels. The general
style of the new errors:

* The error message text. This describes the general problem.
* The primary error label (the '^^^' underline). This describes the 'what' of the error.
It tries to summarize what went wrong in an fairly brief, yet approachable way.
* The secondary error label (the '---' underline). This describes the 'why' of the error. By reading
these the user can see the order of operations that lead to the error.

You can also explore the `src/test/compile-fail` of the compiler source code for a lot of unit
tests that fail intentionally.  By looking through these, you can help find errors that aren't
update or that use confusing notes and labels.

Worth mentioning too, if you're a writer, also keep a lookout for Stage 2 (which hopefully will be
ready in the coming weeks).  We'll need help in the future writing helpful extended messages that
describe errors. A great place to jump into help out here and learn about other places doc writers
can help is #rust-docs on irc.mozilla.org.

# I know Rust but I've never hacked the compiler!

Excellent, a first-time compiler hacker. Let's walk through updating an error
message to the new format. You can look through the
[list of errors](https://github.com/rust-lang/rust/issues/35233) to see which ones need updating.

Once you've downloaded the [Rust compiler source](https://github.com/rust-lang/rust), run the
command:

```> python src/bootstrap/bootstrap.py --step libstd --stage 1```

This will build "Stage 1" of the compiler. After the first compile, this saves us a *bunch* of time
as we test our ideas as it is the minimal compiler rather than a full install.

Once built, we can run this by hand using (here, I'm on OS X):

```./build/x86_64-apple-darwin/stage1/bin/rustc <rust file>```

Let's take a look at an error that needs to be updated. You can find a lot of error tests in the
`src/test/compile-fail` directory in the compiler source. For this example, I'll use
`src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs`. You can also see which
error this is looking for the EXXXX number, which is the error number.  This one is E0384.

```term
error[E0384]: re-assignment of immutable variable `x`
  --> src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:50:13
   |
50 |             x += 1;
   |             ^^^^^^
   |
note: prior assignment occurs here
  --> src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:49:10
   |
49 |         [x,_,_] => {
   |          ^
```

We can see that's still using some of the old logic because the underlines don't have a label. You
can also tell because there's an attached note is simple enough to be that label. Let's change
the note to be a label.

To fix this, we first need to find it in the compiler source. Search tools to the rescue. I tend to
use the error number (and sometimes part of the error message text)

```term
jturner-23759:rust jturner$ grep -r E0384 src
src/librustc_borrowck/borrowck/mod.rs:            self.tcx.sess, span, E0384,
src/librustc_borrowck/diagnostics.rs:E0384: r##"
```

We now have the line that creates the error and another that lists it for us. We only need to change
the first of these.

Looking at the code, sure enough, this is using the older style:

```Rust
struct_span_err!(
    self.tcx.sess, span, E0384,
    "re-assignment of immutable variable `{}`",
    self.loan_path_to_string(lp))
    .span_note(assign.span, "prior assignment occurs here")
    .emit();
```

This is a pretty easy fix. We just move from `.span_note` to `.span_label` and create a simple
label, like this:

```Rust
struct_span_err!(
    self.tcx.sess, span, E0384,
    "re-assignment of immutable variable `{}`",
    self.loan_path_to_string(lp))
    .span_label(assign.span, &format!("prior assignment occurs here"))
    .emit();
```

Once we build and run the new compiler against the error message, we get something where the label
is now part of the main snippet.

```term
error[E0384]: re-assignment of immutable variable `x`
  --> src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:50:13
   |
49 |         [x,_,_] => {
   |          - prior assignment occurs here
50 |             x += 1;
   |             ^^^^^^
```

That looks a little better, but we're not done. You'll notice that we don't have a label on our
primary span (the ^^^^ part). In the new format, we like to have a label here because it's one of
the first places your eye will look when you see an error message.

The second is that the label "prior assignment occurs here". That feels a little awkward. Since the
label is now part of our code snippet, we can use a better wording. Let's update the error to look
like this:

```term
error[E0384]: re-assignment of immutable variable `x`
  --> src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:50:13
   |
49 |         [x,_,_] => {
   |          - first assignment to `x`
50 |             x += 1;
   |             ^^^^^^ re-assignment of immutable variable
```

With a little bit of wording changes and the new primary label, that's not bad. Of course, as you
get comfortable working with these errors you may find even better wording that's more clear. Next,
let's update the code to output this style:

```Rust
struct_span_err!(
    self.tcx.sess, span, E0384,
    "re-assignment of immutable variable `{}`",
    self.loan_path_to_string(lp))
    .span_label(assign.span, &format!("first assignment to `{}`",
                                      self.loan_path_to_string(lp)))
    .span_label(span, &format!("re-assignment of immutable variable"))
    .emit();
```

You'll notice that I reused the same `span` that the main error message uses in my new
`.span_label`. When I do this, I'm telling the DiagnosticBuilder that this span_label is talking
about the primary span and is the root of what is causing the error. Other labels are secondary
spans because they're more informational.

And with that, we're done! Well, almost. We can't forget to update the unit tests.

To quickly check the unit tests that will fail because of our changes, we can run:

```python src/bootstrap/bootstrap.py --step check-cfail --stage 1```

Compile-fail unit tests are set up with their own annotations that say what labels to expect and
where. As we update errors, we'll need to also update the corresponding test the new expected
labels.

```term
failures:
    [compile-fail] compile-fail/asm-out-assign-imm.rs
    [compile-fail] compile-fail/assign-imm-local-twice.rs
    [compile-fail] compile-fail/borrowck/borrowck-insert-during-each.rs
    [compile-fail] compile-fail/liveness-assign-imm-local-in-loop.rs
    [compile-fail] compile-fail/liveness-assign-imm-local-in-op-eq.rs
    [compile-fail] compile-fail/liveness-assign-imm-local-with-init.rs

test result: FAILED. 2381 passed; 6 failed; 15 ignored; 0 measured
```

Sure enough, we need to update some of the unit tests. Let's run these individually. To do this, we
can use:

```python src/bootstrap/bootstrap.py --step check-cfail <test name> --stage 1```

like so:

```python src/bootstrap/bootstrap.py --step check-cfail asm-out-assign-imm --stage 1```

A fair amount of text comes out, but what we're interested in is:

```term
error: /Users/jturner/Source/rust/src/test/compile-fail/asm-out-assign-imm.rs:21: unexpected "note": 'first assignment to `x`'

error: /Users/jturner/Source/rust/src/test/compile-fail/asm-out-assign-imm.rs:24: unexpected "note": 're-assignment of immutable variable'

error: /Users/jturner/Source/rust/src/test/compile-fail/asm-out-assign-imm.rs:21: expected note not found: prior assignment occurs here
```

These lines tell us what's breaking our test case. In this example we see there are notes being
printed that it doesn't expect and one expected note not found. This makes sense, since we changed
the wording in the error. Let's go ahead and fix this.

We can change:

```Rust
    x = 1; //~ NOTE prior assignment occurs here
```

To:

```Rust
    x = 1; //~ NOTE first assignment
```

The `//~` tells the test runner what the expected message is. Here, the message is a NOTE, which
corresponds to the label we changed.

One more to go. We also added a new label on the primary span. We can insert a new note where
this goes:

```Rust
    unsafe {
        asm!("mov $1, $0" : "=r"(x) : "r"(5));
        //~^ ERROR re-assignment of immutable variable `x`
        //~| NOTE re-assignment of immutable
        //~| NOTE in this expansion of asm!
    }
```

*Notice: we use the `//~` when the note is by itself, or `//~^` and `//~|` when we're looking
for multiple notes.*

With that, the test now passes:

```term
running 1 test
test [compile-fail] compile-fail/asm-out-assign-imm.rs ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
```

From there we work our way through the errors. Having these error tests failing gives us a chance
to be sure that our fix is good. Here, when we fix
`compile-fail/liveness-assign-imm-local-in-loop.rs` we see that in a loop the two spans we're using
might be the same span if the error happens in a loop. Let's fix that:

```Rust
let mut err = struct_span_err!(
        self.tcx.sess, span, E0384,
        "re-assignment of immutable variable `{}`",
        self.loan_path_to_string(lp));
err.span_label(span, &format!("re-assignment of immutable variable"));
if span != assign.span {
    err.span_label(assign.span, &format!("first assignment to `{}`",
                                         self.loan_path_to_string(lp)));
}
err.emit();
```

While we're at it, let's run a quick tidy to make sure our code conforms to the compiler coding
standard:

```python src/bootstrap/bootstrap.py --stage 1 --step check-tidy```

Perfect.  Ship it!

After we've finished with updating the error and the corresponding unit tests, we can create a PR.
When you file the PR, put the message "r? @jonathandturner" at the bottom of your commit
message. This will add me to do the code review.

# Extra credit

A keen eye will notice that the file we've been working on did not show up in our failed tests.
"Why is that?" you might ask. Unit tests that look at the compiler messages work in a few different
ways. Some look at how a test looks (eg, tests in the src/test/ui directory). Others make sure the
content is there, like the unit tests we just fixed. Because there is a lot of content to look for
there are two kinds of "look for content" error test cases. The first kind will only look at the
main message. You can see that this is the case because these test cases have no indicators to look
for NOTE or HELP, only ERROR and WARN.

Watch what happens if we add a single NOTE to our original test case:

```term
error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:25: unexpected "note": 'first assignment to `x`'

error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:32: unexpected "note": 'first assignment to `x`'

error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:33: unexpected "note": 're-assignment of immutable variable'

error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:38: unexpected "note": 'first assignment to `x`'

error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:39: unexpected "note": 're-assignment of immutable variable'

error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:44: unexpected "note": 'first assignment to `x`'

error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:45: unexpected "note": 're-assignment of immutable variable'

error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:50: unexpected "note": 'first assignment to `x`'

error: /Users/jturner/Source/rust/src/test/compile-fail/borrowck/borrowck-match-binding-is-assignment.rs:51: unexpected "note": 're-assignment of immutable variable'
```

That's more like it! Now, we can continue fixing this unit test like we did the others.

## Tips

As you convert errors you may need to change `span_err` (or `span_err!`)
to `struct_span_err` (or `struct_span_err!`), add new .span_labels, or tweak the wording for the
error message to feel right.

# I'm an experienced Rust compiler hacker!

Perfect! There are a few hairy things we need to figure out, still, and they may require the keen
eye of a compiler hacker. In the [list of errors](https://github.com/rust-lang/rust/issues/35233)
there is a second list of issues for experienced compiler hackers. Additionally, there are areas
that need a bit more work to make them feel like the new messages.

## Region errors

Regions errors currently provide a decent amount of information, but unfortunately to understand it
you currently need a pretty decent understanding of Rust already. Ideally, these messages would get
a similar treatment to the borrowck errors mentioned above.

Something like:

```term
error[E0495]: cannot infer an appropriate lifetime for lifetime parameter 'r in function call due to conflicting requirements
  --> src/test/compile-fail/regions-infer-call-3.rs:18:24
   |
18 |     let z = with(|y| { select(x, y) });
   |                        ^^^^^^^^^^^^
   |
note: first, the lifetime cannot outlive the anonymous lifetime #1 defined on the block at 18:21...
  --> src/test/compile-fail/regions-infer-call-3.rs:18:22
   |
18 |     let z = with(|y| { select(x, y) });
   |                      ^^^^^^^^^^^^^^^^
note: ...so that reference does not outlive borrowed content
  --> src/test/compile-fail/regions-infer-call-3.rs:18:34
   |
18 |     let z = with(|y| { select(x, y) });
   |                                  ^
note: but, the lifetime must be valid for the call at 18:12...
  --> src/test/compile-fail/regions-infer-call-3.rs:18:13
   |
18 |     let z = with(|y| { select(x, y) });
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^
note: ...so type `&isize` of expression is valid during the expression
  --> src/test/compile-fail/regions-infer-call-3.rs:18:13
   |
18 |     let z = with(|y| { select(x, y) });
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^
```

You can see that we say things like "for the call at 18:12" or "on the block at 18:21". The span
information is available but we're not really using it well. Instead of saying this, we may want to
calculate the actual span that relates to "the call at 18:12" or the end point of "the block at
18:21".

## Macro errors

I've started a thread on
[improving macro errors](https://internals.rust-lang.org/t/improving-macro-errors/3809) on the Rust
internals list. The hope is to look at a better overall design for macro errors that fits better
with the new error style.

# Thanks!

Thanks for wanting to jump in and help! You can find me on irc.mozilla.org as jntrnr and on
github.com as jonathandturner. If you have any questions, please ask. You can also get lots of good
help in #rust-internals and #rustc on irc.mozilla.org.

