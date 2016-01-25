---
layout: post
title:  "Elixir: Learning Recursion with The Little Schemer"
date:   2016-01-17 19:13:00
categories: Elixir
comments: true
---

> Hey there - 
>
> If you're new to Elixir and recursive thinking, then you might
> want to check out The Little Schemer. 
>
> If you're not familiar with the book and you'd like to improve your skills with recursion,
> then the video and information below might be valuable to you.
>
> I talk briefly about some misconceptions that people have with the book, and how
> you might use it as an Elixir developer.
>
> -- Jim

<iframe src="https://player.vimeo.com/video/152112914" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="https://vimeo.com/152112914">The Little Schemer for Elixir developers: What&#039;s it all about?</a> from <a href="https://vimeo.com/user29282688">Jim Whiteman</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Notes

**1**. If you ask Reddit or Twitter for advice on how to best learn recursion, the most consistent good advice you'll
get is to work through the <a href="http://www.amazon.com/Little-Schemer-Daniel-P-Friedman/dp/0262560992" target="_blank">Little Schemer</a>.

Unfortunately, it's advice that very few people take.

**2**. Many people have the misunderstanding that the Little Schemer is somehow a _Scheme_ book.
And they can hardly be blamed for thinking that: it's got Scheme in title, after all.

But The Little Schemer is a Scheme book in the same way that the <a href="http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612/" target="_blank">Gang of Four Design Patterns Book</a> is _a C++ book_. Which is to say, it's not that type of book at all.

If you're worried that working through the book will be tantamount to "learning a new language" and therfore
too arduous or time consuming to be worth it, then you have nothing to worry about:

At the end of the book, you still won't know how to print "hello, world" to stdout. The point being that learning recursion is 100% the focus of the book, and not the language of Scheme.

**3**. The Little Schemer is much closer to a patterns book than a language book. A better name for the book might
have been "The Little Book on Recursion" or "The Patterns of Recursive Algorithms".

**4**. If you want to get up and running with recursion as quickly as possible, then buy the print version and
work through the first 5 or 6 chapters.

For each exercise, write out a solution first in Scheme and then in Elixir second.

If you love the book and are feeling confident, then you can try to tackle the whole book.

**5**. The Little Schemer is a workbook. If you're unfamiliar with the material, then simply reading through it
will probably not help you that much.

**6**. The book is written in a socratic style. If you ever imagined how Yoda might teach Luke recursion, or how
 <a href="http://www.centare.com/wp-content/uploads/2015/01/Daniel-Larusso-Ralph-Macchio.jpg" target="_blank">Mr. Miyagi might teach Daniel-san</a> computation, then you may enjoy the book. It will help if you are slightly masochistic though.

**7**. The Little Schemer starts very simply and ramps up the diffuclty quite rapidly.

This is more or less what it feels like to work through chapter 9 for the first time:

[![Chapter 9](/assets/yodaluke.gif)](/assets/yodaluke.gif)

(NOTE: It looks like Yoda had his own version of the Y-combinator. And it's only marginally less difficult than the one in the book.)

**8**. <a href="http://racket-lang.org/new-name.html" target="_blank">Scheme was recently renamed to Racket</a>, but that shouldn't change anything for you.

You can download Racket <a href="http://racket-lang.org/" target="_blank">here</a>. Just run the Scheme
code you write as Racket and everything should still work without changing anything else.

**9**. If you work through every exercise in the book, you'll still need to investigate how <a href="https://en.wikipedia.org/wiki/Tail_call" target="_blank">Tail Recursion</a> works on your own. Almost all the algorithms you'll
learn in the Schemer series will be body recursive. Although, that's <a href="http://www.erlang.org/doc/efficiency_guide/myths.html" target="_blank">not necessarily the end of the world</a>.

**10**. If you're really ambtious, or you can't be bothered to do the exercises in the book, then you
can find a PDF of additional exercises <a href="http://www.ccs.neu.edu/home/matthias/BTLS/exercises.pdf" target="_blank">here</a>.

**11**.  Shameless plug - you can find <a href="https://github.com/jwhiteman/a-little-elixir-goes-a-long-way" target="_blank">all of the exercises to The Little Schemer written in Elixir here</a>. Quickly browse through the code to see what you'll learn if you work through the book.

Don't spend too much time with it though. Just skim it and refer back to it if you get stuck.

If you don't want to use Dr. Racket, you might be able to use <a href="https://github.com/jwhiteman/lighthouse-scheme" target="_blank">my version of Scheme instead</a>.

It's not a production ready Scheme (and never will be), but it will run all of the examples in the book,
and it's also written in Elixir.

