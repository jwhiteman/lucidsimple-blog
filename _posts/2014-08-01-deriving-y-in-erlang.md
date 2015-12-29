---
layout: post
title:  "Deriving Y (in Erlang & Ruby)"
date:   2014-08-01 08:00:00
categories: Erlang
comments: true
---

### Understanding Y

Let's talk about the Y-Combinator. If you're not hip to what that is, here is a challenge
for you:

Try to write an anonymous recursive function.

Because "anonymous AND recursive" might not be immediately clear, let's go over both
one at a time so that there is no confusion.

The first requirement is 'anonymous'. Here are some anonymous functions (henceforth to be called `lambdas`) in a couple
popular programming languages:


{% highlight ruby %}
# Ruby
lambda { ... }
{% endhighlight %}

{% highlight erlang %}
%% Erlang
fun() -> ... end.
{% endhighlight %}

{% highlight javascript %}
// Javascript
function() { ... };
{% endhighlight %}

Fine. That's easy enough. But what about the recursive part? Let's start with a "regularly defined"
recursive function as a point of reference.
Factorial, which is almost always used in these types of tutorials, will do the trick.

{% highlight ruby %}
# Ruby
def fac(n)
  if n.zero?
    1
  else
    n * fac(n-1)
  end
end

# fac(10)
# => 3628800
{% endhighlight %}

{% highlight erlang %}
%% Erlang.
fac(0) -> 1;
fac(N) -> N * fac(N-1).

%% fac(10).
%% => 3628800
{% endhighlight %}

Great. Easy enough if you have some experience with recursion. If these
examples don't seem easy, then stop this tutorial and work through [The Little Schemer][lil-schemer].
It helped me understand recursion much better. It's a great book.

Still here? OK - let's carry on. The final step is to make a lambda version of factorial,
that's also recursive.

Most people will try to do something like this:

{% highlight ruby %}
#ruby
fac = lambda do |n|    # Setting a local variable called fac here...
        if n.zero?
          1
        else
          n * fac.(n-1) # and then trying to call it down here.
        end
      end
{% endhighlight %}

{% highlight erlang %}
%% Erlang.
Fac = fun(0) -> 1;                 %% Matching against Fac here...
         (N) -> N * Fac(N-1) end.  %% and then trying to use here.
{% endhighlight %}

It's a nice try, but it's a little bit cheap because it's not 100% anonymous. And the Erlang version doesn't even work.

Bummer. Back to the drawing board. Hmm.

What do we need? Well, at the point of recursion, factorial needs to be able to
call itself. We know that much. Would it be possible to pass a copy of factorial
into itself? What would that even look like? Would this work?

{% highlight ruby %}
#ruby
fac = lambda do |f|
        lambda do |n|
          if n.zero?
            1
          else
            n * f.(n-1)
          end
        end
      end

fac.(fac)
{% endhighlight %}

{% highlight erlang %}
%% Erlang.
Fac = fun(F) ->
        fun(0) -> 1;
           (N) -> N * F(N-1) end
      end.

Fac(Fac).
{% endhighlight %}

Before you get too weireded out here, we're not really doing anything too different
except for 2 things.

One, we've wrapped our old example in another lambda
( `lambda do |f|` in ruby and `fun(F)` in erlang). Two, we're calling `Fac(Fac)` (in erlang)
and `fac.(fac)` (in Ruby).

Initially, seeing something like `Fac(Fac)` or `fac.(fac)` or any other example of a function
_taking itself as an argument_ is a bit of a mind stretcher. It's too meta! Your mind will melt
as your inner mental-stack grows out of control...

It's actually not that hard. Let's think if it like this... You see something like G(H), where
you know that both G and H are functions. You could read it like this:

"I'm a function and my name is G. When I get to the point that I need help, I call the function H."

How would you read G(K)?

"I'm a function and my name is G. When I get to the point that I need help, I call the function K."

How would you read Fac(Z)?

"I'm a function and my name is Fac. When I get to the point that I need help, I call the function Z."

So, finally, what is Fac(Fac) ?

"I'm a function and my name is Fac. When I get to the point that I need help, I call the function Fac."

Which is basically another way of saying, "I'm a recursive function".

When you see Fac(Fac) or F(F), etc etc. Repeat the magic sentence and then think: we're just
building a recursive function.

You could see F(F) and say: this isn't a big deal. This guy is just building a recursive function here.
F(K) or F(G) or F(Z) might have been some other type of function. But we want to build a recursive one,
so F(F) will build what we want.

