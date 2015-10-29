---
title: More examples
---

This is an example of some code:

{% highlight rust %}
struct Foo {
    x: i32
}

impl Foo {
    fn add_to(&self, y: i32) -> i32 {
        //self.x = 30;  // <-- would error
        self.x + y
    }
}

fn main() {
    let foo = Foo { x: 10 };
    println!("Result: {}", &foo.add_to(20));   
}
{% endhighlight %}
