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

My solution to problem 1 is pretty straightforward, but you can already start to see some of the characteristic earmarks of Rust.  On **line 2**, there's a let keyword which creates a variable.  After that is the mut keyword for saying that the variable we're creating is mutable, as variables are immutable by default.

**Line 4** shows off using an iterator.  Rust's iterators are lazy, meaning they generate values only as needed.  This gives them some cool properties, which we'll see later, but it has the property of being relatively cheap to iterate over them, since we don't generate a full range before iteration starts.

**Line 5** reminds us that we're not in C.  The if doesn't require parens around its condition.  Indeed, if you put them there, the compiler warns you that you don't need them.  

**Line 13** shows how to print out a value.  Note the '!' after println.  If you're like me, you're bound to leave this off once or twice and get an incomprehensible error message.  Similar to C and many other languages, you pass println! a format string.  Here, we just use "{}".  The {} part means to use a default output of the argument we pass in, which in our case just writes out the number.

# Problem 2

"sum of even fibonacci below 4,000,000"

{% highlight rust linenos %}
struct Fibonacci {
    curr: u64,
    next: u64,
}

// Implement 'Iterator' for 'Fibonacci'
impl Iterator for Fibonacci {
    type Item = u64;
    // The 'Iterator' trait only requires the 'next' method to be defined. The
    // return type is 'Option<T>', 'None' is returned when the 'Iterator' is
    // over, otherwise the next value is returned wrapped in 'Some'
    fn next(&mut self) -> Option<u64> {
        let new_next = self.curr + self.next;
        
        self.curr = self.next;
        self.next = new_next;
        
        // 'Some' is always returned, this is an infinite value generator
        Some(self.curr)
    }
}

// Returns a fibonacci sequence generator
fn fibonacci() -> Fibonacci {
    Fibonacci { curr: 1, next: 1 }
}

fn main() {
    let mut sum:u64 = 0;
    for i in fibonacci().take_while(|&x| x < 4000000).filter(|&x| x % 2 == 0) {
        sum += i;
    }
    println!("sum: {}", sum);
}
{% endhighlight %}

