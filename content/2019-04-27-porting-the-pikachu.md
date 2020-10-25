+++
title = "Creating crossplatform Rust terminal apps"
[taxonomies]
tags = [ "rust" ]
+++

![Pikachu animation in Windows](/images/pikachu.jpg)

_Look Mom, Pikachu running in Windows CMD!_

I've been wanting to play around with the cool [spinning Pikachu demo](https://www.reddit.com/r/rust/comments/bf5uzr/update_on_my_3d_ascii_art_generator_termiontobj/) everyone was talking about. Sadly, it used termion to do its magic, which meant that unfortunately it wouldn't work for me.

Termion has been a boon for Rust, with lots of folks using it to create terminal applications. Unfortunately, as a Windows user, I know there's a good chance that if the crate depends on termion that's the end of the line for me, as termion apps just don't work in Windows. Surely, I thought, there must be a better way, but I never managed to find one.

# Enter crossterm

Yesterday, I discovered a crate I have wanted for at least a year: that holy grail of something like termion that also worked crossplatform. The crate is called [crossterm](https://github.com/TimonPost/crossterm), and it is exactly that. Rather than being an exact copy of termion, I'd call Crossterm a termion-like crate. The API surface feels very similar between the two crates, but it's not identical.  That said, often, the translation is 1:1 (or close to it).

In this post, I'll detail what I had to do to port to crossterm. You can take a look at my [repo](https://github.com/jonathandturner/rust-sloth/tree/crossterm-port) and give it a spin.

## Moving to crossterm

Like termion, crossterm is a single crate to depend on that pulls in what you need for interacting with the terminal:

```
-termion = "1"
+crossterm = "0.9.3"
```

Next, is setting up the terminal:

```
-    let mut stdout = AlternateScreen::from(stdout().into_raw_mode().unwrap()); // Raw output is clean output
-    let mut stdin = async_stdin().bytes(); // Async in so input isn't blocking
+
+    let crossterm = Crossterm::new();
+    #[allow(unused)]
+    let screen = RawScreen::into_raw_mode();
+    let input = crossterm.input();
+    let mut stdin = input.read_async();
+    let cursor = cursor();
+
+    cursor.hide()?;
```

There are a few more steps here, and I took a slightly different approach than termion. In the original, the author uses an AlternateScreen, a way to easily switch back to the previous terminal with the application exits. The crossterm package supports this, though there were is an [issue](https://github.com/TimonPost/crossterm/issues/128) that's being worked on. Using an AlternateScreen is optional, so I just left it off.

The next step was to set up raw mode. Raw mode lets us get user input directly without the normal terminal buffering. The order of operations in crossterm above is important.

Lastly, I hide the cursor so it doesn't mess up the animation.

The event loop is similar, though a little more verbose in my version:
```
-        let b = stdin.next();
-        if let Some(Ok(b'q')) = b {
-            break;
+        if let Some(b) = stdin.next() {
+            match b {
+                InputEvent::Keyboard(event) => match event {
+                    KeyEvent::Char('q') => break,
+                    _ => {}
+                },
+                _ => {}
+            }
```

The event systems between termion and crossterm, while similar, are different enough that I preferred to do two-step match to make sure I understood it. Events for both key and mouse come in through the event stream, which is a nice benefit.

Drawing colors to the screen is also very similar between termion and crossterm:

```
-    format!("{}{}{}{}", color::Fg(color::Rgb(bg.0, bg.1, bg.2)),
-                        color::Bg(color::Rgb(25,25,25)),
-                      src,
-                      color::Fg(color::Reset))
+                    print!(
+                        "{}{}{}",
+                        Colored::Fg(Color::Rgb {
+                            r: (pixel.1).0,
+                            g: (pixel.1).1,
+                            b: (pixel.1).2
+                        }),^M
+                        Colored::Bg(Color::Rgb {
+                            r: 25,
+                            g: 25,
+                            b: 25
+                        }),
+                        pixel.0
+                    )
```

The change from `color::Fg` to `Colored::Fg` is straightforward enough I did most of these changes with search/replace.

# Gotchas

The translation is not without its own set of hiccups. The main one of which you get a hint at here:

```
-    pub frame_buffer: Vec<String>,
+    pub frame_buffer: Vec<(char, (u8, u8, u8))>,
```

Originally, the whole framebuffer was turned into a vector of strings. On most Unix platforms, these would include the ANSI commands to change the color or move the cursor. It's a powerful tool, and being text-based, you can prebuffer the whole frame via string manipulation.

Unfortunately, assuming everything is string-based doesn't work in Windows. When crossterm outputs using println!, it's calling into the Windows Console API between each piece of the println. This allows the same code to work, assuming it doesn't buffer the string.

To work around this, I stopped buffering the string and instead stored the pixel value in the buffer. When it was time, I drew the whole frame using print! statements.

When I first did this, it was noticeably slower than the termion version(roughly 5-10 fps). This is because calling into the Console API that often (once per character) is going to pull down performance.  Luckily, I could work around this by just checking if we were already using the color I wanted to render. If we were, I didn't set the color again. 

Performance between the termion version and the crossterm version now look the same to me, which I call good enough :)

# Conclusion

We have a way to move more of our terminal apps to being crossplatform, so that they work out-of-the-box on Windows as well as Unix-based systems. It was fun to write the port, and I did the port in a couple hours without ever having used crossterm before. 

Also, big thanks to [Mitchell Hynes](https://github.com/ecumene-software) for writing the original!
