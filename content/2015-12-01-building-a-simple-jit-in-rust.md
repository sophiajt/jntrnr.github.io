+++
title = "Building a simple JIT in Rust"
[taxonomies]
tags = [ "rust" ]
+++

The other day I threw together a simple Just-In-Time compiler (or JIT, for short), and I thought it'd be fun to show the steps I did.  With this, it should be possible to create a page of executable memory, write some machine code into it, and then treat it like a function call from Rust.

Let's get started!  You can grab my [JIT project](https://github.com/jntrnr/rustyjit), if you want to follow along.  I've only tested this in OS X, but it should adaptable for other platforms.  

**Updated 12/2/2015:** Thanks to some reader comments, I've updated the code below to not use malloc and instead use posix_memalign directly as the allocation step, which avoids leaking memory.

# Creating executable memory

For our JIT, we'll need to first allocate the memory that will hold our JIT'ted code.  The most important thing here is to create a block of memory that's executable so that we can later jump into it, run code, and then return back out.

To do this, we need a few functions from the standard C library, which we can access through the ```libc``` external module.

```rust
// added to Cargo.toml
[dependencies]
libc = "0.2.2"

// main.rs
extern crate libc;
```

Once we've pulled in libc, we need to allocated memory.  Not just any memory, we need to allocate _aligned_ memory.  Some operating systems, like OS X, require that executable code memory starts at a particular alignment.  For example, that the address is exactly a multiple of 0x1000.

```rust
const PAGE_SIZE: usize = 4096;
// ...
unsafe {
  let mut page : *mut libc::c_void = mem::uninitialized();
  libc::posix_memalign(&mut page, PAGE_SIZE, size);
}
```

Once allocated, we now have memory we can safely jump to.  Well, almost.  Before we can run code in our newly-allocated memory, we need to enable executing code in this area of memory.

```rust
unsafe {
  libc::mprotect(page, size, libc::PROT_EXEC | libc::PROT_READ | libc::PROT_WRITE);
}
```

Now we're ready.  We have something we can write into and then jump into.  Since running JIT code is basically "no man's land" without any safeguards, it's easy to get yourself in trouble.  One step that I add is to also fill the memory block with the ```RET``` instruction, which will let us return from our function even if we happen to accidentally run other memory in the block.

```rust
extern {
    fn memset(s: *mut libc::c_void, c: libc::uint32_t, n: libc::size_t) -> *mut libc::c_void;
}
// ...
unsafe {
  memset(page, 0xc3, size);  // prepopulate with 'RET' calls (0xc3)
}
```

We've allocated our JIT memory, aligned it, set it as executable, and then filled it with the ```RET``` instruction.  We're ready to write some code into it.

*Note:* You'll notice we don't explicitly call ```libc::free()``` after we allocate.  For simplicity, we treat the JIT'ed functions we create as long-lived, but a full-blown JIT would likely use better memory hygiene.

# Getting ready to write our first program

To write our first JIT program, let's first make a way to more easily work with our memory.  In its current state, it's a raw C void* pointer, but this is a bit unweildy as we don't have a lot of support to work with that in Rust.  To make something that feels more natural, we'll create a new struct type that will hold our pointer and allow us to create indexing functions for easier access.  This will let us access the memory like ```m[0] = 0x10```.

Let's create the struct we'll use:

```rust
use std::mem;

struct JitMemory {
    contents : *mut u8
}

fn alloc() -> JitMemory {
  let contents: mut* u8;
  unsafe {
    //note: allocate 'page' as before
    
    //then, transmute
    contents = mem::transmute(page);
  }
  
  JitMemory { contents: contents }
}
```

We've got the JitMemory struct which will hold the memory we've allocated, and we use the ```mem::transmute``` call to convert our raw void pointer to a raw u8 pointer, which will be easier to work with in our next step.

Now that we have the struct, let's create some indexing functions.

```rust
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
```

With these in place, we can now more easily write instructions into memory.  Before we do, let's put everything we'd done so far into a constructor for our struct:

```rust
impl JitMemory {
  fn new(num_pages: usize) -> JitMemory {
    let contents : *mut u8;
    unsafe {
      let size = num_pages * PAGE_SIZE;
      let mut _contents : *mut libc::c_void = mem::uninitialized(); // avoid uninitalized warning
      libc::posix_memalign(&mut _contents, PAGE_SIZE, size);
      libc::mprotect(_contents, size, libc::PROT_EXEC | libc::PROT_READ | libc::PROT_WRITE);

      memset(_contents, 0xc3, size);  // for now, prepopulate with 'RET'

      contents = mem::transmute(_contents);
    }

    JitMemory { contents: contents }        
  }
}
```

# Writing our first JIT program

With our new constructor and indexing functions in place, we can write our first JIT program.  The simplest "hello world" that I use when doing JIT is a function that takes no parameters and returns a simple value.

We can do this with two assembly instructions:

```
MOV RAX, 0x3  ; move our return value (0x3) into RAX, 
              ; the register in x64 used for return values
RET           ; return from the function call
```

Great, now we just need to write this program into our JIT memory.  But wait, we don't have an assembler :)

Not to fear, there are plenty of assemblers to get us started.  There are even [online assemblers](https://defuse.ca/online-x86-assembler.htm) you can use.  Let's plug the first line ```MOV RAX, 0x3``` into our online assembler.

The top line of the result is the raw hex.  This is what we want.  These are the actual bits we'll be writing into memory for our function: 48C7C003000000

With these bytes, and the RET instruction we already filled our memory with, we now have our full function.  We can use our indexing functions to write this out:

```rust
let mut jit : JitMemory = JitMemory::new(1);  // allocate a page of memory

jit[0] = 0x48;  // mov RAX, 0x3
jit[1] = 0xc7;
jit[2] = 0xc0;
jit[3] = 0x03;
jit[4] = 0x00;
jit[5] = 0x00;
jit[6] = 0x00;
```

# Turning our memory into a function

We have an executable block of memory with the code for our function filled in.  The last step is to turn this into a Rust function we can call.  We do this by doing another mem::transmute:

```rust
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
```

And with that, after a few handfuls of unsafe calls and transmutes, we have our function.  The only thing left is to call it:

```rust
fn main() {
  let fun = run_jit();
  println!("{}", fun());
}
```

# Debugging

When working with a JIT, it's inevitable at some point something will go wrong, and we'll need to dive into the debugger.  Luckily, Rust works with LLDB out of the box.

Here are some helpful LLDB commands to get you started:

Start the JIT in LLDB:

```lldb target/debug/rustyjit```

Set a breakpoint for the line that contains the call into our JIT code:

```> br set -line 64```

Run to our breakpoint:

```> run```

Now that we're about to run our JIT code, there are a few things we can do.

The first one is to verify the address of our code:

```> p fun```

Unfortunately, that's not the most helpful thing in the world.  What we often want is look at the memory behind the function:

```> mem read fun```

This gives us back something like this:

```
0x100804000: 48 c7 c0 03 00 00 00 c3 c3 c3 c3 c3 c3 c3 c3 c3  H??....?????????
0x100804010: c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3 c3  ????????????????
```

That's better.  Now we can see if the bytes we expect to be there are in fact there.  If we wanted, we could update memory using ```mem write``` if something was out of place.

We can also use the built in disassembler to read back out what the asm instructions are for our function:

```> dis -A x64 -s fun```

Finally, we can continue with the program:

```> cont```

# Lots ahead

Now that we have a tiny JIT and a way to debug it, the sky is the limit.  Turning source code into machine code is the heart of any compiler, and with these few additional steps that compiler could be made to output code that we can run directly.


