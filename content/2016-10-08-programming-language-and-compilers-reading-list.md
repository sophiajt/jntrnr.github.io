+++
title = "Programming Language and Compilers Reading List"
[taxonomies]
tags = [ "programming languages" ]
+++

This week, the talented [Julien Fitzpatrick](https://twitter.com/_jbfitz) (btw, you should check out their [RustConf talk](https://www.youtube.com/watch?v=Ce6ppwgF4SA) if you haven't already) asked what a good list would be for people who are interested in programming languages and compilers.  I took it as a good excuse to write a blog post with some of my recommendations.  I'm going to attempt to put together a list that doesn't require a lot of background information, but as these things go, sometimes you'll have to re-read portions of books to get their benefit.  Also, this isn't going to be an exhaustive list.  I have never read through an exhaustive reading list before.  Instead, this is definitely biased towards writing real code, getting things working, and learning by doing.  Without further ado, let's get started!

## Getting started

To kick things off, the first thing to do is to remove any fears we have about thinking that compilers are big complicated things that are only possible by the most heroic of efforts.  Yes, some compilers can be quite complex, but that's true of many kinds of industrial software.  At its core, though, the compiler just does a few relatively simple things.  So let's demystify those things.

First off, let's make a small compiler.  [@thejameskyle](https://twitter.com/thejameskyle) has written a [cool tutorial](https://github.com/thejameskyle/the-super-tiny-compiler/blob/master/the-super-tiny-compiler.js) that helps take you through the steps of creating a simple compiler.  Rather than chewing your way through dusty tome after dusty tome before you write your first line of code, my suggestion would be to always have a code editor open at each step of the way.  Play with the ideas as you learn them.

Alright, next, after we've played around with our little compiler and made a few changes and see how our changes affect the output, it's time to start building up our knowledge.  Fundamentally, how a programming language works can be described in abstractions that build on other abstractions.  If we peel these abstractions away, we're left with a pretty simple foundation.

One of the compiler books I liked that help build up these abstractions from a simple base is [Jeremy Siek](http://wphomes.soic.indiana.edu/jsiek/)'s [compiler course](http://ecee.colorado.edu/ecen4553/fall12/notes.pdf).  The notes are laid out by chapter, and each chapter introduces a new concept that you'll add to the compiler.  I enjoyed the hands-on approach, since you can think about how each feature affects each part of the compiler.  It definitely gave me a "oh my god, I can't believe this works!" moments.  You may also want to adjust to fit your particular style. For example, if something like the assembly piece feels awkward, you could instead output code to a language you're more familiar with, and work through the other steps of the compiler.

On the programming language side, probably one of the better survey books I've seen is [Programming Language Pragmatics](https://www.amazon.com/Programming-Language-Pragmatics-Fourth-Michael/dp/0124104096).  I'd probably pair it with something like [Masterminds of Programming](https://www.amazon.com/Masterminds-Programming-Conversations-Creators-Languages/dp/0596515170) or [Coders at Work](https://www.amazon.com/Coders-Work-Reflections-Craft-Programming/dp/1430219483).  These two books are interviews with programming language creators.  Creating programming languages is definitely equal parts art and engineering, and bouncing between the creators talking about the art and seeing how the engineering works in practice might help see parts of the picture a bit clearer.

## Working some problems

I used to hate doing homework.  I mean, who didn't?  But getting a little older, I realize just how useful doing some of the problems in the book can be.  That's the time where you actually are solving little puzzles.  And, if you're lucky, it's also a great chance to have one of those "a ha!" moments that unlock the next set of secrets.

If you're like I was at the time, you've heard of "functional programming" about a million times but weren't sure why anyone should care all that much.  It wasn't really until I was working through [Introduction to Functional Programming by Bird/Wadler](https://www.amazon.com/Introduction-Functional-Programming-International-Computing/dp/0134841891) that things started to click.  The problems in this book are great.  They're juicy enough that you might have to work at them a little, but you come away with a much better sense of the concepts in each chapter.

Another book I enjoyed the problems in is the [Dragon book](https://www.amazon.com/Compilers-Principles-Techniques-Tools-2nd/dp/B009TGD06W).  The problem with the Dragon book, though, is that it definitely feels like a stuffy college textbook. That said, it's worth a skim.  My favorite part about it are the problems in the earlier chapters.  Working through those gave me a bit more confidence I understood the concepts.

## Getting more modern

Perhaps another problem with the well-known dragon book is that the dragon is a bit long in the tooth these days.  Modern techniques have moved on a bit from where they were a few decades ago.

One thing to check out to see a bit more of how modern compilers work is the [LLVM chapter](http://www.aosabook.org/en/llvm.html) of Architecture of Open Source Applications (which may also be a fun read for someone interested in how things work generally).

I don't know of a good book on this topic, but there's a good video that goes into some of the ways that [modern compilers work differently](https://channel9.msdn.com/Blogs/Seth-Juarez/Anders-Hejlsberg-on-Modern-Compiler-Construction) than those in the dragon book.

# For the brave

When you've worked your way to this point and are feeling pretty confident, it's time to scale to the dizzying heights.  To get there, you'll inevitably come across [Types and Programming Languages](https://www.amazon.com/Types-Programming-Languages-MIT-Press/dp/B00AJXZ5JE) (which you might hear pronounced "tapple").  I'm not going to mince words, this book is brutal.  It took months and months of working on it to crack it.  I managed to finally get my head around it by methodically using a hands-on approach.  I went [chapter by chapter](https://github.com/jntrnr/Pierce-and-Types) and implemented each new language concept the author introduced.  It took a little while, but it managed to turn the greek symbols into something a bit more concrete.

At this point you have quite the toolbox.  You've written a few small compilers, you've seen how language features work and how they interact with each other, you can read academic papers (if need be), and maybe you've even made a programming language of your own.  Now you can branch out to almost anywhere.
