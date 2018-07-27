---
title: A Snapshot of Rust's Popularity in July 2018
---

Talking about a language's popularity is traditionally a tricky topic. How do you measure popularity? How do you compare one language to another when they're focused on different styles and different audiences?  So, rather than having one or two charts, I'm going to look at a number of "slices" into Rust's growth to see it front different angles.

There are also *many* ways we could measure popularity. For example, looking at the growth of StackOverflow, internal forums, Twitter traffic, Google Trends, or the various language popularity reports.  Each of these measures has its own strengths and trade-offs.

For our purposes in this post, we're going to focus on a set of data points that should help show growth of *usage* rather than growth of *buzz* (that is, the amount people talking are about Rust). If people are using a language to do real work, that's the best indicator of a language's success.

The first of place we'll look for Rust usage is in open source, and with that, let's look at GitHub.

# Growth on GitHub

GitHub offers a few ways to measure how well a particular language is growing.  GitHub itself uses the number of pull requests in a given language, which it uses in the [GitHub Octoverse](https://octoverse.github.com).

If we were to run this query for 2018 so far, the top fifteen languages (by number of PRs) would be:

1. JavaScript: 1736476
2. Python: 804790
3. Java: 703649
4. Ruby: 560430
5. PHP: 359040
6. C++: 319324
7. TypeScript: 311229
8. Go: 258131
9. C#: 246513
10. CSS: 236795
11. Shell: 168301
12. C: 160889
13. Swift: 67664
14. Scala: 67188
15. Rust: 52936

Rust comes in at 15th place as part of the group of Swift/Scala/Rust.  This is pretty good company to be in. All of the languages on this list are known for doing heavy lifting in real applications, rather than being hobby or boutique languages.

Being 15th is also significant because, if the Octoverse were held today, Rust would make their top languages list for the first time.

Moving on, one gotcha of focusing on PRs is that it lets a relatively small number of highly successful projects skew the results. The popularity of the projects drives high numbers of PRs, and as a result the language looks far better.  To compensate for this, let's also look at "active projects". For our purposes, let's define "active projects" as the number of projects that a) are larger than a "hello world", b) have at least 1 star, and c) have been worked on in the last 30 days:

1. JavaScript: 56437
2. Python: 37554
3. Java: 27912
4. C++: 16787
5. PHP: 16451
6. Shell: 12245
7. C#: 11503
8. C: 10662
9. Go: 8816
10. CSS: 8781
11. TypeScript: 8447
12. Ruby: 7518
13. Swift: 3828
14. Objective-C: 3054
15. Rust: 2604
16. Kotlin: 2343
17. Scala: 2072

Another respectable showing for Rust. Again, we see it as part of the languages for doing serious work.

The GitHub numbers tell one story, but do we see the same story if we look in other places? With this kind of popularity, we'd expect to see visible use of Rust in industry.

# Rust's Commercial Users

Rust has seen a surprising amount of growth in commercial usage over the past year alone. Here's a partial list of names people may know:

