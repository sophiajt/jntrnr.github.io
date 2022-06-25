+++
title = "Digging into scheme/racket"
[taxonomies]
tags = [ "Racket" ]
+++

I spent some of last night and this morning digging into Racket, the modern version of the venerable Scheme language. Every time I return to a Lisp variant, I find the parentheses less annoying that I did the last time. With proper indenting and some syntax highlighting, is there all that much difference?

```scheme
(define factorial
  (lambda (n)
    (if (= n 0) 1
        (* n (factorial (- n 1))))))
```

Arguably the trailing parentheses are an eye chard.

With the download of Racket you also get DrRacket, which is an updated version of DrScheme, a learning tool aimed at programmers of different experience levels (and to my understanding different ages as well)

I've just begun to experiment with DrRacket to see what it can do. To get familiar with the Scheme style, I've been working through Teach Yourself [Scheme in Fixnum Days](http://www.ccs.neu.edu/home/dorai/t-y-scheme/t-y-scheme-Z-H-1.html).