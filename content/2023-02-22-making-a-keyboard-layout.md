+++
title = "Making a keyboard layout"
[taxonomies]
tags = [ "typing" ]
+++

It all started innocently enough. I had gotten interested in more efficient typing at first by learning about different keyboards and what they offered. I was intrigued. Except there was one small problem: fancy keyboards are expensive, especially to import into New Zealand.

So, I took a different route: alternate [keyboard layouts](https://paulguerin.medium.com/the-search-for-the-worlds-best-keyboard-layout-98d61b33b8e1).

These promised to help you type better than the standard qwerty layout. I dove in. I tried colemak, workman, and some of their variants. It was clear with practice you could type better with less hand strain. A normal person at this point might have picked something and stuck with it. Well okay, most people wouldn't have even got that far. But I digress.

I on the other hand thought "you know what would be cool? Making a layout for myself!"

And down the rabbit hole I went.

## Why do a new layout?

It started from a simple idea that came from playing guitar and later made its way to professional Tetris competition: [rolling](https://kotaku.com/nes-tetris-players-call-it-rolling-and-theyre-setting-1846767518). The motion you might have used as a kid when you were bored, drumming your fingers on the table. As the fingers fall one after another, you can create high speeds.

The trick seemed simple enough. Lay out the keyboard to try to get the maximum benefit from rolling without making the typing that doesn't roll harder than before.

While it's still too early to declare victory, I'll tell you that the text you are reading now was written using the layout I describe below.

## Creating the layout

I created a few programs with a relatively simple goal. I want to be able to generate layouts that matched a set of constraints. Once I had one, I'd try it out, change the parameters. Rinse and repeat.

In the end, I had gone through hundreds of layouts.

I learned a few things.

### You need to find a corpus of text close to what you type

Look through the data to make sure it will fit your writing style and that common words are covered.

### Be careful about using text with high counts of an uncommon word

When I first started solving for a good keyboard, I used the source for Nushell, a codebase I've spent a lot of time in. This was great in theory, but in practice the algorithm tried to fit words like "value" and "span", pushing other words out of the way.

### Your hands might help you find what works for you

I would try out an idea and often revise after seeing how that idea worked in practice. I tried many shapes to roll and over time simplified it down to a set that felt I could hit with high enough accuracy.

### It is easy to over-fit your data

Scoring the different moves ended up being a bit of an art. Setting scores high enough, but not too high, took many iterations of trial and error.

## Show us the layout already

![Keyboard with pmfcqxluoy inasbkreh ;vgdzjw,.](https://www.jntrnr.com/images/keyboard_layout.png)
Inas keyboard layout

Here's the Inas layout (named by the home keys on the left hand).

Let's talk about how I scored typing movements, and then talk about some ways we might improve on it.

## Scoring

I broke my corpus of text up into words, then found 1-, 2-, and 3-letter runs of letters. These runs would then make up the "roll" that would get higher points.

I used the following scoring:

* Typing on the home row is preferred
* Center row keys are to be avoided (I learned from guitar that the lift of the finger can be easier on it than lateral stretches)
* Runs of two should be together where possible
* Jumps between the top row and bottom row should be avoided
* Runs of three can fall in a row or can be two in a row and then fall by one row (in the code I call this a j-roll)
* Try to use the pinkies less (unless necessary for the above)

If you look at the keyboard above you can see quite a few patterns that fall out from that:

* Common three letter runs like "ing", "you", "the", and "can" are easier to type.
* Longer words that build on runs are also easier: "there", "this", "think", "thing", "your", "than", "care", "other", "another", and so on. Mixing and matching runs while you type ends up being a fun puzzle.
* Surprisingly, some common words like "and" don't run. I think this comes from how important "ing" is in English. This takes precedence over the "n" key for rolls.

## But is it good?

I've been working with this keyboard layout from the perspective of potential speed. It's still too soon to know how fast you can get with it.

There are also other considerations. Namely, does this help your hands? Layouts like qwerty over time can hurt your hands. Does Inas help at all with it? The jury is still out. One good iteration on what I have so far would be to score movements that are more ergonomic higher. Inas has some of that on a basic level by preferring smaller movements and preferring the home row. Are there more ergonomic patters to include? Undoubtedly.

If you give it a try, let me know how it goes for you.

## Build your own

If you want to improve what I have or build your own keyboard you can [try it out for yourself](https://github.com/jntrnr/create_keyboard_layouts)

