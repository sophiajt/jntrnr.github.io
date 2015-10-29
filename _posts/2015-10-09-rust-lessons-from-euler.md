---
title: Lessons from the first 12 Euler problems in Rust
---

**!! Spoiler Alert !!**

*I'm going to be talking about solutions to select problems from the first 12 Euler problems.  If you haven't solved these for yourself yet, I highly recommend taking the time to solve them on your own first.*

# Background

Before I went to Microsoft for a [stint in JavaScript](http://www.typescriptlang.org/), I did my fair share of systems programming in C++.  I've built compilers, head-tracking software, signal processing software, and emulators and even did some of my first programming language projects in it and [for it](http://chaiscript.com/).

So when word got out four years ago about a new systems language called [Rust](https://www.rust-lang.org/), I was intrigued.  I mean, it was *new* and it was called *rust*.  Obviously there was some cheekiness involved :)

While I've dabbled with it before, this week was the first time I really gave it some dedicated time to learn it.  **TL;DR:** hey, this is kinda neat!

In this post I talk through solving some of the Euler problems in Rust, both de-rusting my programming skills and learning what this Rust thing was all about.

If you're interested, you can grab the [source code to my solutions](https://github.com/jonathandturner/rustnewbie).

# Problem 1

"sum of all multiples of 3 or 5 below 1000"

{% highlight rust linenos %}
fn main() {
    let mut answer = 0;
 
    for x in 0..1000 {
        if x % 3 == 0 {
            answer += x;
        }
        else if x % 5 == 0 {
            answer += x;
        }
    }
 
    println!("{}", answer);
}
{% endhighlight %}
