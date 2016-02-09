---
title: Going down the rabbit hole with Rust traits
---

One of the first things you might notice about Rust, if you come to it from other OOP-style languages, is that Rust separates methods from the data they work on.  You create your ```struct```, then you ```impl``` a few methods on it later.

{% highlight rust %}
struct Rect {
    height: i32,
    width: i32
}

impl Rect {
    fn area(&self) -> i32 { self.height * self.width }
}

fn main() {
    let r = Rect { height: 10, width: 5 };
    println!("area: {}", r.area());
}
{% endhighlight %}

Which, to me, seemed slightly awkward at first.  But, as I dug in a little futher, I started asking questions like "Can I do that to existing types?  Even basic types like the String type?"  Indeed you can.  In this post we'll see some of what's possible.

*Warning: we're going down a rabbit hole under the influence of the [Bad Idea Bears](https://en.wikipedia.org/wiki/Avenue_Q).  I hope you have your snorkel. :)*

# Warming up

We've talked a little about the basics already.  In Rust, ```impl``` gives us a way to "add" methods onto types.  But, when we try to do the same thing to ```String```, we hit a snag:

{% highlight rust %}
impl String {
    fn my_len(&self) -> usize { self.len() }
}

fn main() {
    let name = "foo".to_string();

    println!("my length: {}", name.my_len());
}
{% endhighlight %}

When we try to compile this, the compiler gives us back an error:

```
mylen.rs:1:1: 3:2 error: cannot define inherent `impl` for a type outside of the crate where the type is defined; define and implement a trait or new type instead [E0116]
mylen.rs:1 impl String {
mylen.rs:2     fn my_len(&self) -> usize { self.len() }
mylen.rs:3 }
```

So we can't ```impl``` for a type outside of... oh wait.  We can do it, so long as we give it a trait to work with.

Try #2:

{% highlight rust %}
trait MyLen {
    fn my_len(&self) -> usize;
}

impl MyLen for String {
    fn my_len(&self) -> usize { self.len() }
}

fn main() {
    let name = "foo".to_string();

    println!("my length: {}", name.my_len());
}
{% endhighlight %}

Woah, that worked!  With that, we peek into the rabbit hole.

# Traits and extending strings

To kick off our explorations, let's give ourselves an example to use.  Let's say we want to read in the contents of some file, and we get the filename for that file from the commandline.

{% highlight rust %}
use std::io::Error;

fn read_all(fname: String) -> Result<String, Error> {
    use std::fs::File;
    use std::io::prelude::*;

    let mut contents = String::new();
    let mut f = try!(File::open(fname));
    try!(f.read_to_string(&mut contents));
    Ok(contents)
}

fn main() {
    use std::env::args; 

    if let Some(fname) = args().nth(1) {
        if let Ok(contents) = read_all(fname) {
            println!("{}", contents);
        }
    }
} 
{% endhighlight %}

Wouldn't it be nice to not mix and match helper functions and methods?  Wouldn't it read a little nicer to have```fname.read_all()```?

Using the same pattern we used before, this is pretty easy to do. 

{% highlight rust %}
use std::io::Error;

trait FileReader {
    fn read_all(&self) -> Result<String, Error>;
}

impl FileReader for str {
    fn read_all(&self) -> Result<String, Error> {
        use std::fs::File;
        use std::io::prelude::*;

        let mut contents = String::new();
        let mut f = try!(File::open(self));
        try!(f.read_to_string(&mut contents));
        Ok(contents)
    }
}

fn main() {
    use std::env::args; 

    if let Some(fname) = args().nth(1) {
        if let Ok(contents) = fname.read_all() {
            println!("{}", contents);
        }
    }
}    
{% endhighlight %}

That's not bad!  Go get my nth argument, check if we're good, then read the contents.

If we could stop ourselves there, we'd be mostly okay.  Our example is still pretty readable, though we might get a few dirty looks from the senior devs in the lunch line.

# Traits and extending Option

