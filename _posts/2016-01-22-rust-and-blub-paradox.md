---
title: Rust and the Blub Paradox
---

A few weeks ago, I read an [analysis of Rust, D, and Go](https://www.quora.com/Which-language-has-the-brightest-future-in-replacement-of-C-between-D-Go-and-Rust-And-Why) by Andrei Alexandrescu.  Andrei, a [respected member of the C++ community](http://www.amazon.com/Modern-Design-Generic-Programming-Patterns/dp/0201704315) and a core developer of the [D programming language](http://dlang.org/), took a stab at Rust at the end of his writeup with what seems like a pretty astute observation:

"Reading any amount of Rust code evokes the joke "friends don't let friends skip leg day" and the comic imagery of men with hulky torsos resting on skinny legs. Rust puts safe, precise memory management front and center of everything. Unfortunately, that's seldom the problem domain, which means a large fraction of the thinking and coding are dedicated to essentially a clerical job"

Having met Andrei and seen a few of his talks, I know that he likes to poke fun.  Still, let's take the bait.  Is his observation funny because it's a funny image or is it funny because there's an underlying truth to what he's saying?

# Blub paradox

Whenever I think about how useful certain language features are, I almost always come back to Paul Graham's essay ["Beating the Averages"](http://www.paulgraham.com/avg.html).  In it, he observes an interesting phenomenon among programmers he calls "the Blub Paradox".  To recap for those who haven't heard of it, the paradox goes something like this: let's say there's a programmer using a language called Blub.  Blub falls somewhere in the middle of the programming language pack in terms of capability.  It's not the lowest, most minimal language nor is it the most powerful.

When the Blub programmer looks "down" the spectrum of languages, they can easily see that those languages are less expressive than their preferred Blub language.  Unfortunately, when they look "up" the spectrum of languages, they can't tell they're looking up.  As Paul puts it:

"What he sees are merely weird languages. He probably considers them about equivalent in power to Blub, but with all this other hairy stuff thrown in as well. Blub is good enough for him, because he thinks in Blub."

When I read that years ago, I remember thinking "wow, that's pretty astute."  Little did I know that years later it would become firmly ingrained as part of how I thought about teaching programming as a programming language PM.

As a programming language PM at Microsoft, I worked on [TypeScript](http://www.typescriptlang.org), a typed version of JavaScript.  Without fail, if I was speaking to an audience of predominantly JavaScript developers, I would get many frowns as I tried to enumerate the advantages of adding a bit of type-checking to JavaScript.  Even if it was optional.  Even if I showed half a dozen powerful advantages.  As Paul notes, it just looked "weird".  To a JavaScript developer, TypeScript would often just seem like a language that was equally expressive with "hairy" stuff thrown in.

As I talked to more language PMs, and I met more and more people at conferences, I learned that Paul's observation was not only accurate, it was also surprisingly universal.  Most programmers will struggle when looking at a new language they don't use.  Often, they'll have allergic reactions to features that are foreign. It takes time to work with the features enough to understand why they aren't just unnecessary ornamenation gumming up the works.

In short, the Blub Paradox is something that as programmers we actively need to assume we're falling into and then work to get ourselves out of.

Let's do just that.  Let's look at a few stand-out Rust features and assume these features are weird and unnecessary.  From there, we'll see if we can work to un-Blub ourselves.

# Weird feature #1: Rust-style polymorphism

Let's make a Rust program that uses a bit of polymorphism to print out two different structs.  I'll show you the code first, then break it down.

{% highlight rust %}
use std::fmt;

struct Foo {
    x: i32
}

impl fmt::Display for Foo {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "(x: {})", self.x)
    }
}

struct Bar {
    x: i32,
    y: i32
}

impl fmt::Display for Bar {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "(x: {}, y: {})", self.x, self.y)
    }
}

fn print_me<T: fmt::Display>(obj : T) {
    println!("Value: {}", obj)
}

fn main() {
    let foo = Foo {x: 7};
    let bar = Bar {x: 5, y: 10};
    print_me(foo);
    print_me(bar);
}
{% endhighlight %}

What an eye-full!  There's polymorphism there, but it doesn't look anything like OO.  It's generic, and not only generic, but it has constraints.  And what's this ```impl``` stuff?

Let's break it into parts.  I'm creating two structs to hold our values.  I go a step further and make sure they implement something called ```fmt::Display```.  In C++, you might overload the << operator for ostream.  The end result is similar in Rust.  After I have this in place, I can call print functions and pass these structs directly.

With that, we're half way through the example.

Next, we have our ```print_me``` function.  This function is generic, and will accept anything so long as what is passed in can do ```fmt::Display```.  Luckily, we've already said how both structs can do just that.

The rest is straight-forward.  We create a few structs and pass them to ```print_me```.  

Phew... that seemed like a bit of work.  And really, that's how polymorphism starts in Rust.  It's all with generics.  

Now, let's switch to C++ for a minute.  Someone using C++, especially a beginner, might not go the generic route at first and might instead opt to use OO-style polymorphism:

