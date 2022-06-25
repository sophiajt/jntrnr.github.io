+++
title = "Will parallelism go implicit?"
[taxonomies]
tags = [ "languages" ]
+++

I've been spending quite a bit of time recently with [Implicit Parallel Programming in pH](https://www.amazon.com/Implicit-Parallel-Programming-Rishiyur-Nikhil/dp/1558606440), which luckily the school library had a copy of.  It's quite a good textbook to introduce programmers to functional programming, perhaps even one of the better ones (next to the original [Introduction to Functional Programming](http://www.amazon.com/Introduction-Functional-Programming-International-Computing/dp/0134841891) by Bird, Wadler that I mentioned in an earlier blog post).  What turned me on to it, though, wasn't the treatment of functional programming - it was its far more audacious titular goal.

Which leads to the obvious question. *What is implicit parallelism?*

If you bring up the term around PL geeks, you might get a few questioning looks and some comments that hint at automatic loop parallelization.  Even more confusingly, this second kind of parallelism is sometimes referred to as automatic parallelism.  It refers to taking sequential codes and reasoning about where parallelism could occur without changing any results.  Implicit parallelism is a more holistic approach - what if the only sequentiality in your program comes from the data dependencies?  Then the rest, it stands to reason, can be done in parallel.

I mentioned this as an audacious effort.  The first and foremost reason, to my engineering mind, is *how in the world will you make this efficient?*  I can just imagine the compiler identifying tens, perhaps even hundreds, of tiny tasks that can be done in parallel at any one timeslice.   How do we schedule these tasks in such a way we don't pay a high overhead for synchronization and task switching?

I haven't dug into the implementations of the languages yet to see how they address this issue, but as I was brainstorming this morning it hit me.  Implicit parallelism isn't so different from implicit (or automatic) memory management.  When garbage collectors first came out, they weren't the lean mean things they are today.  That's part of the advantage of having decades to polish an approach, and now fewer and fewer applications programmers use languages that don't have gc support.  It saves more headaches than it causes, and that's generally a winning combination.

Not always, of course.  The HPC community has regularly shunned gc in their languages, where high performance codes are hand-tuned Fortran or C with manual memory management.  Still, I would argue that HPC is boutique, a cottage industry focused on maximal efficiency.

Most everyday programmers don't need these levels of efficiency, and I imagine, will instead opt for automatic support for parallelism.  This saves them the headache of separately tuning for the same application to run comfortably on either 16 or 128 cores.  From the consumer's point of view, this is also a win.  Buying that new 256 core machine actually will make these apps run faster.

**Some final thoughts**

Perhaps it's time to focus on introducing more forms of automatic and implicit parallelism into our programming languages and runtime systems.  Our early attempts, taking from the experiments of the past could kick it off, and once industrial interest has turned to it, I'd expect steady progress in efficiency (just like we've seen in gc).

Is there some practical limit to the amount of parallelism in code?  There's a tension between [Amdahl's law](http://en.wikipedia.org/wiki/Amdahl%27s_law), that maximum speedup through parallelism is limited by the amount of sequential code, and [Gustafson's law](http://en.wikipedia.org/wiki/Gustafson%27s_Law), which says that the increased computing power lets us tackle larger problems.  To continue to gain we may have to change how we approach problems, or even reframe the problems themselves.