The Bad Idea Bears call us deeper!  Can we play a little [code "golf"](https://en.wikipedia.org/wiki/Code_golf) and squeeze out using ```Option``` in our main?  Sure!  All we need to do is create another ```impl``` for our ```FileReader``` trait, this time for ```Option``` (specifically those that contain Strings):

{% highlight rust %}
use std::io::Error;

trait FileReader {
    fn read_all(&self) -> Result<String, Error>;
}

impl FileReader for str {
    fn read_all(&self) -> Result<String, Error> {
        use std::fs::File;
        use std::io::prelude::*;

        let mut contents = String::new();
        let mut f = try!(File::open(self));
        try!(f.read_to_string(&mut contents));
        Ok(contents)
    }
}

impl FileReader for Option<String> {
    fn read_all(&self) -> Result<String, Error> {
        use std::io::ErrorKind;
        match *self {
            Option::None => Err(Error::new(ErrorKind::NotFound, 
                "No such file or directory")),
            Option::Some(ref s) => s.read_all()
        }
    }
}

fn main() {
    use std::env::args; 
    
    if let Ok(contents) = args().nth(1).read_all() {
        println!("{}", contents);
    }
}
{% endhighlight %}

Yay!  Our main is smaller, and we've still managed to maintain at least a *little* of our decency.  I mean, we could have used ```unwrap```, and thrown care to the wind!  Instead, we're still doing error-checking on our contents. Albeit, our error-checking is kinda smooshed into one place.

What's that, Bad Idea Bear?  We should go further?  Okay!

# Traits and extending iterators

We're doing a ```read_all()``` on the Option, but what if we go right to the arguments themselves?

If we poke around in the docs, we find out that args() returns its own type, [Args](https://doc.rust-lang.org/std/env/struct.Args.html). As we read on we see that Args is just an [Iterator](https://doc.rust-lang.org/std/iter/trait.Iterator.html).

That gives us a couple choices.  We can implement a new trait on Args:

{% highlight rust %}
trait FileReaderIter {
    fn read_nth(&mut self, pos: usize) -> Result<String, Error>;
}

impl FileReaderIter for Args {
    fn read_nth(&mut self, pos: usize) -> Result<String, Error> {
        self.nth(pos).read_all()
    }    
}
{% endhighlight %}

Or, we can work with Iterator:

{% highlight rust %}
trait FileReaderIter : Iterator<Item=String> + Sized {
    fn read_nth(&mut self, pos: usize) -> Result<String, Error> {
        self.nth(pos).read_all()
    }
}

impl FileReaderIter for Args {}
{% endhighlight %}

Let's actually stop here for a minute and look at this one.  We can't extend Iterator itself.  This is a Good Thing(tm), because if we could, we would end up having to extend countless implementors of Iterator.  Instead, we need to create a new trait that extends from Iterator.  

Next, we need to say what *kind* of iterator we can work on.  We know ahead of time that this works on specifically iterators over Strings.  We use the ```<Item=String>``` to lock that knowledge in place for the compiler.  This is called an *associated type*.

With the new trait and associated type, we're almost done.  Lastly, we're doing something a little different this time around.  We have a default implementation inside of the trait instead of just having a prototype.  A default implementation saves you the trouble of having to repeat yourself with each type that implements the trait, if the function is general and trivial.

The gotcha is that if you use a default implementation that uses ```self```, the compiler needs to know how big ```self``` will be because it will pass it by-value.  Hence, anything that implements ```FileReaderIter``` must have a known size so the compiler knows how many bytes to copy in when ```self``` is passed to the method.  To ensure this, we add another trait to inherit from, ```Sized```.  With that, we can say that Args trivially implements this trait.

Phew, okay, that's it.  What have we gotten for our effort? 

Both approaches give us this simple main:

{% highlight rust %}
fn main() {
    use std::env::args;

    if let Ok(contents) = args().read_nth(1) {
        println!("{}", contents);    
    }
}
{% endhighlight %}

At this point we're far enough down in the rabbit hole that there's only a pinpoint of light from the entrance. 

Let's work our way back out, shall we?

*(ps: if you want to follow Bad Idea Bear further, this little nugget is hidden in the Rust docs: both Result and Option implement iterator traits and can be [used as iterators](https://play.rust-lang.org/?gist=3a890c9c2617c4f1d4f1&version=stable))*

# Climbing back out of the rabbit hole

Cleverness and readability often trade off with each other.  As we tried to get more clever and squeeze our main down, the end result became more code we had to maintain.  We also increased the amount of 'magic' we used, whether it was treating multiple failure styles/cases as one case or the amount of additional capabilities we were giving types like String and Option.

Rust's trait system is powerful and clever.  It lets you express the algorithms that are core to your project in a way that can even extend the basic Rust API.  Indeed, that's one reason why the Rust language and core libraries are intentionally kept pretty small and tidy: so you can make these kind of additions (using eg. modules from [creates.io](https://crates.io/)) and the standard library doesn't get in your way.

But, as they say, with great power comes great opportunities to follow Bad Idea Bears down rabbit holes.