**Amazon** - Building [tools in Rust](https://github.com/amzn/askalono).

**Atlassian** (makers of Jira) - Using [Rust in the backend](https://github.com/rust-lang/rust-www/pull/922).

**Dropbox** - Using Rust in [both the frontend and backend](https://air.mozilla.org/rust-meetup-may-2017/).

**Facebook** - Tools for [source control](https://twitter.com/Sunjay03/status/1019782490800603136).

**Google** - As part of the [Fuchsia project](https://github.com/fuchsia-mirror?utf8=✓&q=&type=&language=rust).

**Microsoft** - Using Rust in part of their [new Azure IoT work](https://twitter.com/maxgortman/status/1012011425353461760).

**npm** - Using Rust in some of the [npm core services](https://github.com/rust-lang/rust-www/pull/634).

**Red Hat** - Creating a [new storage system](https://github.com/stratis-storage)

**Reddit** - Using Rust in its [comment processing](https://www.reddit.com/r/rust/comments/7utj4t/reddit_is_hiring_a_senior_rust_engineer/)

**Twitter** - As part of the [build team support for Twitter](https://twitter.com/stuhood/status/978410393944047617?s=19).

You can see even more on the [Friends of Rust](https://www.rust-lang.org/en-US/friends.html) that include other familiar names like Baidu, Wire, Mozilla, Samsung, Cloudflare, Chef, Canonical, and more.

Across the industry, software companies are beginning to adopt Rust at some level in their organization.  As expected, these may be tentative at first, focusing on Rust for build tools, for backend processing, or targeting a specific platform like IoT.  Early adopters like Dropbox and Canonical have now grown beyond initial investments to using Rust in an increasing amount of their codebase.

All told, it's still early days for Rust. Large companies take time to try new technology before they adopt it more broadly. We're seeing Rust in that part of the adoption curve, and it's doing well here.  Languages -- like Go and TypeScript -- that saw explosive growth in recent years also saw similar patterns at this part of their lifecycle.

Because of Rust's closeness with C++, we should also look at industries that are traditionally strongly populated by C++ developers. One such industry is game development.

# Zooming into Games

Despite the game industry's use of established development technologies, Rust is beginning to make in-roads with game developers.  Here's a list of usage of Rust in some familiar names among game developers:

**Chucklefish Games** - Using Rust in [multiple upcoming games](https://www.rust-lang.org/pdfs/Rust-Chucklefish-Whitepaper.pdf).

**Electronic Arts** - Using Rust in [SEED](https://twitter.com/ZigguratVertigo/status/1021562281056980993).

**Frostbite Engine** - Using Rust in [backend processing](https://twitter.com/Ca1ne/status/983612241235804160).

**Ready at Dawn Studios** - [All new development](https://twitter.com/AndreaPessino/status/1021532074153394176) will be in Rust. 

**Unity** - Using Rust in [data engineering](https://twitter.com/bltroutwine/status/1002234680949719040).

We're also seeing [other indie developers](https://twitter.com/SergiusIW/status/1021236971786694656) beginning to use it for their titles. 

Just as we saw in the "big name" usage, we see a similar pattern with game developers. Initial usage is targeted, perhaps at backend or data-focused projects. After this, there's a path to developing core technology, and even whole games, in Rust.

# Jobs

When we ran the [2017 Rust survey](https://blog.rust-lang.org/2017/09/05/Rust-2017-Survey-Results.html), one of the big themes that came up for people not using Rust was around their companies not using it. The jobs just weren't there.

In 2018, with a growing numbers of companies and studios using Rust, has this situation improved?

In short, yes, though there's still a lot of room to grow. A quick look at LinkedIn's job list shows a number of Rust opportunities from the likes of Amazon as well as a number of companies involved in blockchain, security, science/healthcare, and more. Still, when compared to the volume of job offerings for languages like Go or TypeScript, there's still a gap in terms of availability.

Let's move on from Rust's use in companies to look at the growth of the community that supports Rust.

# Community and Governance

In times past, a language could survive on being used solely for commercial development. These days, however, a language's community forms its foundation. It offers a way to ramp up new users, support new projects, and set the tone for the project going forward.

![Map showing Rust being used in over 130 meetups around the world](/images/rust_meetups.png)
_Rust meetups now total over 130 around the world_

The in-person piece of the Rust community, the Rust meetups, have seen 30% growth in the last nine months and now total over 130 around the world.  We've likewise seen steady growth in the number of conferences that users can attend for more training and networking.

We've also seen some impressive growth in indicators of the online community as well since the [last time I reported on them](http://www.jonathanturner.org/2017/10/fun-facts-about-rust-growth.html). For example, the Rust reddit recently passed 40k users (up from the 29k of nine months ago), the GitHub repo is nearly at 30k stars (up from 24k), and the official Rust twitter has over 31k followers (up from 24k).

This enthusiasm has also translated into the teams that govern Rust itself. Combined, the governing teams have doubled in size in the last 9 months to over [100 positions across the various teams](https://www.rust-lang.org/en-US/team.html). The scope has also grown in tandem, with new teams that cover infrastructure, developer experiences, and dependency management.

# Conclusion

These are just some data points about Rust's use in industry and in the software development community. It's difficult to get a complete picture, and even more difficult to predict what will come next. That said, it does seem to shine a promising ray of light on where Rust is today.

Where will Rust go tomorrow?  That is anyone's guess. I, for one, will certainly be watching and cheering it along. 