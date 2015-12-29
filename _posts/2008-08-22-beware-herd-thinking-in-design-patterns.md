---
layout: post
title:  "Beware Herd Thinking in Design Patterns"
date:   2008-08-22
categories: Design Patterns
---

I was reading [Object Design][object-design] a few months back and came upon an example illustrating double dispatch.

The example was showing a simple way to model [Rock, Paper, Scissors][rps]. Here is the code translated to Ruby from Java:

Programmers with some experience with metaprogramming may instinctively feel uneasy with this code. I mean, it looks like a pattern. And we know that if we wished to extend the game, then we're obligated to add another boxy class with all of those silly `loses_to...` methods.
{% highlight ruby %}
class Rock
  def beats?(obj)
    obj.loses_to_rock?
  end

  def loses_to_rock?; false; end
  def loses_to_paper?; true; end
  def loses_to_scissors?; false; end
end

class Paper
  def beats?(obj)
    obj.loses_to_paper?
  end

  def loses_to_rock?; false; end
  def loses_to_paper?; false; end
  def loses_to_scissors?; true; end
end

class Scissors
  def beats?(obj)
    obj.loses_to_scissors?
  end

  def loses_to_rock?; true; end
  def loses_to_paper?; false; end
  def loses_to_scissors?; false; end
end
{% endhighlight %}

Here's a *much* better way:

{% highlight ruby %}
# this makes me happy :)
class Piece
  class << self
   def beats(*args)
     class_eval do
       define_method(:beats) do
         args.map { |s| Object.const_get(s.to_s.capitalize) }
       end
     end
   end
 end

 def beats?(opp)
   beats.include? opp.class
 end
end

class Rock < Piece
  beats :scissors
end

class Scissors < Piece
  beats :paper
end

class Paper < Piece
  beats :rock
end
{% endhighlight %}

In this way, it is trivial to extend the game:

{% highlight ruby %}
class JamesWhiteman < Piece
 beats :paper, :scissors, :rock
end

# ok, let me instantiate myself...
james_whiteman = JamesWhiteman.new

# and time to start kicking some ass :)
james_whiteman.beats? Rock.new
# => TRUE

james_whiteman.beats? Paper.new
# => TRUE

james_whiteman.beats? Scissors.new
# => TRUE
{% endhighlight %}

[object-design]: http://www.textbooks.com/BooksDescription.php?BKN=580415&mcid=XCS-Shoppingdotcom-9780201379433-U&utm_medium=shoppingengine&utm_term=9780201379433U&utm_source=shoppingdotcom&
[rps]: https://en.wikipedia.org/wiki/Rock-paper-scissors