So...back to our examples. Do they work? Walk through them (the Ruby if you're a Rubyist, the Erlang
if you're more comfortable with Erlang). Try using both 0 and 1 for values of n.

For example, what is `fac.(fac).(0)` and `fac.(fac).(1)`?

Or in Erlang, what is `((Fac(Fac))(0).` and `((Fac(Fac))(0).`?

Do they work? No.

0 will work. But 1 won't work. We need to make an adjustment.

{% highlight ruby %}
#ruby
fac = lambda do |f|
        lambda do |n|
          if n.zero?
            1
          else
            n * f.(f).(n-1)
          end
        end
      end

# fac.(fac).(10)
# => 3628800
{% endhighlight %}

{% highlight erlang %}
%% Erlang.
Fac = fun(F) ->
        fun(0) -> 1;
           (N) -> N * (F(F))(N-1) end
      end.

%% (Fac(Fac))(10).
%% => 3628800
{% endhighlight %}

What did we do? We needed to change `f.(n-1)` to `f.(f).(n-1)` in Ruby and
`F(N-1)` to `(F(F))(N-1)` in Erlang. Why? Think back to this:

"I'm a function and my name is F. When I get to the point that I need help, I call the function F".
F isn't ready by itself to take N (an integer). F(N) won't work. F forever and always needs a helper
function, and because we want it to be recursive, that will take the form of F(F).

So we need to change F(N) to be (F(F))(N). F(F) is our recursive function builder, and
we were hoping to call a recursive function.

If you understand this, you literally have everything you need to understand the Y-combinator.
The only thing you need to get the Y-combinator is to _refactor_ the example above. Literally,
and to re-iterate: the Y-combinator is just a refactoring of these final two examples.

So If you understand this, then please proceed. If you don't, bookmark this page, stop here,
open up irb or erl and play with the examples until they make sense. It may take a while.
That's ok. This may very will be very different than what you're used to. I spent a few months
with [The Little Schemer][lil-schemer] to get here.

### Refactoring to Y

So we're refactoring now. The main impetus in each step will be, "I don't like how this looks. Let's
make it look nicer". Nothing new in functionality. Just changing the looks.

Starting points:

{% highlight ruby %}
#ruby
fac = lambda do |f|
        lambda do |n|
          if n.zero?
            1
          else
            n * f.(f).(n-1)
          end
        end
      end

# fac.(fac).(10)
# => 3628800
{% endhighlight %}

{% highlight erlang %}
%% Erlang.
Fac = fun(F) ->
        fun(0) -> 1;
           (N) -> N * (F(F))(N-1) end
      end.

%% Fac(Fac).
%% => 3628800
{% endhighlight %}

What don't I like? Making the user write `(Fac(Fac)(10)`or `fac.(fac).(10)` to
get the factorial of 10 (in this example), isn't very user friendly. Something like
`G(10)` would really be a nicer looking api. So let's make that happen:

{% highlight ruby %}
#ruby
G = lambda { |h|
       h.(h)
    }.(
      lambda do |f|
        lambda do |n|
          if n.zero?
            1
          else
            n * f.(f).(n-1)
          end
        end
      end
    )

# G.(10)
# => 3628800
{% endhighlight %}

{% highlight erlang %}
%% Erlang.
G = (fun(H) -> H(H) end)(
      fun(F) ->
              fun(0) -> 1;
                 (N) -> N * (F(F))(N-1) end
            end
   ).

%% G(10).
%% => 3628800
{% endhighlight %}

If the above looks intimidating: all we are doing is passing our old function into
a new function of the form `lambda { |h| h(h) }` or `fun(H) -> H(H) end`. All that's
doing is making it so that the user doesn't have to do Fac(Fac) anymore.

What else don't I like? Well, the user no longer has to do Fac(Fac), which is nice,
but it still lives in the function body which is kind of ugly. We need to get rid of it,
which we'll do in 3 steps so you can see exactly how to do this type of refactoring.

### Step 1. Wrap the offending code with an inline  lambda:

{% highlight ruby %}
#ruby
G = lambda { |h|
       h.(h)
    }.(
      lambda do |f|
        lambda do |n|
          if n.zero?
            1
          else
            n * lambda { |x| f.(f).(x) }.(n-1)
          end
        end
      end
    )

# G.(10)
# => 3628800
{% endhighlight %}

{% highlight erlang %}
%% Erlang.
G = (fun(H) -> H(H) end)(
      fun(F) ->
              fun(0) -> 1;
                 (N) -> N * (fun(X) -> (F(F))(X) end)(N-1) end
            end
   ).

%% G(10).
%% => 3628800
{% endhighlight %}

So we wrapped the ugliness with a lambda.
`f.(f).(x)` became `lambda { |x| f.(f).(x) }.(n-1)`. Convince yourself
that these are identical statements.

It's no different than this:
changing `1 + 2` to `lambda { |x| 1 + x }.(2)`.

So we've made ugly code even uglier. Great job! Heh. Remember, this is just
step 1. We're going slow so you can see. "Wrap offending code with an inline lambda".
We've done that. On to step 2.

### Step 2. replace inline lambda with a function (m, in this case)

{% highlight ruby %}
#ruby
G = lambda { |h|
       h.(h)
    }.(
      lambda do |f|
        lambda do |m|
          lambda do |n|
            if n.zero?
              1
            else
              n * m.(n-1)
            end
          end
        end.(lambda { |x| f.(f).(x) })
      end
    )

# G.(10)
# => 3628800
{% endhighlight %}

{% highlight erlang %}
%% Erlang.
G = (fun(H) -> H(H) end)(
      fun(F) ->
        (fun(M) ->
          fun(0) -> 1;
             (N) -> N * M(N-1) end
        end)(fun(X) -> (F(F))(X) end)
      end
   ).

%% G(10).
%% => 3628800
{% endhighlight %}

Looks pretty crazy, huh? Remember how we got here. We've just been refactoring.

If you're still stuck, stare at the following examples and convince yourself that
they are all the same. Open up erl or irb and enter each example and see that
the value doesn't change.

{% highlight ruby %}
# Ruby:
# phase 1
1 + 2
# => 3

# phase 2
lambda { |x| 1  + x }.(2)
# => 3

# phase 3
lambda { |m|
  m.(2)
}.(lambda { |x| 1 + x })
# => 3
{% endhighlight %}

{% highlight erlang %}
% Erlang:
% phase 1
1 + 2.
% => 3

% phase 2
(fun(X) -> 1 + X end)(2).
% => 3

% phase 3
(fun(M) ->
  M(2)
end)(fun(X) -> 1 + X end).
% => 3
{% endhighlight %}

So if you understand that, then you should be able to see what we did in the crazy
examples above. Just replace 1 + 2 with `fac.(fac).(n)` in ruby or `(Fac(Fac))(N)`
in Erang.

### Step 3: replace inline lambda with a function (c, in this case)

{% highlight ruby %}
G = lambda { |c|
      lambda { |h|
        h.(h)
      }.(
        lambda { |f|
          c.(lambda { |x| f.(f).(x) })
        }
      )
    }.(
      lambda do |m|
        lambda do |n|
          if n.zero?
            1
          else
            n * m.(n-1)
          end
        end
      end
    )

# G.(10)
# => 3628800
{% endhighlight %}


{% highlight erlang %}
%% Erlang.

G = (fun(C) ->
      (fun(H) -> H(H) end)(
        fun(F) ->
          C(fun(X) -> (F(F))(X) end)
        end
      )
    end)(
      fun(M) ->
        fun(0) -> 1;
           (N) -> N * M(N-1) end
      end
    ).

%% G(10).
%% => 3628800
{% endhighlight %}

Congrats. You've derived Y. Here they are, written generically, in all their glory.

{% highlight ruby %}
# Ruby
Y = lambda { |c|
      lambda { |h|
        h.(h)
      }.(
        lambda { |f|
          c.(lambda { |x| f.(f).(x) })
        }
      )
    }
{% endhighlight %}


{% highlight erlang %}
%% Erlang.

Y = fun(C) ->
      (fun(H) -> H(H) end)(
        fun(F) ->
          C(fun(X) -> (F(F))(X) end)
        end
      )
    end.

{% endhighlight %}

As you've already seen, Y takes an argument which is a higher-order function
of the form of:

{% highlight ruby %}
lambda do |f|
  lambda do |n|
    # stuff
  end
end
{% endhighlight %}

Where f is a function that needs to recur. Don't forget F(F) ->

"I'm a function and my name is F. When I get to the point that I need help, I call the function F".

The Y-combinator has the distinction of being easier to derive than to read. Just remember
what you've learned: when you see h(h) or f(f) say: he's building a recursive function here.
If you see f.(f).(x) say: he's building a recursive function and then passing it an argument.
That will get you through the trickier parts of reading it.

A Y-combinator helps anonymous, higher order functions recur that couldn't otherwise
recur on their own. [Paul Graham's incubator][graham] is appropriately named, I think.

### Final thoughts:

Erlang developers have more use for this than Rubyists, but there is a [nicer way now][joe] for
Erlang devs (see
  section on named funs).

Challenge: Can you derive a Y-Combinator that helps functions of 2, 3 or 4 arguments recur?
It's [easier than you think][my-gist] if you were able to follow the refactorings above.

If I've made some mistakes (and I no doubt have), please send me an email at jimtron9000@gmail.com
and I'll make the necessary corrections.

Thanks - and be sure to check out the [Little Schemer][lil-schemer]. It's the best computer
science book I've ever read.

-- Jim

[my-gist]: https://gist.github.com/jwhiteman/34d447a0d249d2e29035
[joe]: http://joearms.github.io/2014/02/01/big-changes-to-erlang.html
[mrf]: http://www.youtube.com/watch?v=HVIdy9_wqQw
[graham]: http://www.ycombinator.com/
[lil-schemer]: http://www.ccs.neu.edu/home/matthias/BTLS
[ruby-fp]: https://github.com/jwhiteman/jimtron-fp
