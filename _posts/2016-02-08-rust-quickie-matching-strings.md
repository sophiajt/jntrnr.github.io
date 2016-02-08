---
title: Rust quickie - matching Strings
---

In case you find yourself trying to match a ```String``` (perhaps as part of an ```Option``` or ```Result```), here's a little trick.

As a concrete example, let's say you're working with commandline args and want to do different things if it's there and equal to a special value (like "-"), if it's any other filename, or if it's not there.

{% highlight rust %}
match args().nth(1) {
    Some("-")     => println!("Input is stdin"),
    Some(ref x)   => println!("Open file: {}", x),
    None          => println!("Open default file /foo/bar")
}
{% endhighlight %}

When you try to compile this, you get:

```
error: mismatched types:
 expected `collections::string::String`,
    found `&'static str`
(expected struct `collections::string::String`,
    found &-ptr) [E0308]
```

Bummer! At least on the surface, it looks like we don't have an easy way to use a string constant like "-" in our pattern match when what we're matching against a String.

Luckily, Rust pattern matching has a way to help us.  We can approximate the above by using ```match guards```.  Match guards allow us to put an additional ```if``` expression on the pattern.

{% highlight rust %}
match args().nth(1) {
    Some(ref x) if x == "-" => println!("Input is stdin"),
    Some(ref x) => println!("Open file: {}", x),
    None        => println!("Open default file /foo/bar")
}
{% endhighlight %}

Match guards give you an extra bit of "juice" to express your logic.
