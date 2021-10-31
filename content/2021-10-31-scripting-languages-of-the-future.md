+++
title = "Scripting languages of the future"
[taxonomies]
tags = [ "languages", "scripting" ]
+++

I've been thinking recently about what the future for scripting languages could look like. We've had a good run with Python, JavaScript, and Ruby for the last few decades. I wondered, what might the next few decades look like.

What is the scripting language of the future? How should it work? Here are some thoughts.

## Scalability

Working on [TypeScript](https://devblogs.microsoft.com/typescript/announcing-typescript-0-9/), something that became a constant refrain for us was that "TypeScript is JavaScript that scales".

I might be a bit biased, but this *is* actually incredibly valuable. If your team invests time into writing an app in a scripting language, there shouldn't be an impedance mismatch that grows as the app gets larger. For JavaScript, we'd watch teams grow their apps only to have to increase testing and onboarding training to prevent bugs from creeping in. And yet, they still crept in.

TypeScript gave tools a chances to catch these errors earlier, even in [well-tested codebases](https://devblogs.microsoft.com/bharry/typescript-a-real-world-story-of-adoption-in-tfs/). Back in 2012, this was something viewed with a healthy dose of skepticism, and today [over third of professional developers use TypeScript](https://insights.stackoverflow.com/survey/2021#most-popular-technologies-language-prof).

What TypeScript showed is that you could join together the idea of a flexible lightweight (and optional!) type system onto an existing programming language, and do so successfully.

The question then is - what if you created a programming language from the start to have this kind of support? No need for transpilers, while also having stronger engine integration and maybe even better runtime errors?

It's a tempting mix of possibilities and one well worth being a part of the scripting language of the future.

## Tune-ability

I've known multiple people who were hired to rewrite a piece of infrastructure originally written in a scripting language. Just like scripts need to be able to scale to larger sizes, scripts need to be tune-able. The choice to go dynamic shouldn't be a hard stop on real performance tuning and maintenance.

The work on numpy, scipy, pandas and others for Python is one possible blueprint. Instead of having to leave your language of choice, powerful libraries build up around it to support the high performance needs of different disciplines.

There will likely be some resistance to this point, as there seems to be just an inevitable trade-off between performance and the flexibility that a scripting language provides. In all honesty, this feels like an older, and perhaps outdated, way of looking at this trade-off.

For years, JavaScript developers have looked to techniques like [monomorphism](https://medium.com/wolfram-developers/performance-through-elegant-javascript-15b98f0904de) to tune the performance of JavaScript code. They're structuring code in a way that's more "VM friendly". The VM has to do less work to understand the types and structures in the code, and as a result guesses correctly the first time when doing runtime code generation.

The ability to tune code can come from other sources, too. Type decorations is one tempting direction to mention, though so far no established languages (that I know of) have used the technique to general type hints to speed up code. Unless type hints are used across hot paths, and with a sound type system, the transformations risks actually slowing performance.

There is, however, a large exception. Dataframe systems like pandas structure operations across arrays of known types. While the setup code is all in Python, the code that actually runs is happily in systems-level tuned native libraries. In a way, Python is merely orchestrating these operations.

Future languages could take advantage of this pattern in other ways. The associated standard libraries of these scripting languages could encourage more orchestration and less manual processing in the language itself.

## Easy parallelism

On a related point to the one above: future languages need easy parallelism. By now, we've heard it a million times. The "free lunch is over", tomorrow's CPUs will have a zillion cores and will run at 1hz. Okay, maybe not quite that bad, but still the writing is very clearly on the wall and very much already happening. It's not far-fetched to imagine that today's 8-16 core machines over the next few decades grow to be dozens or even hundreds of cores. Any language that hopes to have a strong footing, even scripting languages, need to be able to handle this.

This isn't without its own challenges. As has been quotes a few times recently, Matz - the creator of Ruby - famously said ["I regret adding threads"](https://betterprogramming.pub/ruby-has-its-own-2020-new-years-resolution-77b801dfaacc). Having now used a lot of different kinds of parallelism systems in a lot of languages, I can easily see why. Traditional threading, and things like it, honestly feel like goto to me. Just like goto, it's easy to write code that's hard to read and, over time, hard to maintain.

Luckily, traditional threads aren't the only game in town. [Work-stealing task pools](https://github.com/rayon-rs/rayon) like popular libraries in Rust use are very fast and can be built to work with other abstractions like parallel iteration and reduction operations.

Making abstractions like this part of the language gives more opportunity to use more of the machine across all scripts written in that language. All this without, we hope, running afoul of the design choices that make language creators mourn years later.

## IDE support

This might sound completely alien to some folks, but the future for scripting languages includes strong IDE support. Gone are the days of most scripting being written in a simple text editor. Today, [more than 86% of JavaScript is developed in an IDE](https://2020.stateofjs.com/en-US/other-tools/#text_editors).

Future languages are competing against established languages which have tooling and IDE support polished over many years. Any new language that wants to compete against that will have to swing for the fences.

At a minimum, support for the [language server protocol](https://microsoft.github.io/language-server-protocol/) these days in a must. Its broad support across editors gives languages much more leverage than ever before to get in front of developers.

## Looking ahead

There's definitely a large space to explore for scripting languages of the future. The points I list above are important, as are additional points that we'll learn as we go. Perhaps we'll be able to bring the performance that more modern JS engines enjoy to a broader range of languages through an effort similar to what LLVM did for lower-level languages.

I, for one, am looking forward to this future and what future languages it will bring.