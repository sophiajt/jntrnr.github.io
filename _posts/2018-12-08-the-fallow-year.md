---
title: The Fallow Year, my Rust2019 post
---

__Definition:__ _fallow: (of agricultural land) Ploughed but left unseeded for more than one planting season._

Rust is an amazing project. It's unlike anything I've ever seen. Not only are we seeing an ever-growing number of big name users of Rust, we also continue to see leaps in productivity and functionality. Yesterday, [the Rust 2018 edition shipped](https://blog.rust-lang.org/2018/12/06/Rust-1.31-and-rust-2018.html). The culmination of three years of work, it shows off what the community is capable of: new features, backwards compatibility, new ergonomics without sacrificing performance, and the list goes on and on.

Which is why it might comes as a little bit of a surprise that I suggest we let the field rest for the year. More specifically, that we should let one field rest while we plant another.

Rather than focusing on new designs, I suggest we turn our attention outside of RFCs to other areas of Rust.  

# Examples

**The compiler**: The compiler team has been understaffed while also being heavily taxed by the effort of keeping up with new designs. I think they've done a stellar job with the resources they have, but they need help -- and lots of it. RFCs have gone unimplemented simply because not enough people are contributing. Compile times need work. The compiler team is generally too busy to implement core functionality that it hoped to get done this year, like compiler-drive IDE completions.

Let's clear through the backlog of the growing list of accepted RFCs. Let's take the time to do the deep refactoring that we haven't had time to do because we were stretched too thin. Let's give ourselves space to experiment with new ways of doing things. Let's get the compiler ready to serve both build and IDE scenarios.

**RLS and related tools**: We've had big success with projects like rustfmt, and it would be great to mirror that success in the Rust Language Server, or RLS, which powers Rust's IDE support. Unfortunately, the current RLS has to rely on many hacks to work with what is available from the compiler. By freeing up the compiler team, we also give space to the dev tools team to collaborate and finish the RLS, removing the need for hacks that cause incomplete functionality and unpredictabilty. 

**The community**: That's right, the community needs a pause too. Everyone loves new features, but we need a bit of time to let all the new ones sink in before we start designing more. What design patterns will arise in practice? What are the best ways to work with async code at scale? On and on. Getting all this experience will let us see more clearly where to design next.

**Documentation, videos, and other materials:** Along with the growing knowledge the community is getting by more using the current Rust more deeply, we can also translate this knowledge into more documentation. The need is strong for mid-level Rust developer documentation. What are the Rust design patterns? How does one get the most benefit from Rust quickly? How does one translate C++ knowedge to Rust? And so on.  

**Libraries:** We can use the knowledge we have to create new and better libraries. Existing libraries could continue to mature and adopt Rust 2018 idioms. Areas like GUI development need some clear direction and focus.

**Outreach:** We've worked on some outreach to companies who might be interested in Rust, but there's more we can do here. New whitepapers, new documentation about how to use Rust in specific sub-fields, like finance, data mining, machine learning, etc. 

# The list

As it turns out, these aren't my ideas. They're yours.  

[This year's Rust survey](https://blog.rust-lang.org/2018/11/27/Rust-survey-2018.html) results highlighted these areas of improvement (sorted by number of comments):

* the need for better library support
* a more improved IDE experience
* the need for broader adoption of Rust generally
* a richer ecosystem of tools and support
* an improved learning curve
* the need for important language features and crates to be stable and supported
* support for async programming
* support for GUI development
* better documentation
* improved compile times

You'll notice that only one of these, "support for async programming", implies adding new functionality to Rust itself. Rather than asking for new features, users wanted better documentation, tools, library support, IDE support, learning materials, and overall stability. 

# The idea

Let's finish what we started. As much as is possible and practical this year, let's set aside new designs and ship what we've already designed. Let's tackle challenges we haven't had time to give our full attention.

In short, let's grow Rust where it needs it most.

