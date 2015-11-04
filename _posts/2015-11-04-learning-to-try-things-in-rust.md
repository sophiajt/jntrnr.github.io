---
title: Learning to 'try!' things in Rust
---

I started learning Rust in earnest a few weeks ago.  Coming from C++, a fair share of the idioms felt right at home.  There was clear memory management and an eye towards lightweight abstractions.

But when I started looking around for how to do exceptions, I was surprised to find that Rust had none.  In its place were types like this:

{% highlight rust %}
fn foo_maker() -> Result<Foo, io::Error> { ...}
{% endhighlight %}

I recognized this style, having come from playing with languages like Haskell, but despite doing some functional programming before I wasn't entirely sure how to approach it.  At first blush, the Result value seems pretty straightforward, it either returns something of Foo or an io::Error.  We know which one it returns by pattern matching out the value:

{% highlight rust %}
var result = foo_maker();
match result {
  Ok(foo) -> foo,
  Err(err) -> println!("Error: {:?}", err) // Ooops, we hit an error, tell the user
}
{% endhighlight %}

Great, you might think, that's cute.  Well, at least at first.  Real code tends to have a few reads in a row:

{% highlight rust %}
let num_chans = read_u16_le(&mut f);
let samples_per_sec = read_u32_le(&mut f);
let avg_bytes_per_sec = read_u32_le(&mut f);
let block_align = read_u16_le(&mut f);
let bits_per_sample = read_u16_le(&mut f);
{% endhighlight %}

Since we're reading from a file, any of these reads could hit an error.  Perhaps the hard drive dies, or we prematurely hit the end of the file.  Because of that, each function returns a Result that we later have to pattern match to get the value out.  This means we have to pattern match every result.  Yuck.

If you poke around on the internets, you'll find a sneaky way around this using ```.unwrap()```.  The ```.unwrap()``` call does just what it name implies: it unwraps the Result and gives us the value, assuming the result is Ok.  What if it errored with an Err? Well, then the application panics.

{% highlight rust %}
let num_chans = read_u16_le(&mut f).unwrap();
let samples_per_sec = read_u32_le(&mut f).unwrap();
let avg_bytes_per_sec = read_u32_le(&mut f).unwrap();
let block_align = read_u16_le(&mut f).unwrap();
let bits_per_sample = read_u16_le(&mut f).unwrap();
{% endhighlight %}

If you felt a cold shiver run down your spine looking at that, good on you :).  Now let's talk about how we actually fix this code.

# Do or do not. There is a try!

Looking a bit further in Rust docs, I learned about another way to call functions using the ```try!``` macro.  Maybe, I thought, ```try!``` would save us.  So I tried it out:

{% highlight rust %}
use std::io;
use std::fs::File;

fn main() {
    let mut f = try!(File::open("foo.txt"));
}
{% endhighlight %}

...and **BOOM**

```
<std macros>:5:8: 6:42 error: mismatched types:
 expected `()`,
    found `core::result::Result<_, _>`
(expected (),
    found enum `core::result::Result`) [E0308]
<std macros>:5 return $ crate:: result:: Result:: Err (
<std macros>:6 $ crate:: convert:: From:: from ( err ) ) } } )
read_a_file.rs:5:17: 5:44 note: in this expansion of try! (defined in <std macros>)
<std macros>:5:8: 6:42 help: run `rustc --explain E0308` to see a detailed explanation
error: aborting due to previous error
```

Okay, seriously?  I feel like I jumped in the Haskell pool after drinking a C++ template error cocktail.  What *is* that?

Luckily after asking around, I was finally able to piece together what was going wrong.  The hint lies in these lines:

```
 expected `()`,
    found `core::result::Result<_, _>`
```

Ah, some type mismatch is happening.  Our main function doesn't return anything, which is the ```()``` in the 'expected' above, but ```try!``` only works inside functions that return a ```Result```.  This might be the first time in my life I've ever seen a function in a C-style language that was callable only inside functions of a particular type.  

Shouldn't be too hard to fix, just put it in a function that returns Result, right?

{% highlight rust %}
use std::io;
use std::fs::File;

fn read_file() -> Result<(), io::Error> {
    let mut f = try!(File::open("foo.txt"));
    Ok(())
}
fn main() {
    read_file();
}
{% endhighlight %}

Yup, that did it.  Then I promptly closed my editor and walked away.  I mean, this thing just felt *alien*.  Unnatural.  Gross.

What I didn't realize was that a few weeks later I'd grok the genius in this approach.  

## Genius #1 - Unlike try/catch, try! is precise

Let's look at a real-life example from a project I'm working on:

{% highlight rust %}
fn load_data(fname: &String) -> Result<Data, io::Error> {
  let mut f = try!(File::open(fname));
  let num_pages = try!(f.read_u8()) * 4;
 
  let mut pages : Vec<Vec<u8>> = Vec::new();
  for _ in 0..num_pages {
      let mut buffer = [0; 4096];
      try!(f.read(&mut buffer));
      
      pages.push(buffer.iter().cloned().collect());
  }
  // ...
}
{% endhighlight %}

It may take your eyes a minute to adjust to the new syntax, but knowing what we know, we can look at the code above and know exactly which functions might give us errors.  The try!() call even works on calls that are part of an expression, like ```try!(f.read_u8()) * 4```.  

Compare this to traditional try/catch blocks and how they tend to get put around whole sections of code when in reality only parts of the code blocks may throw exceptions.  The end result is code that's very clear and doesn't muddy exceptional code and value-based code together.

## Genius #2 - Feels like exceptions

With this little bit of syntax, we get something that is value-based, but still feels like exceptions.

Let's say your task is to write code that will load a variety of audio file formats.  In each audio format loader, you have various subsections that get loaded (eg, a header, a metadata chunk, a chunk of audio data, etc).  It'd be great if you could create specialized functions for each.  If there are any errors, you just want to bubble that back out to the original caller loading the file.  The resulting code may look something like:  

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

We can refactor our file loading into helper functions, and we continue to use the same familiar ```try!``` pattern.  How does this work?  The trick is that all the Results agree that ```io::Error``` is the error case.  If one of my helper functions hits a bad read and has to return an error, that error naturally bubbles all the way out without calling any additional code.  

The end result is that we have precise syntax that is transparent about errors at every layer, stays pretty lightweight, and gives us the familiar "bubble out" feel of languages with exceptions.

# Trade-offs

Of course, there are trade-offs.  We end up repeating try!() in places we might have just wrapped in one big try/catch block.  For example, the fix to our earlier use case might look like this:

{% highlight rust %}
let num_chans = try!(read_u16_le(&mut f));
let samples_per_sec = try!(read_u32_le(&mut f));
let avg_bytes_per_sec = try!(read_u32_le(&mut f));
let block_align = try!(read_u16_le(&mut f));
let bits_per_sample = try!(read_u16_le(&mut f));
{% endhighlight %}

Personally, I'm happy making the trade-off for the added precision.  

We also saw another trade-off earlier.  If you use ```try!``` in a function that doesn't return a Result, you get an incomprehensible error message.  But once you know what's going on, you can use it with confidence.
