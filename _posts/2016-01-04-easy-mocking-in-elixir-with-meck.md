---
layout: post
title:  "Easy Mocking in Elixir with Meck"
date:   2016-01-04 10:47:00
categories: Elixir
comments: true
---

> Here is a short article and video using <a href="https://github.com/eproxus/meck" target="_blank;">meck</a> in Elixir.
>
> If you're pretty comfortable with Elixir but are unfamiliar with meck, then the
> article should be enough.
>
> Otherwise, the video might be useful. It's one take, using my laptop microphone. My apologies for its roughness.

### Video

<iframe src="https://player.vimeo.com/video/150702113" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="https://vimeo.com/150702113">Easy Mocking in Elixir with Meck</a> from <a href="https://vimeo.com/user29282688">Jim Whiteman</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

You can grab the code from the video <a href="https://github.com/jwhiteman/lucid-simple-projects">here</a>.

### Using Meck

Here is our hypothetical setup: a module named Math that supports addition,
subtraction and multiplication.

Let's pretend that multiplication is special and we have to introduce a dependency
to get the job done.

{% highlight elixir %}
# math.ex
defmodule Math do
  def add(x, y) do
    x + y
  end

  def sub(x, y) do
    x - y
  end

  def mul(x, y) do
    SomeDependency.mul(x, y)
  end
end

# math_test.exs
defmodule MathTest do
  use ExUnit.Case

  test "addition" do
    assert Math.add(1, 1) == 2
  end

  test "subtraction" do
    assert Math.sub(1, 1) == 0
  end

  test "multiplication" do
    assert Math.mul(2, 3) == 6
  end
end
{% endhighlight %}

Let's also pretend that `SomeDependency.mul` is pretty expensive to run.

We'll imagine it makes a bunch of HTTP requests and then defrags a couple of hard disks for good measure.

It's useful enough that we need it, but it's a giant boat anchor weighing down
math_test.exs. Every time we hit multiplication in our test, `SomeDependency.mul`
comes along for the ride.

As a result, our math_test runs very slowly.

So, let's bring `:meck` into the mix to mock it out and bring sanity back to our tests.

{% highlight elixir %}
defmodule Math.Mixfile do
  # ...

  defp deps do
    [{:meck, "~> 0.8.2", only: :test}]
  end
end
{% endhighlight %}

To make things work we'll make 3 invocations, `:meck.new`, `:meck.expect` and `:meck.unload`.

{% highlight elixir %}
# math_test.exs
defmodule MathTest do
  test "offers multiplication" do
    home = self

    :meck.new(SomeDependency) # setup

    :meck.expect(
      SomeDependency,
      :mul,
      fn(2, 3) -> send home, :called! end
    )

    Math.mul(2, 3)

    assert_receive :called!, 10

    :meck.unload(SomeDependency) # teardown
  end
end
{% endhighlight %}

`:meck.new` creates a new mocked module based on what you pass in. It has a
<a href="https://github.com/eproxus/meck/blob/master/src/meck.erl#L148">decent
number of options on its own</a>.

`:meck.expect` is where the magic happens. You pass in the module and function
you wish to mock, and pass in the function you want to replace it with.

In the above test, I'm not really testing that multiplication works anymore, only
that it hits some particular service named SomeDependency.

So instead of returning a calculation, I'm just sending a message back home with `:called!`, and then asserting it ends up in our mailbox within a 100th of a second or so.

You could set this up a million different ways. There is nothing special about the
symbol :called!, nor do you need to send a message back to yourself here. The replacement function can be whatever you want it to be. This is just what I chose for this example.

`:meck.unload` unloads the module from memory, cleaning things up for the rest of
your tests.

And that's that.

In the above test case, the real implementation of `SomeDependency.mul` won't be touched, and `fn(2, 3) -> send home, :called! end` will be called instead.

Impressive!

### Drying things up with a simple macro

`:meck.new` and `:meck.unload` feel like boilerplate to me and should probably
live in `setup` and `on_exit`, respectively.

Instead, though, let's create an easy macro to take care of the setup and teardown
for us on a per-test basis.

{% highlight elixir %}
# test_helper.exs
ExUnit.start()

defmodule TestHelpers do
  defmacro mocking(module, fun, replacement, do: body) do
    quote do
      :meck.new(unquote(module))

      :meck.expect(
        unquote(module),
        unquote(fun),
        unquote(replacement)
      )

      unquote(body)

      :meck.unload(unquote(module))
    end
  end
end
{% endhighlight %}

This turns our test case into this:

{% highlight elixir %}
test "offers multiplication" do
  home = self

  mocking(SomeDependency,:mul,fn(2, 3) -> send home, :called! end) do
    Math.mul(2, 3)

    assert_receive :called!, 10
  end
end
{% endhighlight %}

Not too shabby.

Actually, <a href="https://github.com/jjh42">jjh42's</a> <a href="https://github.com/jjh42/mock">nice Elixir wrapper for mech</a> <a href="https://github.com/jjh42/mock/blob/master/lib/mock.ex#L40">does something similar.</a> But it wasn't too difficult to roll something simple for ourselves.

At any rate, there is a ton more you can do with Meck. If you'd like to use it,
it would be a good use of your time to take a look at <a href="https://github.com/eproxus/meck/blob/master/README.md">the project's README</a>.

### Wrapup

The only thing I'd like to add is that I'm not entirely convinced that this is
the _Elixir_ way to do things. In particular, I don't see it used too heavily
out in the wild.

The advice I tend to hear instead is, "put as much of your logic as you can in stateless modules". Maybe if 90% of your code is setup that way, then mocking in this manner becomes
less of an issue. Or not. I'm still figuring this out myself.

Regardless, it's a pretty slick library that feels comfortable, especially coming from Ruby.

<div class="cta">Did you find this post helpful? If so, <a href="/subscribe">you may want to consider subscribing</a>.</div>
