+++
title = "Embedded scripting in Rust"
[taxonomies]
tags = [ "rust", "rhai" ]
+++

For the last few weeks, I've been working on an embedded scripting language for Rust, based loosely on [ChaiScript](http://www.chaiscript.com) called [Rhai](https://github.com/jntrnr/rhai).  What's an embedded scripting language?  While the definition might depend on who you ask, for this post embedded scripting has a few distinct features: 

* A language that derives all functionality from bindings to the hosting environment
* As a result, it allows complete control over the API exposed to the script, letting you lock down the script to your application's needs

In short, it's like the little brother of a full scripting language.  It's smaller, thinner, but still useful for a variety of apps.  

Because I'm using Rust as the host language, I also put a few other constraints on the project to make it work with a wider range of use cases.

* No use of 'unsafe' - this lets us script where security is a concern
* No additional dependencies - keeps our design small
* Easy-to-use - even being safe and lightweight, it still needs to be easy to get started

# Introducing: Rhai

Let's take a look at what Rhai looks like, then we'll break down how it works.

## Hello world

As I mentioned earlier, one of the goals with Rhai was to have minimal setup to get going.  This example is a full "hello world" of scripting that evaluates some script and gives us the result:

```rust
extern crate rhai;
use rhai::Engine;

fn main() {
    let mut engine = Engine::new();

    if let Ok(result) = engine.eval("40 + 2".to_string())
        .unwrap().downcast::<i32>() {
        
        println!("Answer: {}", *result);  // prints 42
    }
}
```

Looking a little closer.  We pull in the Rhai crate and get the scripting engine into view:

```rust
extern crate rhai;
use rhai::Engine;
```

Next, we create an instance of the engine:

```rust
let mut engine = Engine::new();
```

Finally, we call into the engine.  Perhaps the most interesting part here is the roundtrip.  The 'eval' method takes a ```String```, and returns a ```Result<Box<Any>, EvalAltResult>```.  Because we're in a scripting context, everything we're working with is dynamically typed, so as values get passed around, we make liberal use of ```Box<Any>```.  Here, the expression "40 + 2" is going to evaluated to give us a Box'ed number.  Of course, we can also have errors occur during script execution, so we need to return a Result.

To go from a Result to the actual numeric value, we have to take a few steps to peel away the layers.  I cheat and use 'unwrap' to get through the Result, then the downcast takes us from a ```Box<Any>``` to a ```Box<i32>```.

```rust
engine.eval("40 + 2".to_string()).unwrap().downcast::<i32>()
```

That's it.  We've done a full round trip of creating script and then making sense of its output.  The language, at this point, has only basic functionality because we haven't registered anything the script can use.  So let's fix that.

## Registering your functions

To make your functions visible to the script, you'll need to register them with the script engine.  Let's do the same example, but this time we'll create an add function in our native code and call it from script:

```rust
extern crate rhai;
use rhai::{Engine, FnRegister};

fn add(x: i32, y: i32) -> i32 {
    x + y
}

fn main() {
    let mut engine = Engine::new();

    &(add as fn(x: i32, y: i32)->i32).register(&mut engine, "add");

    if let Ok(result) = engine.eval("add(40, 2)".to_string())
        .unwrap().downcast::<i32>() {
        
        println!("Answer: {}", *result);  // prints 42
    }
}
```

The big change in this example is this line:

```rust
&(add as fn(x: i32, y: i32)->i32).register(&mut engine, "add");
```

This uses the function registration capability taken from the FnRegister trait that we've added to our imports.  It's a trait implemenented on functions which gives them a register method.  We call the method and pass into it both our engine and the name we want the function to be called in script.  With that, we're good to go and can call the function.

Let's do two more examples so we can see how some of the other details play out.

## Function overloading

Rhai also has supported for function overloading.  This gives you a way to work with generic functions (like the example below) or to register multiple functions to the same name.  During call resolution, the engine will call these functions and will fall through to the case that matches the types at runtime.

```rust
use std::fmt::Display;

extern crate rhai;
use rhai::{Engine, FnRegister};

fn showit<T: Display>(x: &mut T) -> () {
    println!("{}", x)
}

fn main() {
    let mut engine = Engine::new();

    &(showit as fn(x: &mut i32)->()).register(&mut engine, "print");
    &(showit as fn(x: &mut bool)->()).register(&mut engine, "print");
    &(showit as fn(x: &mut String)->()).register(&mut engine, "print");
}
```

## Working with custom types

Finally, in addition to working with your functions, Rhai also supports working with custom types.  In this example, we introduce a few more features.  First, you can register functions that take a ```&mut``` as their first argument.  This allows them to be used as methods in the script.  Next, we register a special getter function.  This lets us read (or write, if we register a setter also) the member directly in script.

You'll notice too that we make sure our object is clone-able.  As we'll talk about in the next section, this ends up being a requirement when working with values in the script engine.  We make sure to register our custom type with the engine to make cloning possible from inside the engine.

