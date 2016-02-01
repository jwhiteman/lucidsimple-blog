---
layout: post
title:  "A Y-Combinator for Josh Susser"
date:   2008-06-26 08:00:00
categories: Ruby
---

> Author note: This post is old.

A couple of days ago Josh Susser posted a very [attractive recursive lambda][susser] in his blog.

A couple of people in the comments said that a recursive lambda is a prime opportunity to use the Y-combinator.

In Scheme? Sure. In a Rails application? I don't think so.

While I think that functional programing represents a more powerful programming paradigm than what's typically used in Ruby, I think it's a mistake to parade Ruby around like it's Lisp (or Haskell, or ML, etc). It can 'do' Lisp (sans macros) but it's so much better at being itself.

Anyhoo, I whipped this thing up just to see what it would look like. I started writing in Ruby 1.9, but for some reason, when you coerce a lambda into a block (i.e function(& my_lambda)) it turns hash iterations (i.e |key,value|) into arrays ( |array| ). So I did it in 1.8 -- for which lambda syntax sucks. So I'm going to use `&lambda` instead of lambda 'cuz it just looks good.

{% highlight ruby %}
class ApplicationController < ActionController::Base
  before_filter { |controller| controller.params.each(&λ { |le|
      λ {|f|
        f[f]
      }.call( λ {|f| le.call( λ { f[f] }) })
    }.call(λ {|m|
            λ {|k,v|
              case v
                when String then v.remove_accents!
                when Hash   then v.each(&m.call)
              end
             }
        }))
   }
end
{% endhighlight %}

Well...it is completely anonymous now. Are we having fun yet? Obviously, there are more readable ways to use Y but none of them result in anything nicer than what Josh wrote.

This is basically a long way of saying avoid overly complicated code. You can never justify masturbatory programming because you're using strict FP -- especially in Ruby and even more so in Rails.

<div class="cta">Did you find this post helpful? If so, <a href="/subscribe">you may want to consider subscribing</a>.</div>

[susser]: http://blog.hasmanythrough.com/2008/6/20/recursive-lambda
