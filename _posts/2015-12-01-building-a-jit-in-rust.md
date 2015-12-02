---
title: Building a JIT in Rust
---

The other day I threw together a simple JIT, and I thought it'd be fun to show the steps I did.  With this, it should be possible to create a page of executable memory, write some machine code into it, and then treat it like a function call from Rust.

Let's get started!  You can grab my [JIT project](https://github.com/jonathandturner/rustyjit), if you want to follow along.  I've only tested this in OS X, but it should adaptable for other platforms.  

# Creating executable memory

For our JIT, we'll need to allocate the memory that will hold our JIT'ted code.  The most important thing here is to create a block of memory that's executable so that we can later jump into it, run code, and then return back out.

To do this, we need a few functions from the standard C library.  

{% highlight rust %}
// added to Cargo.toml
[dependencies]
libc = "0.2.2"

// main.rs
extern crate libc;

// ...
unsafe {
  let mut page : *mut libc::c_void = libc::malloc(size);
}
{% endhighlight %}

Once we've pulled in libc, we need to ```malloc``` some memory.  This will give us a chunk of memory we can work with at the size we want.  Notice also, that we get back a raw pointer to ```c_void```.  In C, we might write this ```void *```.  We'll work with it in this form for a bit, but later move it into a form that's a bit easier to work with in Rust.

Next, we need to align the memory.  Some operating systems, like OS X, require that executable code memory starts at a particular alignment.  For example, that it starts at an address that is exactly a multiple of 0x1000.

{% highlight rust %}
const PAGE_SIZE: usize = 4096;
// ...
unsafe {
  libc::posix_memalign(&mut page, PAGE_SIZE, size);
}
{% endhighlight %}

Once aligned, we now have memory we can safely jump to.  Well, almost.  Our last required step is to actually enable the ability to execute code in this area of memory.

{% highlight rust %}
unsafe {
  libc::mprotect(page, size, libc::PROT_EXEC | libc::PROT_READ | libc::PROT_WRITE);
}
{% endhighlight %}

Now we're ready.  We have something we can write into and then jump into.  Since running JIT code is basically no-mans-land, it's easy to get yourself in trouble.  One step that I add is to also write in the machine code for the ```RET``` instruction, which will let us return from our function even if we happen to accidentally run other memory in this block.

{% highlight rust %}
extern {
    fn memset(s: *mut libc::c_void, c: libc::uint32_t, n: libc::size_t) -> *mut libc::c_void;
}
// ...
unsafe {
  memset(page, 0xc3, size);  // for now, prepopulate with 'RET'
}
{% endhighlight %}

We've allocated our JIT memory, aligned it, set it as executable, and then filled it with the ```RET``` instruction.  We're ready to write some code into it.

# Getting ready to write our first program

To write our first JIT program, let's first make a way to more easily work with our memory.  In its current state, it's a raw C void* pointer, but this is a bit unweildy as we don't have a lot of support to work with that in Rust.  To make something that feels more natural, we'll create a new struct type that will hold our pointer and allow us to create indexing functions for easier access.  This will let us access the memory like ```m[0] = 0x10```.

Let's create the struct we'll use:

{% highlight rust %}
use std::mem;

struct JitMemory {
    contents : *mut u8
}

fn alloc() -> JitMemory {
  let contents: mut* u8;
  unsafe {
    //allocate 'page' as before
    contents = mem::transmute(page);
  }
  
  JitMemory { contents: contents }
}
{% endhighlight %}

We've got the JitMemory struct which will hold the memory we've allocated, and we use the ```mem::transmute``` call to convert our raw void pointer to a raw u8 pointer, which will be easier to work with in our next step.

Now that we have the struct, let's create some indexing functions.

{% highlight rust %}
use std::ops::{Index, IndexMut};

impl Index<usize> for JitMemory {
    type Output = u8;

    fn index(&self, _index: usize) -> &u8 {
        unsafe {&*self.contents.offset(_index as isize) }
    }
}

impl IndexMut<usize> for JitMemory {
    fn index_mut(&mut self, _index: usize) -> &mut u8 {
        unsafe {&mut *self.contents.offset(_index as isize) }
    }
}
{% endhighlight %}

With these in place, we can now allocate and start writing our instructions into memory.  Before we do so, let's put everything we'd done so far into a constructor for our struct:

