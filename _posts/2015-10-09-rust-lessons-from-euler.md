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

# Problems 5 through 7

Problems [#5](https://github.com/jonathandturner/rustnewbie/blob/master/euler/src/ex5.rs), [#6](https://github.com/jonathandturner/rustnewbie/blob/master/euler/src/ex6.rs), and [#7](https://github.com/jonathandturner/rustnewbie/blob/master/euler/src/ex7.rs) are all straightforward and don't show off anything new in the language.  Let's jump ahead to the next interesting one.

# Problem 8

"largest product of 13 adjacent digits"

{% highlight rust linenos %}
fn main() {
    let input = 
        "73167176531330624919225119674426574742355349194934\
        96983520312774506326239578318016984801869478851843\
        85861560789112949495459501737958331952853208805511\
        12540698747158523863050715693290963295227443043557\
        66896648950445244523161731856403098711121722383113\
        62229893423380308135336276614282806444486645238749\
        30358907296290491560440772390713810515859307960866\
        70172427121883998797908792274921901699720888093776\
        65727333001053367881220235421809751254540594752243\
        52584907711670556013604839586446706324415722155397\
        53697817977846174064955149290862569321978468622482\
        83972241375657056057490261407972968652414535100474\
        82166370484403199890008895243450658541227588666881\
        16427171479924442928230863465674813919123162824586\
        17866458359124566529476545682848912883142607690042\
        24219022671055626321111109370544217506941658960408\
        07198403850962455444362981230987879927244284909188\
        84580156166097919133875499200524063689912560717606\
        05886116467109405077541002256983155200055935729725\
        71636269561882670428252483600823257530420752963450";

    let mut largest = 0;
    let input_bytes = input.as_bytes();
    let mut largest_string: &[u8] = &input_bytes[0..1];

    let span_width = 13;

    for i in 0..(input_bytes.len() - span_width + 1) {
        let mut sum = 1u64;
        for j in 0..(span_width) {
            sum *= (input_bytes[i + j] - 48) as u64;
        }
        if sum > largest {
            largest = sum;
            largest_string = &input_bytes[i..(i+span_width)];
        }
    }

    println!("Largest: {} is {:?}", largest, std::str::from_utf8(largest_string));
}
{% endhighlight %}

In Problem #8, we see an example of working with strings, this time using a new feature called a 'slice'.  We start with the block from **line 3** to **line 22** that gives us this giant input string.  Notice that I use the backslash '\' to continue the line to the next line.  Rust is smart enough to let us indent without introducing new whitespace into the string. 

Now that we have our string, on **line 25** we turn it into a array of bytes we can search through.  Notice we didn't use ```.chars()``` this time.  Recall that ```.chars()``` gave us an iterator.  Rather than iterating over one value at a time here, we want the flexibility to look across spans in our string.  To do that, we use ```.as_bytes()``` to get an array (technically, a byte slice).

**Line 26** introduces the slice we'll use.  A slice is a window into values in memory.  Since we want to look at a span of values, we create a holder for this slice.  We'll later set it to spans of our 13 adjacent digits.  PS: you'll notice I set it to a dummy default value, which I later throw away.  This is to get around the compiler warning about uninitialized values.

The rest of the search follows fairly naturally.  We do an iterator over the indices in our input, stopping short of the length of the span.  We use that index to calculate the product at that span (**line 32** to **line 34**).  Once we find a match, we save it off, using a slice to save off the winning span (**line 37**).

Finally, we use another format string ```{:?}```, which can print out our winning span for us using the debug formatter.  The ```{:?}``` format is a handy builtin which can handle a wider range of types with a default formatter.

# Problem 9

Problem [#9](https://github.com/jonathandturner/rustnewbie/blob/master/euler/src/ex9.rs) is another solution that's simple and doesn't show off any new features.  Moving on.

# Problem 10

"sum of all primes below 2 million"

{% highlight rust linenos %}
fn main() {
    let mut sum = 0;
    const SIZE: usize = 2000000;
    let mut slots: [bool; SIZE] = [true; SIZE];
    slots[0] = false;
    slots[1] = false;
    
    // We calculate the primes using a simple stride and marking off multiples 
    for stride in 2..(SIZE/2) {
        let mut pos = stride;
        while pos < (SIZE - stride) {
            pos += stride;
            slots[pos] = false;
        }
    }
    
    for (idx, pr) in slots.into_iter().enumerate() {
        if *pr { sum += idx; }
    }

    println!("Sum: {}", sum);
}
{% endhighlight %}

Reading the problem description, you might have guessed that I would use the iterator solution from earlier.  The astute reader probably already noticed that doing so would be. very. slow.  Especially, as we look at primes above a million, where each step is itself taking hundreds of thousands, if not millions, of calculations.  While I could let it run and heat up my apartment, it's better to do a more direct approach.

We trade space for time. 

On **line 4**, I create my first fixed size array that will hold whether or not the number in that position is prime.  Once we have the variable, it's initialized using another cool shorthand: ```[true; SIZE]```.  This creates an array of boolean true values of the size given on the right of the ';', here the SIZE constant I defined.  For example, you can create a 500-element array of zeros using ```[0; 500]```.  Neat.

Once we have our slots, we loop over them using any possible multiple and check that off the list by setting that position to false.  Once we're done, we have an array that tells us where the primes are. 

The only trick remaining is how to get the numbers back out again.  To do this, we use the iterator on **line 17**.  Similar to our fluent iterators before, this time we turn our array of slots into an iterator, and then call ```.enumerate()```.  The ```.enumerate()``` call is a special kind of zip that gives us the index at that point in the iteration paired with the actual value.  Just as before, we destructure the index and the boolean that indicates if it's a prime number separately.

On **line 18**, we check if the number is prime, and if so we add it to our sum.  You'll notice the ```*pr``` call here.  Just as in C, this lets me dereference and get at the value at that location in memory.  I could have instead written for ```(idx, &pr)``` in and just gotten to the value that way, using destructuring instead of dereferencing.  With that change, the line would have been ```if pr { ... }```, and I could get at the value ```pr``` without needing to dereference.

# Problem 11

"find the largest product in a grid"

{% highlight rust linenos %}
let input = "\
    08 02 22 97 38 15 00 40 00 75 04 05 07 78 52 12 50 77 91 08 \
    49 49 99 40 17 81 18 57 60 87 17 40 98 43 69 48 04 56 62 00 \
    81 49 31 73 55 79 14 29 93 71 40 67 53 88 30 03 49 13 36 65 \
    52 70 95 23 04 60 11 42 69 24 68 56 01 32 56 71 37 02 36 91 \
    22 31 16 71 51 67 63 89 41 92 36 54 22 40 40 28 66 33 13 80 \
    24 47 32 60 99 03 45 02 44 75 33 53 78 36 84 20 35 17 12 50 \
    32 98 81 28 64 23 67 10 26 38 40 67 59 54 70 66 18 38 64 70 \
    67 26 20 68 02 62 12 20 95 63 94 39 63 08 40 91 66 49 94 21 \
    24 55 58 05 66 73 99 26 97 17 78 78 96 83 14 88 34 89 63 72 \
    21 36 23 09 75 00 76 44 20 45 35 14 00 61 33 97 34 31 33 95 \
    78 17 53 28 22 75 31 67 15 94 03 80 04 62 16 14 09 53 56 92 \
    16 39 05 42 96 35 31 47 55 58 88 24 00 17 54 24 36 29 85 57 \
    86 56 00 48 35 71 89 07 05 44 44 37 44 60 21 58 51 54 17 58 \
    19 80 81 68 05 94 47 69 28 73 92 13 86 52 17 77 04 89 55 40 \
    04 52 08 83 97 35 99 16 07 97 57 32 16 26 26 79 33 27 98 66 \
    88 36 68 87 57 62 20 72 03 46 33 67 46 55 12 32 63 93 53 69 \
    04 42 16 73 38 25 39 11 24 94 72 18 08 46 29 32 40 62 76 36 \
    20 69 36 41 72 30 23 88 34 62 99 69 82 67 59 85 74 04 36 16 \
    20 73 35 29 78 31 90 01 74 31 49 71 48 86 81 16 23 57 05 54 \
    01 70 54 71 83 51 54 69 16 92 33 48 61 43 52 01 89 19 67 48";
    
let input_split = input.split_whitespace();
let input_as_num: Vec<i32> = input_split.map(|x| 
    match i32::from_str_radix(x, 10) {
        Ok(v) => v,
        Err(u) => { println!("Garbage in input: {}", u); 0 }
    }
).collect();
{% endhighlight %}

I'm only showing the interesting part of the solution since it's a bit long, but you can read the [whole solution](https://github.com/jonathandturner/rustnewbie/blob/master/euler/src/ex11.rs).

Here we do a bit more string manipulation using some new methods.  The ```.split_whitespace()``` method on **line 23** lets us turn our input string into an iterator of strings, splitting at any whitespace. 

Once we have this iterator, **line 24** takes this iterator and maps a function over it.  This function attempts to convert each substring to a signed 32-bit integer ```i32``` using the ```.from_str_radix(x, 10)``` call.  This call returns a ```Result```, a way of handling either a success or failure condition of an action that might fail.  As is the case with converting from strings to numbers, if you happen to pass in something that can't be converted to a number, there needs to be a way to signal back out that the conversion failed.  Some languages do this through exceptions, which break out of normal execution and give you an error you can handle.  Rust uses a simpler, more functional approach and treats errors as just any other value.  With this method, the type system encourages us to be vigilant and always have code available to handle errors.

To see which of success/failure is returned, we use the match keyword.  Just as in other languages with pattern matching, the match keyword lets us ask which of the possible values is in the Result: ```Ok``` or ```Err```.  Pattern matching lets us destructure and find the success value or error message.

Finally, on **line 29**, we use the ```.collect()``` method to run through the iterator and create a vector of ```i32```.

# Problem 12

"triangle number with over 500 divisors"

{% highlight rust linenos %}
fn main() {
    let mut num = 0u64;

    // Cache the primes we'll be using
    let first_primes: Vec<u64> = primes().take(1000).collect();

    for i in 1..1000000u64 {
        num += i;

        let mut num_div = 1u64;

        let mut num_tmp = num;
        for &x in &first_primes {
            let prime = x;
            if (num % prime) == 0 {
                let mut exponent = 1;

                while (num_tmp % prime) == 0 {
                    exponent += 1;
                    num_tmp /= prime;
                }
                num_div *= exponent;
            }
            if num_tmp == 1 { break; }
        }

        if num_div > 500 {
            println!("num: {}", num);
            return;
        }
    }
}
{% endhighlight %}

Again we'll trim down to the interesting parts, since we've seen the prime iterator before, but you can read the [full file](https://github.com/jonathandturner/rustnewbie/blob/master/euler/src/ex12.rs).

This looks pretty familiar.  On **line 5**, we run our primes iterator, take enough to use later, and turn that into a ```Vec<u64>``` we can use a bunch of times.

The other interesting line is **line 13**.  If you've played with vectors in Rust before, you know that you can iterate over them, like this:

{% highlight rust %}
fn main() {
    let x: Vec<u32> = (1..10).collect();

    for i in x {
        println!("{}", i);
    }
}
{% endhighlight %}

But if we try to do this in our Euler problem, we get a new error:

```
error: use of moved value: `first_primes`
```

What is a *moved value*?  

It turns out that a moved value is part of Rust's ownership system.  Just like the first time a programmer sees a pointer in C or a monad in Haskell, ownership in Rust is one of those things that's so uniquely Rust that it takes getting used to.  Once you begin to understand it, you'll be able to see how the type system is trying to help you describe your intentions a bit more clearly.

If you're scratching your head, here's an example: imagine malloc'ing an array in C and then passing that malloc'ed array into a function.  Who is responsible for cleaning up that memory, the function you called or the function that created the array?  In C, you don't know by looking at the code, you only know as the writer of the code what your intention was.  Maybe the function is called 'cleanup' at which case, yes, you do want it to free that memory.  Or maybe the function is called 'inspect' and you decidedly *don't* want it to free the memory.  This is why Rust makes this very explicit.  By looking at the code, you can follow who is responsible for what.  The set of checks that do this make up Rust's ownership system.

There are a lot of [resources](https://doc.rust-lang.org/book/references-and-borrowing.html) to understand ownership,which do a good job of explaining ownership in detail.  For us, ownership is like trying to answer questions like "who is responsible for this thing?" and "when does this thing get removed?"  Ownership can be passed from variable to variable, and its strictly checked by the compiler.  

When you first encounter the above error, if you haven't already read about ownership, you might be in for a few hours of reading to get caught up.  But we have some hints here that we can get started with to help figure out what's happening.

In our small example, we could iterate over a vector and all was well.  When did things start to go wrong in the Euler problem?

On **line 13**, we want to loop over a vector, and when we look out to **line 7**, we see this is actually an inner loop being used by an outer loop.  That means that if the inner loop takes control of our vector, the next time around the outer loop we've lost the ownership.  

We don't want the inner loop to take ownership and then lose the ability to use the vector again in the next iteration of the loop, so we have to 'borrow' ownership.  As the name implies, by borrowing, we're only taking temporary ownership for a time, and we have to follow all the rules stated when we borrow.  With that in mind, let's look at **line 13** again:

{% highlight rust %}
for &x in &first_primes
{% endhighlight %}

We borrow ```first_primes``` instead of taking permanent ownership using the ```&``` operator.  This operator says "I would like to borrow first_primes, and I can not edit the contents".  The compiler will then be able to check that we're using what we've borrowed correctly.  You might be curious borrowing and also being able to mutate.  To do that, we could've used the ```&mut``` operator assuming the original vector was mutable.

Finally, we use the same destructure trick we did earlier to get at the values of ```x```.

**PS:** In this example, since all we were doing was iterating, we could have used the ```.iter()``` rather than borrowing, though that won't always be the case in other examples that need more direct interaction with the vector.

# Cargo

After I got enough of these problems together that I got tired of building them by hand, I moved over to using Cargo.  Cargo is Rust's build/package tool that is quite handy.  

```
[package]

name = "eulerinrust"
version = "0.1.0"

[[bin]]
name = "ex1"
path = "src/ex1.rs"
```
*An example of Cargo.toml*

Cargo lets you describe dependencies that your project needs, which it fetches from crates.io.  You can also control whether you're doing a release or debug build, and quite a bit more.  I'll no doubt start directly with Cargo for any future Rust projects.

# Summary

These are just tiny examples created while playing around in a fairly simple set of problems, and yet there's actually quite a lot of Rust's richness shining through.  Rust has a powerful iterator story that feels composable and clear.  Rust's error checking is keeping us vigilant about handling error cases, matching types, and remembering who is responsible for values in the system.  Overall, it feels like a very well thought-out language that holds together well.
