---
layout: post
title:  "Pattern Matching Helps Reduce Conditional Logic"
date:   2016-01-24 18:17:00
categories: Elixir
comments: true
---

> If you read the post title and thought "no shit, Sherlock", then
> you can probably be safe in skipping this week's post.
>
> If you'd like to hear about another selling point of Elixir that isn't
> often mentioned directly, then by all means read on.
>
> This is probably a 2 minute read.

Object Oriented programmers are often preoccupied with avoiding conditional logic.

It is a topic that a lot of <a href="http://www.sandimetz.com/" target="_blank">smart</a> <a href="http://www.martinfowler.com/" target="_blank">people</a> have spent a <a href="http://stackoverflow.com/questions/1337565/avoiding-if-statements" target="_blank">lot of energy on</a>, and the concept 
even <a href="http://antiifcampaign.com/" target="_blank">has its own campaign</a>.

Take a look at this example:

{% highlight javascript %}
function fun(x, y) {
  if (x == 5) {
     if (y == 3) {
       return 7;
     } else {
       return 11;
     }
  } else {
     return y;
  }
}
{% endhighlight %}

I don't think I'm the only one, but when I have to deal
with even small amounts of code like this I find that my
developer happiness temporarily takes a dive.

But why should it? The little chunk of pseudo code has a lot going for it:

1. it's small
2. it's only testing with `==`
3. it's not setting any local variables
4. it's not mutating state
5. it's only returning integers.

And still, most of the developers I know wouldn't call this "good" code. To be fair,
they probably wouldn't call it terrible either. But it would be the sort of thing
that they would hope to minimize.

And that's where a lot of brain cycles start to burn up: the conditions themselves are legitimate 
parts of your domain logic, and they may even be simple when considered one-at-a-time. But when expressed
as a whole it somehow transforms itself into an unmaintainable morass.

If you're clever, then you can mitigate the problem with polymorphism and other OO fare.

But those techniques exact a penalty, too: If you design things correctly, then you'll pay with indirection.
If you design things poorly, then you will still pay with indirection, but you also end up with <a href="https://pbs.twimg.com/media/BiJPfXBCIAAShKW.jpg" target="_blank">the wrong abstractions</a>.

As you might have guessed, this is all just a setup to demonstrate how a language with pattern matching (e.g Elixir)
handles this problem:

{% highlight elixir %}
# The "difficult" pseudo code written in Elixir:
def fun(5, 3), do: 7
def fun(5, _), do: 11
def fun(_, y), do: y
{% endhighlight %}

The simplicity of this solution makes it the spiritual equivalent of this gif:

![Indiana Jones(/assets/jones.gif)](/assets/jones.gif)

Note how pattern matching allows each condition to live on its own at the top level:

{% highlight elixir %}
def fun(5, 3), do: 7
def fun(5, _), do: 11
def fun(_, y), do: y
{% endhighlight %}

You can parse each case individually without the mental gymnastics needed to keep track of which conditional branch you're in.

To be clear, this isn't to say that you should avoid <a href="http://elixir-lang.org/getting-started/case-cond-and-if.html" target="_blank">conditionals in Elixir</a>.
That would be absurd.

My main point is that the nastiness of nested conditionals often evaporates when using a language that supports pattern matching.

And that using Elixir often feels like bringing a gun to a knife fight.

### NOTES

**1**. Elixir and Erlang aren't the only languages with pattern matching. If you like Elixir, then you might find <a href="http://www2.lib.uchicago.edu/keith/ocaml-class/home.html" target="_blank">OCaml</a> interesting. Others <a href="https://www.reddit.com/r/elixir/comments/42k0v2/elixir_pattern_matching_helps_reduce_conditional/czbgehl" target="_blank">have their own favorites</a>.

<div class="cta">Did you find this post helpful? If so, <a href="/subscribe" target="_blank">you may want to consider subscribing</a>.</div>

