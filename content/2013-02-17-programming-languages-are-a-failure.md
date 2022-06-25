+++
title = "Programming languages are a failure"
[taxonomies]
tags = [ "languages" ]
+++

After spending years working in programming languages, helping out with some big-name projects and trying to generally make programmers' lives easier, I've decided something.

Programming languages are a failure.

Okay okay, before you write this off as a troll for attention, hear me out for a minute longer.

Calling a programming language a 'language' is false advertising.  Think of what comes to mind when I say 'language'.  A language, as in human-to-human language, has some pretty interesting properties:

 - It can be used to express intent
 - It can transmit information even if one (or even both) of the speakers are non-native

The act of programming is inherently not like speaking.  Imagine having a conversation with someone in the same way you program.

 - Say what you want the other person to understand
 - Say it again, this time with slightly different wording
 - Say it again, with yet more slightly different wording
 - (Rinse, repeat.  Time passes...)
 - Test the other person on what you've said to be very sure they understood
 - Move on to the next thing you want the other person to understand

Of course, the reason we have to do this in programming is because we want what we say to be precise, so the computer exactly what to do, right?  Human language is filled with all that icky imprecision, which is why we use a programming language.  The special kind of language that precisely describes what we mean to the computer, so long as we're patient enough to explain ourselves just so.  

What we've actually created is something akin to legalese for programming rather than legal documents, a sort of programese.  It's a one-way conversation with the computer in a style that removes as much room as possible for misinterpretation.  The dirty secret is that programming languages, like many legal documents, never really finish going through refinement, even after years and years of refinement.

Why not embrace the ambiguity?  Better yet, why not make programming a two-way communication that more closely resembles people talking?

Instead of this:

```
> The bird sits on the pole.
Error: I don't understand 'bird'.  I don't understand 'pole'. 
> The bird, an animal, sits on the telephone pole.
Error: The animal 'bird' can not "sit". 
> The bird, an animal, stands on the telephone pole.
Okay.
```

Why not have this:
```
> The bird sits on the pole.
Question: By "sits" do you mean "stands"? 
> Yes
Understood.
```

Let the programming system ask you when it truly doesn't understand.  The difference is, where it can infer an understanding based on context and background knowledge, it does.

This opens up further possibilities.  What the programming system could be doing for you behind the scenes is fill out what it understands, changing the text much like Telescopic Text.  This would replace "The bird sits on the pole" after you type it in to be a telescopic text  you could click on `"[The bird] [sits on] [the pole]"`.  If you clicked on `[The bird]`, the text would telescope in one level, giving you: `"[The bird, an animal][sits on][the pole]"`. 

Imagine how much easier 
