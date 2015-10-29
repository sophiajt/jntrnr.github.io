---
title: Telling the truth in code, function signatures and transparency
---

When people talk about reasons to use a type system, there are always a couple usual suspects: error checking, documentation, and if you're an IDE person like me, tooling/editor tricks that use type information, like auto-complete.

But there's another aspect to the types of function signatures that's related to documentation but is perhaps stands on its own: transparency.

# What is transparency?

For this post, transparency helps us answer a few questions:

* What does this function require?
* What does this function return?
* What is this function going to do with what I give it?  
    * Can it modify it?
    * Can it delete it?
* Can this function fail?  If so, how?
* Does this function do anything besides use its value?  
    * ie) does this function "change the world" in some way?

# Why is transparency important?

A way most of us can relate to transparency is in the relationships you have in your life.  When you're not transparent with another person -- keeping all of your thoughts to yourself, never saying when you're angry, etc -- how does the other person ever know how to react to you? 

Transparency in code works similarly.  When a function is transparent, you know what you need to do when the function is called, and you know how to react when the function is called.  Let's look at a few examples.  In each example, we're going to talk about a function that takes in an employee name and writes that name to a file. 
Examples

## Python

{% highlight python %}
def writeToFile(name):
    # write out to the file
{% endhighlight %}

We'll start with the simplest form of transparency: minimal transparency.  Looking at what we have it's tempting to say we don't know anything at all, but that's not quite true.  We do know the function name and that it takes a single parameter.

Things we don't know:

* What kind of thing does it take?
* Can it modify the thing we give it?
* Can it delete what we give it?
* Does it return anything?
* Does it throw any exceptions?
* Does it have any permanent effect on our system
    * eg) send files over the network, add to a database, launch the missiles?

Without this knowledge, we resort to aggressively testing, commenting, and using naming and coding conventions to get the idea across.

## C

{% highlight c %}
int writeToFile(char *name) { 
    /* write out to the file */
}
{% endhighlight %}

Moving into C, we start to see a bit more information popping out.  We know that it takes in a ```char*``` for the name.  Since this also isn't ```const```, we know that it's allowed to change the contents of the string we pass in.  We also know it returns an ```int```, likely for error codes.   

We are starting to see into what the function can do, but there's still quite a bit we don't know:

* Can this function change our system?
* Can the ```name``` value be deleted? 
* Can the function raise a signal (a C-flavored way of exceptions/events)?  
* And what about those error codes?  How do we know the ```int``` is for error codes, and not returning the length of data written? If it is for error codes, how do we know which ones it supports?

We've improved some, but we still lean heavily on conventions, code comments, and testing to ensure that this function behaves correctly.  As an aside: one interesting point of C is that it doesn't have exceptions, like many languages, so you're almost encouraged to have a more value-based programming style, though it lacks many common features that help fill out the value-based story (pattern matching, algebraic data types, etc).

## Rust

{% highlight rust %}
fn writeToFile (name: &str) -> Result<()> {
  // write out to the file
}
{% endhighlight %}

Above is a simple Rust version.  If you've never seen Rust before, let's quickly break down it says:

* The ```fn``` keyword creates our function
* Next, we give the function a name: writeToFile
* Our function gets one argument called 'name' which has the type ```&str```
  * ```&str``` is a type in two parts: 
    * ```str``` is the simple string type
    * ```&``` meaning that this value is being borrowed and is owned by someone else (aka you're not allowed to delete it)
* Finally, we return something of type ```Result<()>```
  * ```Result<()>``` is the Rust way of saying I can return either:
    * a) successfully, without a value (the () above)
    * b) an error

The Rust example tells a bit more of the story than the C version.  In Rust, everything is immutable by default, so we know from looking at the signature that what we pass in can't be changed.  It also won't be deleted, because what is passed is 'borrowed' rather than us giving ownership over to this function.  Rust is similar to C in that we don't have exceptions and instead use return values to denote success or failure (read more in Rust Error Handling).  Rust encodes the success or failure in the return value, requiring the caller to code defensively around possible errors.  This has the effect of encouraging code to be more robust by handling both success and failure where they can occur.

An improvement over where we were in C, but we still have a lingering question: does this function have any permanent effect on the system?  To be able to answer that question, we go one example deeper down the rabbit hole.

## Haskell

{% highlight haskell %}
writeToFile :: String -> IO ()
writeToFile name = do 
{% endhighlight %}

Since this might be unfamiliar let's take a second to breakdown what we're seeing:

* We start by declaring the type (which isn't terribly dissimilar from a forward declaration in C)
* The type has two parts:
  * We take one argument that has a type String
  * We return an ```IO ()``` [see below]
* After the type, we write out the actual implementation of the function
* Just as in Rust, Haskell is immutable by default.  When we say we pass in a String, we're also implying that this string can't be changed or deleted.  

Also similar to Rust, in Haskell there are no exceptions, so you must handle all possible values when you call the function.  By saying ```IO ()```, this might at first seem similar to the Result<()>.  In some sense, it is, in that IO carries along any possible errors.  

It also goes a bit further.  In essence, it says: "I'm going to go change the world, and when I come back you need to carry that truth on."  In Haskell, if you call a function that performs I/O like our writeToFile function, you must also carry on that fact in your own return type.  You have to be fully transparent all the way through the call stack of who is changing the world, even indirectly.  This lets you clearly document who is modifying the world and who isn't.

# Dark side of transparency

But, it can't be all roses, right?  Let's look at what I see as the dark side of transparency: encapsulation (or its lack thereof).

As good software engineers, we know an important piece of building a system is having good encapsulation.  Any module or object worth its weight exposes only necessary functionality to the world and hides the rest.  This encourages loose coupling, so that components don't depend on each others' implementation details.

In a way, transparency in the type runs counter to information hiding in that it encourages more coupling in the code.  Some of this is natural.  For example, in the C code, we have to remember what the return values are.  If we add a new one, we may have to change the code handling the call as well.  Likewise, in Python, if we change the types of parameters we allow, we may have to go update the callers.

On the other hand, knowing that a function may fail may be sufficient.  In the Rust case, code that handles the call may need to be updated if you change how the function fails, even if it only cares about if it fails.

Likewise, in Haskell you may want to drop some probe into your system to catch the result of a function while you're debugging.  Such changes can have a ripple effect in your codebase as this probe may change a series of function calls to reflect that I/O has been performed.  (Not withstanding workarounds with fun names like ```unsafePerformIO```)

# All a balance

Types are there to help.  Leaning on them not enough means encoding more information in your tests and code comments.  Leaning on them too much may mean tightly coupling parts of your system.  What the right balance is may change from project to project, or developer to developer.  

Just as in our relationships with other people, it makes sense to not tell everyone we meet everything about us.  It's a balance.  Finding the right amount of transparency that encourages flexible, resilient systems is an art.

In the last few weeks, in playing with Rust more I've grown to appreciate its pragmatic set of trade-offs.  Yes, you don't get to know if the system was changed, but if something can impact the rest of the program, you're encouraged to handle it.  Not that there aren't workarounds or ways to easily opt-out, but just as in life, it's all a balance. 