Lest you think that I somehow jumped from beginner level to advanced in a few minutes, rest assured that I used the tried and true method of the [copy/paste](http://rustbyexample.com/trait/iter.html).  In trying to find out how to make an iterator so I could iterate over a fibonacci sequence, I found the source to do just that.  Ah, the internet.

There's a fair bit going on here, so let's break it down by line.

**Lines 1-4:** Create a simple struct to hold the current value to be returned as well as the next value.  Fibonacci needs at least these two values to continue to calculate each iteration.

**Line 7:** Now Rust is showing off the fancy.  What is going on here?  In Rust, a type, like our Fibonacci struct from lines 1-4, can implement a trait.  What's a trait?  Think of it as a small contract with the compiler.  If you can show how your type satisfies all the requirements of the contract, then wherever that trait is required, the compiler knows how to use your type.  In this example, we want to use our type in the ```for..in``` language feature.  In some languages, this wouldn't be possible.  In Rust, if we can satisfy the Iterator trait, the compiler has enough information to allow that type to be used in the ```for..in``` feature.

**Line 8:** Ah the problem with copy/paste.  As I started writing this blog post, I had no idea what that line was doing.  Docs to the rescue.  It turns out that this line is saying what the 'associated type' is.  With some traits, there's going to be both a) the type that is implementing the trait (here: our Fibonacci struct) and b) an additional type that needs to be known to get the rest of the story.  For us, since we're creating an iterator over the fibonacci sequence, we need to let Rust know what the type of each "turn of the crank" on the iterator will output.  Each fibonacci number will be a 64-bit unsigned integer, so we use the Rust short-hand type ```u64```.

**Lines 12-20:** Here's where we describe how to turn the crank for each step of our iterator.  We calculate what the next turn will output and then return the current value.  This allows us to keep one in reserve ready to be used to calculate the next value, and so on.  You can see on line 19 that instead of just returning a ```u64``` directly, we instead say that it's optionally a ```u64```.  While in our case we iterate forever, using ```Option``` here gives us a way to shut off the valve and finish the iterator if we ever wanted to. 

Notice, too, on **line 19** the value is said last and the return statement is elided.  This is just shorthand for returning a value from the function.

**Lines 24-26:** Create a simple function to give us our starting value to begin our fibonacci sequence.  You can see a couple Rust-isms here, too.  As before, a value by itself is an implied return value.  It also shows how to create an instance of the struct without a constructor.

**Line 30:** We're finally in the main!  I've forgotten what problem we were solving by now.  Ah right, "find the sum of even fibonacci numbers less than 4,000,000".  Let's look at the solution.  Since there's a lot going on in this fluent-style approach, let's break it up into steps:

```fibonacci()``` - we call our function and get out the default value to start iterating with.  Since we already told Rust how to use values of this type as iterators, it gets all the capabilities that iterators have.

```.take_while(|&x| x < 4000000)``` - that didn't take long, we're already jumping in and using the functionality of the iterator to check each turn of the crank, and if we exceed 4,000,000 stop.  

The ```|&x| x < 4000000``` is a simple function that takes in a reference to each element we bind to x, and then we check the value we're handed in the body of our function.  If you're coming from C++, this reference style might look a little strange.  Aren't you risking someone updating the value?  Turns out Rust's references are [immutable by default](http://doc.rust-lang.org/book/references-and-borrowing.html#borrowing).  The reference here also shows us the performance characteristics of ```.take_while```.  By this I mean, when we look at the ```.take_while``` call, we see it will only pass references, so know ahead of time if we're iterating over big structures, we're not paying the price of copying each one with each step of the iterator.

Speaking of performance, it may look like .take_while is going to be exceedingly expensive.  Is it consuming all of our fibonacci sequence until we have all the items we need to get to 4,000,000?  Luckily, it isn't.  Instead, ```.take_while``` itself returns an iterator that you can continue your fluent calls on.  I think of it like a series of machines in a factory:

-> [  ] -> [  ] -> [  ] ->

If I start the conveyor up and I pull from the right hand, I'm moving whatever was on the far left all the way through the system.  In our case, we're starting up the fibonacci iterator and getting that machine running.  It spits out a single fibonacci-shaped number for us.  This number chugs along and enters the next machine, which is our machine that checks if it's too big.  If it's not, the conveyor chugs along and out drops a single fibonacci-shaped number of the acceptable size.  

As these machines work in lock-step, we're getting just what we need when we need it rather than trying to run ahead and doing a bunch of work in advance.  This allows you compose a lot of steps together and still stay efficient.

```.filter(|&x| x % 2 == 0)``` -  Just like ```.take_while``` above, this one checks each number as it passes through.  This time, it only lets the number through if it's divisible by 2, and hence even.  

Now we have all the steps in place, each time we turn the crank of the whole iterator, only fibonacci numbers that are less than 4,000,000 and even will pop out.

**Line 31:** We've already done all the hard work.  Now we just have to sum those numbers together.

With that, we're done.  While it was quite a mouthful, we can see the compositional nature of Rust, a bit of the type system, and how iterators work in more detail.

# Problem 3 

"highest prime factor of 600851475143"

{% highlight rust linenos %}
fn is_prime(num:u64) -> bool {
    for i in 2..(num / 2 + 1) {
        if num % i == 0 {
            return false;
        }
    }
    return true; 
}    

struct Prime {
    curr: u64,
}

impl Iterator for Prime {
    type Item = u64;
    fn next(&mut self) -> Option<u64> {
        let mut new_next = self.curr + 1;
        while !is_prime(new_next) {
            new_next += 1;
        }

        self.curr = new_next;

        Some(self.curr)
    }
}


// Returns the primes
fn primes() -> Prime {
    Prime { curr: 1 }
}

fn main() {
    let mut num:u64 = 600851475143;
    let mut highest_prime_factor = 0;
    
    for i in primes() {
        if num % i == 0 {
            highest_prime_factor = i;
        }
        while (num % i == 0) && (num >= 2) {
            num /= i;
        }
        if num == 1 {
            break;
        }
    }
    println!("num: {}", highest_prime_factor);
}
{% endhighlight %}

Very similar to Problem #2 above, I create an implementation of Iterator, so I can use it later.  This time, rather than creating a stream of fibonacci values, I create a stream of prime numbers.

Just like we had a struct for Fibonacci in Problem #2, I create one here for Prime numbers.  Since we don't need the previous one to calculate the next one, I only keep the current value around and then start with that number + 1 when trying to find the next prime.

Once we have our iterator, we take each prime, and then try to divide out all its multiples from our give number.  What's left should be our highest prime factor.  Easy peasy.

PS: I know it's probably already too late, but I should have warned you to avert your eyes from my inefficient prime number skills :)

# Problem 4

"largest palindrome product of two 3-digit numbers"

{% highlight rust linenos %}
fn is_palindrome(num: u64) -> bool {
    let s = num.to_string();
    for (i1, i2) in s.chars().zip(s.chars().rev()) {
        if i1 != i2 {
            return false;
        }
    }
    return true;
}

fn main() {
    let mut largest = 0;
    for i in 100..999 {
        for j in 100..999 {
            if is_palindrome(i * j) && ((i * j) > largest) {
                largest = i * j;
                println!("Largest: {}, {} x {}", largest, i, j);
            }
        }
    }
}
{% endhighlight %}

Problem #4 also uses some iterators to find the palindrome product (**line 13** and **line 14**).  If it finds a palindrome product, and that new product is larger than what it saw before, it replaces it with the new one.

There's one fun trick in this solution on **line 3**.  We need to check if the number is a palindrome.  While you can do this [numerically](http://stackoverflow.com/questions/199184/how-do-i-check-if-a-number-is-a-palindrome), I couldn't help but use some more iterators.  This time, after converting the number to a string, we create an iterator over the characters in the string (using ```.chars()```).  We want to see if the number is a palindrome, so one way to test this is that if we reverse the string and if still get the same value, we know we've got one.

That's exactly what we do.  Here, I use ```.zip(s.chars().rev())``` to combine the iterator that goes forward over the characters with one that goes in reverse.  The ```.zip()``` call brings both iterators together and iterates over both iterators for us, returning a pair (or tuple) of values.  Think of it running both machines side by side, and with each turn of the crank one value pops out of each and zip returns both values tied together with a bow.

Now that we have a single pair, we immediately use it.  At the beginning of line 3, you can see the for ```(i1, i2)``` in part of the line.  Just as before ```for..in``` will run over the iterator.  We know that the iterator is going to be outputting pairs of values.  We could have just grabbed the pair as a single value and used it later, but here I go one step further by taking the pair the iterator gives me and pulling it apart so I can use it.  This pulling apart, called destructuring, is a handy way of working with structured data when you're interested in its constituent parts rather than the structure itself.  In this case, this lets me more easily compare the left side to the right side, so I destructure the pair into two values: ```i1``` and ```i2```.
