+++
title = "Feature Bias, or Rethinking the Blub Paradox"
[taxonomies]
tags = [ "philosophy" ]
+++

I got a lot of great feedback on my previous post ["Rust and the Blub Paradox"](http://www.jonathanturner.org/2016/01/rust-and-blub-paradox.html), which inspired me to write a follow-up.

In its original formulation, the [Blub Paradox](http://www.paulgraham.com/avg.html) is something of a tool for people to feel smug about their favorite language.  I heard a lot of feedback that it feels condescending.  I hadn't realized that in my mind the Blub Paradox had morphed from its original form into something different.

In this post, I'd like to break down the Blub Paradox into to what I see as its fundamental observation: that we, as programmers, can have a form of unconscious bias.  I'll call this *Feature Bias* so that we don't confuse it with Paul's original idea.

# There are no universally better languages

There is no way to universally sort programming languages on a continuum that is monotonically increasing in awesomeness.  It's impossible.  Not only are languages too complex to have any kind of universal ranking, they are tools and as tools the value is left to the tool user.  It's like asking "is pencil better than a paint brush?"  It all depends on what you're doing and what you're more comfortable with.

The types of languages that Paul criticizes in Beating the Averages aren't universally worse than his favorite languages.  Taking one example, everyone loves to put Java down, but have you ever watched Notch (the creator of Minecraft) [code in it](http://www.twitch.tv/notch/b/302823358)?  He seems perfectly content to create with his tool of choice, and he's certainly capable of doing so.

# Feature Bias happens locally per feature

Rather than trying to sort languages universally, a useful version of the Blub Paradox is a local phenomenon.  It happens on a feature by feature level.

If a programmer accustomed to working with a feature sees a language that lacks the feature, they can also see the use cases where programmers in that other language might benefit from its use.

If a programmer who doesn't use a feature looks at a language with this feature, until they've spent time to understand it, they may see the feature as unnecessary.

# Feature Bias happens to most programmers, even the best

As a language PM at Microsoft sitting on a few programming language committees, I saw Feature Bias happen to people from junior developers to famous language designers with decades of industry experience.  The first time a developer sees a new feature, regardless of skill level, there's a chance they may experience the bias.

True story: the C# feature LINQ had to 'bake' for at least a year with the design committee before enough members saw its value for it to become part of the language.  All the while the champions behind it were trying to educate and get other committee members to give it a chance.

# Features don't have universal value

Once the developer puts in the time to understand the feature, they *still* may not like it.  That's not the paradox/bias anymore.  They've put the time in to understand the trade-offs, and they don't feel the trade-offs are worth it.  That's a totally acceptable response.

# The root of the bias

To summarize, just like other unconscious biases, this bias is pointing out something that's common for people to experience and they may not realize they're experiencing it.  

A lot of programmers I know, including myself on a good day, use an understanding of the bias as a way to train ourselves to take time with each feature and work through the knee-jerk "oh this is crap!" reaction.  I definitely still fall victim to it.  Just as with other biases, you have to keep working at it.

In seeing the responses from people, it's unfortunate that the original Blub Paradox has been used as a club to beat people with.  It makes it that much harder to see where it can be helpful.  In my experience, it can shine a light on an area that helps us become better programmers and teachers.  