```rust
extern crate rhai;
use rhai::{Engine, FnRegister};

#[derive(Clone)]
struct TestStruct {
    x: i32
}

impl TestStruct {
    fn update(&mut self) {
        self.x += 1000;
    }

    fn get_x(&mut self) -> i32 { self.x }

    fn new() -> TestStruct {
        TestStruct { x: 1 }
    }
}

fn main() {
    let mut engine = Engine::new();

    engine.register_type::<TestStruct>();

    &(TestStruct::new as fn()->TestStruct).register(&mut engine, "new_ts");

    &(TestStruct::update as fn(&mut TestStruct)->())
        .register(&mut engine, "update");
    &(TestStruct::get_x as fn(&mut TestStruct)->i32)
        .register(&mut engine, "get$x");

    if let Ok(result) = 
        engine.eval("var myts = new_ts(); myts.update(); myts.x".to_string())
            .unwrap().downcast::<i32>() {
        
        println!("result: {}", *result); // prints 1001
    }
}
```

# How it works

Let's take a look at how the scripting engine works.  There may be even better ways to do some of these steps, so I'm eager to hear your feedback.  

Without further ado, let's jump in.

## Function resolution

The heart of the system is the function resolver.  In Rhai, all function calls, even all operators like plus and minus, end up going through the resolver.  Yet, it's a relatively simple piece of code.

* First, find all functions which match the name.  Let's say 'foo'
* For each 'foo' function, try to call the function with the argument(s) given
* If it succeeds, return the resulting value
* If it fails, continue trying the rest of the functions registered to 'foo'
* If none succeed, error

Going through all the functions which match a name gives us the ability to do overloaded functions.  It also lets us do operators, which themselves are commonly overloaded.

## Function registration

The next piece to working with functions in the engine is registering them.  There are two kinds of functions that are registered: inner functions, or those created inside the script itself, and external functions, or those created in the native code.  

Inner functions are relatively simple.  They work by creating a new scope, adding the function arguments to this new scope, and then executing the function body.

External functions, if you will, are where the magic happens.  Here, we handle the boundary between the scripting world and the native world.

Let's look at how to register a function with one pass-by-value argument ```fn(T)->U```:

```rust
impl<T: Any+Clone, U: Any+Clone> FnRegister for fn(T)->U {
    fn register(self, engine: &mut Engine, name: &str) {        
        let wrapped : Box<Fn(&mut Box<Any>)->Result<Box<Any>, EvalAltResult>> = 
            Box::new(
                move |arg: &mut Box<Any>| {
                    let inside = (*arg).downcast_mut() as Option<&mut T>;
                    match inside {
                        Some(b) => Ok(Box::new(self(b.clone())) as Box<Any>),
                        None => Err(EvalAltResult::ErrorFunctionArgMismatch)
                    }
                }
            );

        let ent = engine.fns.entry(name.to_string()).or_insert(Vec::new());
        (*ent).push(FnType::ExternalFn1(wrapped));
    }
}
```

There's a lot going on, so let's break it down:

```rust
impl<T: Any+Clone, U: Any+Clone> FnRegister for fn(T)->U
```

As I mentioned earlier, all the types used in the scripting engine need to be clone-able.  We enforce that here as well because we'll use the clone feature in the call.  We are implementing the FnRegister trait on all functions that match this constraint.

Next, we are going to create a wrapper function which will call the function given:

```rust
let wrapped : Box<Fn(&mut Box<Any>)->Result<Box<Any>, EvalAltResult>>
```

All functions of arity one have this internal wrapper function, regardless of if they are pass by value or reference.  This lets us work through a vector of functions easily during function resolution, without having to have too much information out the specific type of the function.

Next, we have the wrapper function itself:

```rust
Box::new(
    move |arg: &mut Box<Any>| {
        let inside = (*arg).downcast_mut() as Option<&mut T>;
        match inside {
            Some(b) => Ok(Box::new(self(b.clone())) as Box<Any>),
            None => Err(EvalAltResult::ErrorFunctionArgMismatch)
        }
    }
);
```

The wrapper is a closure that moves its environment. In this case, it's moving the function we're wrapping.  Once it has ownership, we can safely return the wrapper and complete registration.

Inside the closure, we handle working with the argument.  Because we know the type of the function during registration, we use this knowledge to downcast the argument and give it the proper type.  If this succeeds, we know we have the correct function and can call it.  If it fails, we know this isn't a match and later can try the next function in the overloads.

To call the function, since we're pass by value, we need to take the extra cloning step to get to a value rather than a reference.  This lets us respect the ownership model, albeit at the cost of the clone.

That's the heart of how the resolution system works.  Resolution works through each possible match, and registration gives us a wrapper function that can try each function against the types known at compile time.

## The rest of the story

Many of the remaining features built on these two main components.  As mentioned earlier, when you register special get and set functions for fields, the engine will use them when you work with the fields in script.  

The engine will register a set of basic arithmetic and comparison operators to get started with.  These also use the same function resolution mechanism.

For variables, the engine works by passing a scope stack around.  For lookup, in order to follow the ownership rules (and maintain pass-by-value semantics), we clone variable values when they're used in an expression.  During assignment, we use a mutable reference to their value in the scope stack so that it can be updated with the right-hand side of the assignment.

# What's next?

Rhai is still very young and has a bit of growing to do before it's ready for primetime.  Parsing and error reporting are barebones and don't have the kind of error recovery you'd expect from more industrial-strength scripting.  There are also a number of convenience features that would be relatively easy to add, like vectors, iterators, for..in.., and so on.
 
If you'd like to help out, jump over to the [GitHub site](https://github.com/jntrnr/rhai).