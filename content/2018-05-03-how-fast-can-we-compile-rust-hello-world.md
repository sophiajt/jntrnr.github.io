+++
title = "How fast can we compile Rust hello world?"
[taxonomies]
tags = [ "rust" ]
+++

Seeing [Nick Nethercote's blog post about speeding up the compiler](https://blog.mozilla.org/nnethercote/2018/04/30/how-to-speed-up-the-rust-compiler-in-2018/), I started wondering just how fast could a Rust compiler be?  How fast could we compile a simple example?  How fast can we compile a Rust hello world?

## Starting out

When you do a `cargo new hello_rust --bin`, you get a simple Rust hello world:

```rust
fn main() {
    println!("Hello, world!");
}
```

Awesome.  Let's see how long this takes to build:

```term
> time cargo build
   Compiling hello_rust v0.1.0 (file:///home/jonathan/Source/hello_rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52 secs

real     0m0.548s
user     0m0.468s
sys      0m0.080s
```

Half a second on my XPS 13 9360, with a i7-7500U @ 2.70GHz with 16 gigs of RAM.  That seems, a bit long for a hello world.  Maybe cargo is doing extra work?

```term
> time rustc src/main.rs

real     0m0.378s
user     0m0.289s
sys      0m0.083s
```

That's definitely better.  We're just over 1/3rd of a second.  

What should we expect?  If only there were another language we could compile a hello world for that's comparable... 

```term
> cat hello_c.c
#include <stdio.h>

int main() {
    puts("Hello, from C!");
    return 0;
}
> time clang hello_c.c

real     0m0.082s
user     0m0.021s
sys      0m0.041s
```

Woooaah!  Wait a second.  You're telling me that compiling C hello world is over 4.5x faster.  That's... that's something.  Maybe the compiler is doing a bit too much work for this small example?  I mean, we don't need much from the standard library, just the println macro.

You know, a reasonable person would probably stop here and walk away.  Chalk it up to "Rust is more complicated" or "maybe if I knew the right commandline options"

No, that's not us.  Not today.  Today, we're going to write our own Rust translator.  We're going to see just how fast we can compile Rust to a working hello world.

## First stop: let's build a Rust->C translator.

Here's my thinking: if we write our own translator, we can translate the Rust to C ourselves, and then compile the C.  That way, we should end up with a compile time close to the C.

```term
> cargo new rabbithole --bin
```

Writing a full compiler is complex work, but we're going for speed here.  And speed means cutting corners.  To that end, we're implementing *just enough* of a compiler to translate our Rust to C, then we're calling that good enough.

First step, we need a parser.  Luckily, there already is a crate that can parse Rust code for us.

```
[dependencies]
syn = {version = "0.13", features = ["full", "extra-traits"] }
```

```rust
extern crate syn;

use std::env;
use std::fs::File;
use std::io::Read;

fn main() {
    let mut args = env::args();
    args.next();

    match args.next() {
        Some(fname) => {
            let mut src = String::new();
            let mut file = File::open(&fname).expect("Unable to open file");
            file.read_to_string(&mut src).expect("Unable to read file");

            let syntax = syn::parse_file(&src).expect("Unable to parse file");
            println!("{:#?}", syntax);
        }
        _ => {
            println!("Please supply the file to compile");
        }
    }
}
```

```term
> cargo run -- hello.rs
File {
    shebang: None,
    attrs: [],
    items: [
        Fn(
            ItemFn {
                attrs: [],
                vis: Inherited,
                constness: None,
                unsafety: None,
                abi: None,
                ident: Ident {
                    term: Term {
                        sym: main
                    }
                },
                decl: FnDecl {
                    fn_token: Fn,
                    generics: Generics {
                        lt_token: None,
                        params: [],
                        gt_token: None,
                        where_clause: None
                    },
                    paren_token: Paren,
                    inputs: [],
                    variadic: None,
                    output: Default
                },
                block: Block {
                    brace_token: Brace,
                    stmts: [
                        Item(
                            Macro(
                                ItemMacro {
                                    attrs: [],
                                    ident: None,
                                    mac: Macro {
                                        path: Path {
                                            leading_colon: None,
                                            segments: [
                                                PathSegment {
                                                    ident: Ident {
                                                        term: Term {
                                                            sym: println
                                                        }
                                                    },
                                                    arguments: None
                                                }
                                            ]
                                        },
                                        bang_token: Bang,
                                        delimiter: Paren(
                                            Paren
                                        ),
                                        tts: TokenStream [
                                            Literal {
                                                lit: "Hello, world!"
                                            }
                                        ]
                                    },
                                    semi_token: Some(
                                        Semi
                                    )
                                }
                            )
                        )
                    ]
                }
            }
        )
    ]
}
```

This might look a bit intimidating, so let's break it down.  The top-most thing is a File.

* File

Each File continues zero or more Items.  An Item are things like declarations.  Function declarations, for example, are one kind of Item.

* File
  * Item

Since this file has one function, it has one item, an ItemFn.  This holds all the information about our main function.  You can see in here that we have the function's declaration (sometimes called the function prototype, if you come from C).  This tells us how many parameters it takes, what its return type is, etc.  Luckily for us, this is all empty because main has nothing interesting here.  This lets us focus on the body of the function, here called 'block'

* File
  * Item (ItemFn)
    * Block

Blocks contain statements and expressions.  This is really the meat of what actually *does* things, and which is where we'll actually spend our time writing the compiler.  Let's take a closer look:

```
block: Block {
    brace_token: Brace,
    stmts: [
        Item(
            Macro(
                ItemMacro {
                    attrs: [],
                    ident: None,
                    mac: Macro {
                        path: Path {
                            leading_colon: None,
                            segments: [
                                PathSegment {
                                    ident: Ident {
                                        term: Term {
                                            sym: println
                                        }
                                    },
                                    arguments: None
                                }
                            ]
                        },
                        bang_token: Bang,
                        delimiter: Paren(
                            Paren
                        ),
                        tts: TokenStream [
                            Literal {
                                lit: "Hello, world!"
                            }
                        ]
                    },
                    semi_token: Some(
                        Semi
                    )
                }
            )
        )
    ]
}
```

On closer inspection we see this block has only one statement.  A something-something macro something or other.  Oh right, println is a macro!  Let's focus on just the macro invocation itself:

```
mac: Macro {
    path: Path {
        leading_colon: None,
        segments: [
            PathSegment {
                ident: Ident {
                    term: Term {
                        sym: println
                    }
                },
                arguments: None
            }
        ]
    },
    bang_token: Bang,
    delimiter: Paren(
        Paren
    ),
    tts: TokenStream [
        Literal {
            lit: "Hello, world!"
        }
    ]
},
```

Now this is a little more like it.  It's still a little cumbersome, but we can see the parts we need.  There's a `sym` thing that says what the macro is.  Later on, we can see the string that we're printing is a literal string.  We can ignore the rest.

That's it.  Make sure it's invoking println, and then we'll look for the string that's being printed.

To make things a little easier on ourselves, let's make a quick-and-dirty intermediate representation.  This will only have one command so far, but it'll be an easy way to de-couple the function that does the parsing from the one that does the code generation.

```rust
enum Command {
    PrintLn(String)
}
```

With that, we're ready to make our parser that can take in the `syn::File` and output our Command.  There are a few ways we can do this, but to keep things pretty simple, I'm going to make a separate `parse` function for each layer we talked about earlier: File, Item, and Block.  You can think of this as us unpacking the useful information from what `syn` gives us into much simpler data structures.

```rust
extern crate syn;

use std::env;
use std::fs;
use std::io::Read;

#[derive(Debug)]
struct File {
    functions: Vec<Function>,
}

#[derive(Debug)]
struct Function {
    body: Vec<Command>,
}

#[derive(Debug)]
enum Command {
    PrintLn(String),
}

fn parse_body(body: &syn::Block) -> Result<Vec<Command>, String> {
    let mut stmts = vec![];
    for stmt in &body.stmts {
        match stmt {
            syn::Stmt::Item(syn::Item::Macro(ref im)) => {
                let macro_name = im.mac.path.segments[0].ident.as_ref();
                if macro_name == "println" {
                    match im.mac.tts.clone().into_iter().next() {
                        Some(ref arg) => {
                            stmts.push(Command::PrintLn(arg.to_string()));
                        }
                        None => return Err("Expected argument in function".into()),
                    }
                } else {
                    return Err(format!("Unknown macro: {}", macro_name));
                }
            }
            _ => return Err("Unexpected statement in function".into()),
        }
    }
    Ok(stmts)
}

fn parse_fn(item: &syn::ItemFn) -> Result<Function, String> {
    Ok(Function {
        body: parse_body(&item.block)?,
    })
}

fn parse_file(file: &syn::File) -> Result<File, String> {
    let mut functions = vec![];
    for item in &file.items {
        match item {
            syn::Item::Fn(ref item_fn) => {
                functions.push(parse_fn(item_fn)?);
            }
            _ => return Err("Unexpected item in file".into()),
        }
    }
    Ok(File { functions })
}

fn main() {
    let mut args = env::args();
    args.next();

    match args.next() {
        Some(fname) => {
            let mut src = String::new();
            let mut file = fs::File::open(&fname).expect("Unable to open file");
            file.read_to_string(&mut src).expect("Unable to read file");

            let syntax = syn::parse_file(&src).expect("Unable to parse file");
            let result = parse_file(&syntax);

            println!("{:#?}", result);
        }
        _ => {
            println!("Please supply the file to compile");
        }
    }
}
```

Which results in:

```term
Ok(
    File {
        functions: [
            Function {
                body: [
                    PrintLn(
                        "\"Hello, world!\""
                    )
                ]
            }
        ]
    }
)
```

That's much nicer!  It took a little work reading the `syn` documentation, but once we got the hang of it, we can now work with real Rust source code and turn it into something we can use.

Oh, before we go, let's clean one thing up.  You might have noticed that extra set of quotes in the macro argument.  No problem, we'll just turn:

```rust
stmts.push(Command::PrintLn(arg.to_string()));
```

Into:

```rust
stmts.push(Command::PrintLn(arg.to_string().replace("\"", "")));
```

Which gives us:

```term
Ok(
    File {
        functions: [
            Function {
                body: [
                    PrintLn(
                        "Hello, world!"
                    )
                ]
            }
        ]
    }
)
```

Beautiful.  Now that we have it in this form, we have a much simpler form that we can hand to the other half of the translator.  In this half, we'll be outputting the C code and compiling it.

Since we only care about one function in one file, let's just make a quick-and-dirty function to generate the C:

```rust
fn compile_to_c(file: &File) -> String {
    let mut c_output = String::new();

    c_output += "#include <stdio.h>\n";
    c_output += "int main() {\n";

    for stmt in &file.functions[0].body {
        match stmt {
            Command::PrintLn(ref msg) => {
                c_output += &format!("puts(\"{}\");\n", msg);
            }
        }
    }

    c_output += "}\n";
    c_output
}
```

That's it.  Let's call it.

```rust
println!("{}", compile_to_c(&result.unwrap()));
```

Running it again, we get:

```term
#include <stdio.h>
int main() {
puts("Hello, world!");
}
```

Woah!  That looks like C to me.  Dare we try to compile it?

```term
> cargo run -- ../hello_rust/src/main.rs  > testme.c
> clang testme.c
> ./a.out
Hello, world!
```

It... works!  We just made a compiler!

```term
> time ./target/release/rabbithole ../hello_rust/src/main.rs > testme.c

real    0m0.014s
user    0m0.010s
sys    0m0.004s
> time clang testme.c -o testme
real    0m0.100s
user    0m0.043s
sys    0m0.041s
```

Nice, by cutting lots of corners and making our own translator, we got the compile times down from 0.378s to 0.114s, less than 1/3rd of where we were.

Should we stop there?  Surely we can go further down the rabbit hole.  Oh right, C is not the lowest we can go

## Deeper down the rabbit hole

Let's cook up some assembly.  First, let's test how fast we can assemble a hello world in assembly.

```term
> cat hello_asm.s
 .text
 .global main
main:
 mov $hw_str, %rdi
 call puts
 mov $0, %rax
 ret

 .data
hw_str:
 .asciz "hello asm"
> time as hello_asm.s -o hello_asm.o
real    0m0.019s
user    0m0.008s
sys    0m0.011s
> time ld -o hello -dynamic-linker /lib64/ld-linux-x86-64.so.2 /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/x86_64-linux-gnu/crti.o -lc hello_asm.o /usr/lib/x86_64-linux-gnu/crtn.o

real    0m0.044s
user    0m0.036s
sys    0m0.008s
```

Wow, the assembly output cuts the compile almost in half!  If you haven't seen x64 assembly before, that might look a little strange.  Instead of working with C code with functions, we've dropped down much closer to the machine itself.  The above represents the lowest we can go in Linux.  Let's unpack what's happening:


```asm
 .text
 .global main
```

Our assembly file starts with two things.  One, a section that we'll be talking about the code itself (.text) and then the one symbol we'll be making visible (main).  This will be the main function that the linker will use later.

Next, main begins:

```asm
main:
```

What follows are series of assembly commands.  The hardware keeps numbers and addresses in registers, similar to how you might store them in variables.

```asm
mov $hw_str, %rdi
call puts
```

With that in mind, this doesn't seem quite so bad.  The first line is going to set up our register that we'll use as the argument to the puts function (here, in Linux, %rdi).  After the argument is ready, we call puts.

```asm
 mov $0, %rax
 ret
```

After that, we return 0, just like we did in C.  Here, in 64-bit assembly, we're returning the return value in %rax.  

That's it.  Oh wait, one more thing.  So this was the code section, we still haven't told it what the string is to print (which we called $hw_str earlier).

```asm
 .data
hw_str:
 .asciz "hello asm"
```

That's not bad.  We have a data section that contains our strings.  We have one string, called `hw_str`, and we give it a null-terminated string to print.

You may have been wondering about our linker line, too:

```term
> time ld -o hello -dynamic-linker /lib64/ld-linux-x86-64.so.2 /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/x86_64-linux-gnu/crti.o -lc hello_asm.o /usr/lib/x86_64-linux-gnu/crtn.o
```

This bit of magic is actually the kind of tasks that your C compiler does for you behind the scenes.  In Linux, for a program to run, it needs to be able to access things like the Linux program loader, the C standard library, and the Linux subsystems.  The above takes the barebones object file we made with the assembler and gives it all it needs to be a complete Linux executable.

Before we write our assembly-outputting function, let's take a quick pause and talk about Windows.  We've been giving a lot of examples for Linux, but you can do the same kinds of things in Windows.  Make sure you have a 64-bit [nasm](https://www.nasm.us/) installed.

```term
C:\Users\Jonathan\source\fast_hello>type nasm_hello.asm
    global main
    extern puts

    section .text
main:
    push rbp
    mov rbp, rsp
    mov rcx, message
    call puts
    mov rax, 0
    pop rbp
    ret
message:
    db 'Hello, Rust!', 0
C:\Users\Jonathan\source\fast_hello>nasm -f win64 nasm_hello.asm -g -o nasm_hello.obj

C:\Users\Jonathan\source\fast_hello>link -out:b.exe -defaultlib:libcmt "-libpath:C:\\Program Files (x86)\\Microsoft Visual Studio 14.0\\VC\\lib\\amd64" "-libpath:C:\\Program Files (x86)\\Windows Kits\\10\\Lib\\10.0.15063.0\\ucrt\\x64" "-libpath:C:\\Program Files (x86)\\Windows Kits\\10\\Lib\\10.0.15063.0\\um\\x64" -nologo nasm_hello.obj
```

You can see that the assembly looks different, though it's similar.  We do a little more setup, but the core of the work is the same.  We set up where the string is, we send it to the `puts` function, and we return 0 via the `rax` register.  Likewise, after we have an object file from the assembler, we have to do a similar dance with the linker in order to have a fully working Windows executable.

With that, we can see all the techniques we're using in Linux can work in Windows, too.

Okay, let's finish this up and see where we are.  Just like we wrote a C output function, we're going to write an assembly output function.  This function will create two strings: one for the code and one for the data, and then stitch them together.

```rust
fn compile_to_asm(file: &File) -> String {
    let mut asm_code_output = String::new();
    let mut asm_data_output = String::new();

    asm_data_output += ".data\n";

    asm_code_output += ".text\n";
    asm_code_output += ".global main\n";
    asm_code_output += "main:\n";

    let mut temp_num = 0;
    for stmt in &file.functions[0].body {
        match stmt {
            Command::PrintLn(ref msg) => {
                asm_code_output += &format!("mov $str_{}, %edi\n", temp_num);
                asm_code_output += "call puts\n";

                asm_data_output += &format!("str_{}:\n", temp_num);
                asm_data_output += &format!(" .asciz \"{}\"", msg);
                temp_num += 1;
            }
        }
    }

    asm_code_output + &asm_data_output
}
```

```term
> time ./target/release/rabbithole ../hello_rust/src/main.rs > hello_asm.s

real    0m0.015s
user    0m0.010s
sys    0m0.005s
> time as hello_asm.s -o hello_asm.o
real    0m0.022s
user    0m0.009s
sys    0m0.012s
> time ld -o hello -dynamic-linker /lib64/ld-linux-x86-64.so.2 /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/x86_64-linux-gnu/crti.o -lc hello_asm.o /usr/lib/x86_64-linux-gnu/crtn.o

real    0m0.043s
user    0m0.029s
sys    0m0.011s
```

From 0.114s to 0.08s.  We dropped another 30% off the compile time!

To recap, we started at 0.548s with Cargo, then 0.378s with rustc, 0.114 going via C, and 0.08 going via assembly.  Roughly a 6.8x speedup.  Can we squeeze anything else out?  

You might be wondering about keeping more things in memory and getting a bit more caching.  It's a good hunch, let's try it!

```term
> time (./target/release/rabbithole ../hello_rust/src/main.rs > hello_asm.s && as hello_asm.s -o hello_asm.o && ld -o hello -dynamic-linker /lib64/ld-linux-x86-64.so.2 /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/x86_64-linux-gnu/crti.o -lc hello_asm.o /usr/lib/x86_64-linux-gnu/crtn.o)

real    0m0.048s
user    0m0.028s
sys    0m0.021s
```

That's another 40% off the compile time.  We've now gone from 0.548s to 0.048s, a whole 11x faster.  There's even more we can do.  For example, we can squeeze out more by writing our own parser, outputting the .o ourselves, and more.  But we'll leave that as an exercise for the reader.

## Conclusion

I hope you had as much fun reading this post as I had in writing it.  There are amazing benefits to all the features that cargo and rustc provide, and the Rust ecosystem builds on those benefits.  At the same time, it's fun to play around with compiler technology and see that when we get down to it, it's just another Rust program that we can tune to our heart's content.  

You can see the [rabbithole source](https://github.com/jonathandturner/rabbithole) if you want to play with it yourself.  