{% highlight cpp %}

#include <iostream>

class Foo {
    public:
        int x;
        virtual void print();
};

class Bar: public Foo {
    public:
        int y;
        virtual void print();
};

void Foo::print() {
    std::cout << "x: " << this->x << '\n';
}

void Bar::print() {
    std::cout << "x: " << this->x << " y: " << this->y << '\n';
}

void print(Foo foo) {
    foo.print();
}

void print2(Foo &foo) {
    foo.print();
}

void print3(Foo *foo) {
    foo->print();
}

int main() {
    Bar bar;
    bar.x = 5;
    bar.y = 10;

    print(bar);
    print2(bar);
    print3(&bar);
}
{% endhighlight %}

Pretty easy, right?  Okay, here's a quick quiz for you.  What does the C++ code print?

If you guessed wrong, don't worry.  You're in good company.  

If you guessed right, congrats!  Now, think a minute about how much do you have to understand about C++ to know the answer.  From what I see, you need to understand the stack, how objects are copied, when they are copied, pointers, references, v-tables, and virtual dispatch. Just to do a few simple lines of OO.

When I first started learning C++, I had a pretty steep hill to climb.  Luckily, [my cousin](http://blog2.emptycrate.com/) is an expert C++ programmer and took me under-wing to show me the ropes.  Still, I made tons of beginner mistakes like the one above.  Why?  In part, the hill is steep because of the cognitive load of learning C++.

Some of that cognitive load is inherent in programming a computer at a system's level.  You need to understand the stack.  You need to understand how pointers work.  What C++ adds is another layer of complexity around knowing when a value isn't fully copied and when virtual dispatch is and isn't used -- all without any warning to the user if they do something that is "probably a bad idea(tm)".

This isn't to knock on C++.  Much of Rust is based on keeping the C++ philosophy of low-level efficient abstractions intact.  In fact, you can even write code that's [very similar to the Rust example](https://gist.github.com/jonathandturner/a182347b763398d8ea4f).

What Rust *does* do in this case is to separate inheritance from polymorphism and push you towards genericity from the start.  In doing so, it gets you thinking generically from day #1.  It might seem weird to separate inheritance from polymorphism if you're used to them being used together.  

That separation gives us one of our first Blub moments: what's the advantage of separating inheritance and polymorphism?  In fact, does Rust even have inheritance?

Believe it or not, at least as of Rust 1.5, Rust does not allow inheritance of structs.  Instead, we can grow the functionality of structs from outside them using a feature called 'traits'.  Traits allow you a way to add methods, require methods, and retrofit datastructures for existing systems.  They're also allowed to inherit, so one trait can extend another.

If you poke around in the Rust, you'll also notice something else.  We don't have the same issues we had with C++.  There's no worrying about what gets lost when the function is called or how virtual dispatch works.  In Rust, we're uniform regardless of the type.  Because of this, a whole class of beginner errors vanish.

# Weird feature #2: what, no exceptions?

While we're on the subject of things Rust doesn't have, its next weird feature is that it lacks exceptions.  Isn't that like going backwards?  How do we talk about errors that only occur sometimes?  How do we bubble out errors so we can handle them in one place?

Next you'll be saying I have to learn monads.

Actually... no, just kidding, you don't need to learn monads.  Rust's exception-less programming is a lot more straight-forward.  Here's an example of what this looks like in practice.  First, examples of what the functions look like:  

{% highlight rust %}
impl SystemTime {
  /// Returns the system time corresponding to "now".
  pub fn now() -> SystemTime;

  /// Returns an `Err` if `earlier` is later
  pub fn duration_from_earlier(&self, earlier: SystemTime) -> Result<Duration, SystemTimeError>;
}
{% endhighlight %}

Notice the ```now``` function returns a SystemTime and doesn't have any error cases, while ```duration_from_earlier``` does have a Result that can be either a Duration or the error SystemTimeError.  At a glance, you know a function's success and possible failure cases.

But all those failure cases are bound to stack up.  Who wants to see them all over the code?  Sure, it's good to do error checking, but the point of exceptions is to be able to handle them not only locally but also to let them bubble up and handle them in one place.

Rust lets you do that, too.

{% highlight rust %}
fn load_header(file: &mut File) -> Result<Header, io::Error> {
  Ok(Header { header_block: try!(file.read_u32()) })
}

fn load_metadata(file: &mut File) -> Result<Metadata, io::Error> {
  Ok(Metadata { metadata_block: try!(file.read_u32()) })
}

fn load_audio(file: &mut File) -> Result<Audio, io::Error> {
  let header = try!(load_header(file));
  let metadata = try!(load_metadata(file));
  Ok(Audio { header: header, metadata: metadata })
}
{% endhighlight %}

While it may not look like it at first, the above example uses the "bubbling" style of exceptions.  The trick is in a Rust macro called ```try!```.  What it does is pretty simple.  It will call a function for you.  If that call returns with a success value, it will hand that to you.  If it instead returns an error, ```try!``` will return from the containing function with that error.

The end result means that if if ```load_header``` ever has a problem when calling ```file.read_u32()```, that Error will be returned instead.  If it is, the same thing happens in ```load_audio```, meaning that it, too, will return the same Error.  And so on, until the calling function finally handles the error case.  

I wrote a [post about ```try!```](http://www.jonathanturner.org/2015/11/learning-to-try-things-in-rust.html), if you'd like to learn more.

# Weird feature #3: the borrow-checker

You know, it's funny.  Many people mention the borrow-checker as the first thing about Rust.  Often, they even rank it as *the* thing that sets Rust apart.  To Andrei, this is the "hulky torso" part of Rust.  To me, though, the borrow-checker is just another pass in the compiler.  Just like type-checking, it helps catch a level of bugs before they happen at runtime.  That's it.  It might seem hefty at first, but I'd argue that's just because it's different and learning to work with it grows a new muscle as a programmer, not unlike working with a new type system.

What kinds of bugs does the borrow-checker catch, you ask?

## Use-after-free

Ah yes, the classic problem of freeing memory and then trying to use it later.  Quite often, this is what makes programs crash with the dreaded "null pointer exception".

There are a good handful of C++ best practices to help avoid this (using RAII, preferring references or smart pointers over raw pointers, documenting the ownership in your API docs, etc), which is likely what Andrei referred to as "large fraction of the thinking and coding are dedicated to essentially a clerical job".  A team of highly trained C++ programmers would see avoiding use-after-free as largely clerical because that's about what it amounts to -- so long as you obey all the best practices, never cheat, and you always only grow the team with more experts.  

## Invalid iterator

Ever modify a container you're iterating over in C++ and then have a random crash later on?  I have.  If you managed to add or remove an item to the container that was enough to force memory reallocation, your iterator has become invalid.

I don't often make this error, but I still do from time to time.

## Data races

In Rust, data is either shared or mutable.  If it's mutable, it can't be shared, so you don't risk mutating from two threads in a race.  If it's shared, you can't mutate, so you can read to your heart's content from multiple threads.

If you're coming from C++, or any other language with a set of good parallel libraries, this is going to feel overly strict to say the least.  Luckily, this isn't the whole story, but instead the general basis that gives you a simple set of rules to build more abstractions on.  The rest of the story is still young and forming.  There are a growing number of [libraries focused on concurrency](http://areweconcurrentyet.com/) that you can follow if you're interested to learn more.

## Tracking ownership

This may seem a bit redundant, but it's actually something that C++ struggles with.  I alluded earlier to "documenting the ownership in your API docs".  The problem is that this information is in a comment rather than the code itself.

Here's a scenario: you're in C++ and you need to call a library that you didn't write.  Let's say it's a C-based library and takes a raw pointer.  Will it delete what you passed in?  Will it take ownership of it and put it into one of its data structures?  Maybe you're calling into a scripting engine like Ruby.  Then who owns it?  

Rather than reading through docs, Rust lets you verify that your assumptions match reality when it tracks how you are using the API with the borrow-checker.

## And more

Borrow checking helps other classes of errors.  For example, it helps improve local reasoning when you're writing a function because you can safely assume that any mutable values coming in are disjoint and won't interact with each other when you change them.

This incidentally also opens the door for additional optimizations because the compiler can ensure that values can never be both aliased and mutable, something that is difficult to do in C-based languages with alias analysis.

There's more good info about the borrow checker and classes of errors it prevents in ["You can't spell trust without Rust"](https://twitter.com/rustlang/status/689823725219377152) if you're interested.

# Weird feature #4: rules are meant to be broken

I'd argue that one of Rust's strongest suits is that it's pragmatic.  Most of Rust's stringent checks can be circumvented using features like ```unsafe```, ```mem::transmute```, and more.  That's right, don't like the borrow-checker for your use case?  No problem, turn it off.

Rust is made to be able to express anything you can do in a C-like systems language.  The advantage of Rust is that it's much easier to start with safe defaults that can be pushed out of the way when needed.  It's much more difficult to build safety on top of unsafe language features.

While Rust lets you pick and choose, it biases towards not letting you shoot your foot off. 

# Did Rust miss leg day?

Speaking of feet, did Rust miss leg day after all?  Is it lopsided?  Has it been focused on the wrong things?

Rust is getting stronger, and luckily it has enough safeguards in place to know how to go into the squat without pulling its back.  This part can't be understated.  The philosophy behind Rust is one of having good fundamentals.  As long as the good fundamentals are there, the language can grow on that foundation.

But you don't have to take my word on it.  Like any fix for the Blub Paradox, it comes down to programmers picking up Rust and seeing if the kinds of capabilities Rust gives them allows them to express themselves efficiently and in a way that avoids classes of bugs that distract from the problem at hand.  

It's natural to think that some features are ornamental when you're first coming to them.  Time and exposure are the real test.

*Special thanks to Yehuda Katz and Jason Turner for reviewing this post and sending me their feedback*