{% highlight rust %}
impl JitMemory {
  fn new(num_pages: usize) -> JitMemory {
    let contents : *mut u8;
    unsafe {
      let size = num_pages * PAGE_SIZE;
      let mut _contents : *mut libc::c_void = libc::malloc(size);
      libc::posix_memalign(&mut _contents, PAGE_SIZE, size);
      libc::mprotect(_contents, size, libc::PROT_EXEC | libc::PROT_READ | libc::PROT_WRITE);

      memset(_contents, 0xc3, size);  // for now, prepopulate with 'RET'

      contents = mem::transmute(_contents);
    }

    JitMemory { contents: contents }        
  }
}
{% endhighlight %}

# Writing our first JIT program

With our new constructor and indexing functions in place, let's write our first JIT program.  The simplest "hello world" that I use when doing JIT is a function that takes no parameters and returns a simple value.

We can do this with two assembly instructions:

{% highlight nasm %}
MOV RAX, 0x3  ; move our return value (0x3) into RAX, 
              ; the register in 64-bit used for return values
RET           ; return from the funcation call
{% endhighlight %}

Great, now we just need to write this program into our JIT memory.  But wait, we don't have an assembler :)

Not to fear, there are plenty of assemblers for us to get us started.  There are even [online assemblers](https://defuse.ca/online-x86-assembler.htm) that you can use.  Let's plug the first line ```MOV RAX, 0x3``` in the online assembler.

The top line of the result is the raw hex.  This is what we want.  The actual bits we'll be writing into memory for our function: 48C7C003000000

With these bytes, and the RET instruction we already filled our memory with, we now have our full function.  We can use our indexing functions to write this out:

{% highlight rust %}
let mut jit : JitMemory = JitMemory::new(1);  // allocate a page of memory, like we did earlier

jit[0] = 0x48;  // mov RAX, 0x3
jit[1] = 0xc7;
jit[2] = 0xc0;
jit[3] = 0x03;
jit[4] = 0x00;
jit[5] = 0x00;
jit[6] = 0x00;
{% endhighlight %}

# Turning our memory into a function

We have an executable block of memory with the code for our function filled in.  The last step is to turn this into a Rust function we can call.  We do this by doing another mem::transmute:

{% highlight rust %}
fn run_jit() -> (fn() -> i64) {
  let mut jit : JitMemory = JitMemory::new(1);

  jit[0] = 0x48;  // mov RAX, 0x3
  jit[1] = 0xc7;
  jit[2] = 0xc0;
  jit[3] = 0x03;
  jit[4] = 0x00;
  jit[5] = 0x00;
  jit[6] = 0x00;

  unsafe { mem::transmute(jit.contents) }
}
{% endhighlight %}

And with that, after a few handfuls of unsafe calls and transmutes, we have our function.  The only thing that's left is to call it:

{% highlight rust %}
fn main() {
  let fun = run_jit();
  println!("{}", fun());
}
{% endhighlight %}

# Debugging

When working with a JIT, it's inevitable at some point something will go wrong and we'll need to dive into the debugger.  Luckily, Rust works with LLDB, so we can do just that.

Here are some helpful LLDB commands to get you started:

```lldb target/debug/rustyjit```

Start the JIT in LLDB.

```> br set -line 64```

Set a breakpoint for the line that contains the call to jump into our JIT code

```> run```

Run to our breakpoint.  Now that we're about to run the code, there are a few things we can do:

```> p fun```

Show the pointer address of our function ```fun```.  Unfortunately, that's not the most helpful thing in the world.  What we actually want is look at the memory behind the function.

```> mem read fun```

Which gives us back something like this:

```0x100804000: 48 c7 c0 03 00 00 00 c3 c3 c3 c3 c3 c3 c3 c3 c3  H??....?????????
0x100804010: c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3  ????????????????```

That's better.  Now we can see if the bytes we expect to be there are in fact there.  If we wanted to, we could update memory here using ```mem write``` if something was out of place.

Finally, we can continue with the program:

```> cont```

# Lots ahead

Now that we have a tiny JIT and a way to debug it, the sky is the limit.  Turning source code into machine code that we can run is the heart of any compiler, and with these few additional steps that compiler could be made to output code that we can run directly.

