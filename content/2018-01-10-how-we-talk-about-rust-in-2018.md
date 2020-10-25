+++
title = "Talking about how we talk about Rust in 2018"
[taxonomies]
tags = [ "rust" ]
+++

_This is an entry in the #rust2018 blogging efforts to talk about Rust in 2018_

When you come to Rust, you're bound to hear a lot of phrases.  "Wrestling with the compiler". "Rust evangelism task force". "Non-lexical lifetimes". "Systems programming".  Some seem strange. Others are a bit scary.  Some are just a common set you quickly need to learn to before you can even read the Rust news.

In this post, I want to talk about how we talk about Rust in 2018.  I think there are some ways we can change what we're saying and how we're saying it. These changes will help Rust be more approachable for beginners and an even nicer place in general.

# "Wrestling with the compiler"

I think we've all been there. Especially when we're starting with Rust, or trying to do something new with it we haven't done before, we feel like the compiler is fighting us every step of the way.

Sure, it's frustrating. On a bad day, we might even say "forget it". I wonder, though, is there a better way to talk about it? If you're a Rust user, do you trust the compiler?  Are the error messages helping to clarify your thinking?  Is it, as some people like to say, a pair programming partner for you?

I suggest in 2018, we kick the idea of wrestling with the Rust compiler to the curb and focus on how it helps us rather than the idea of it beating us down.  Niko mentioned [error messages](http://smallcultfollowing.com/babysteps/blog/2018/01/09/rust2018/) here to help with that, and I think we should push hard there.  While we can change how we talk about the compiler, we can also change how the compiler talks to us as well.

By the end of 2018, no more wrestling.  It's a tool to help us, and we should work hard both in how we talk and how it works to ensure this is true.

# "Rust evangelism task force"

We've seen the jokes about Rust having its own task force that swoops in and says things like "you should rewrite it in Rust".  Sometimes the swoop happens just as a new security exploit is released.  Sometimes it comes out of nowhere.

Or maybe you've seen this: someone on a thread, maybe on reddit, posts that *maybe* Rust isn't doing so well in a benchmark or in a particular scenario.  Minutes later there are half a dozen comments dissecting what happened.

It's intimidating.  And really, it's not helpful to Rust.

As a community, it's great that we have so much enthusiasm.  I just want to be sure we're channeling that enthusiasm in a way that keeps Rust growing.  There are ways we can be courteous by not piling on or jumping in with uninvited suggestions.  That might seem like little things, but at scale I think they'll make a big difference.

Some of you may be familiar with burntsushi's ripgrep tool.  The author of a similar tool, ack, had a [great quote](http://blog.petdance.com/2018/01/02/the-best-open-source-project-for-someone-might-not-be-yours-and-thats-ok/) about the other day that applies to more than just search tools:

> "If someone uses ripgrep instead of ack, it doesn’t hurt me.  It’s the difference between an abundance vs. scarcity view of the world.  I choose abundance."

We can adopt the same way of thinking.  If someone wants to use Go or C instead of Rust, that's great!  Perhaps that's a better fit for what they're working on.  Maybe they'll try Rust again in the future.  We should make it easy to come and go.

# "Systems programming"

There are a lot of terms that we use when talking about Rust that I think are simply confusing.  Have you ever had this conversation?

**Person 1:** Have you heard of the Rust programming language?

**Person 2:** I'm not sure, what is it?

**Person 1:** It's a systems programming language.

**Person 2:** What's a systems programming language?

Rather than answering the question, we just made it seem even more complicated.  Confusingly, even among people who have used Rust for a while there isn't even total consensus on what is meant by a systems programming language.  Doubly confusingly, Rust isn't even *just* a systems programming language anymore.  With the ability to create web applications and tie into other languages, it starts to feel more than just systems-level development.

You can use Rust to build pretty much anything from a low-level operating system component to a JSX-like web app.  In the future, [we'll be in space](https://twitter.com/marshall_law/status/944031810941603840).  Systems is both too confusing and too limiting for what Rust will be used to do.

Maybe there's a better term that we can use, or a small phrase that captures the right feel?

# "Non-lexical lifetimes" and more

Lots of the Rust community is pretty facile when it comes to computer science.  Some of us have dabbled in Haskell.  Others are old-hands at C++.  We've seen programming from different angles and have a lot of vocabulary to use to describe Rust.

This started out as pretty effective short-hand.  "This is like typeclasses in Haskell" we might say.  "Lifetimes are lexically-scoped".  "You need to use a lifetime variable, they're just like generics..." Etc.  We can use it to communicate, but the end result can be a bit of a mouthful.

I'm not going to suggest we remove these terms from our vocabulary entirely, but I want us to think about how we can improve what we write publically.  People we don't know will come by IRC, read our blog posts, and see our forum threads.  They'll be coming from a variety of backgrounds, many of which will be different from our own.

If where we're talking is public with a wide audience, let's keep that in mind as we communicate.  Keep the door open for more people to join.

Rust, at its heart, isn't intimidating.  It's a technology that a wide range of programmers can use to target a wide range of applications.  We don't need to make it sound or feel more complicated, and those extra few minutes to reword a sentence here or there can help.

