---
layout: post
title:  "ExUnit Cheat Sheet"
date:   2016-01-31 12:01:00
categories: Elixir
comments: true
---

> There is a lot of good documentation on ExUnit, not the least of which
> is the ExUnit section at elixir-lang.org.
>
> This little cheat sheet isn't meant to be comprehensive, but instead is a
> list of the ExUnit techniques I use on almost every project.

### 1. Mark Tests as Pending

If you want to mark certain tests as _pending_ and automatically exclude them from your test runs, use `ExUnit.configure`
in your test helper:

{% highlight elixir %}
# test_helper.exs
ExUnit.configure(exclude: [pending: true])

ExUnit.start()
{% endhighlight %}

Then you can use a tag to mark it as pending which will cause it to be skipped
when you run `mix test`:


{% highlight elixir %}
defmodule SomeModuleTest do
  use ExUnit.Case

  @tag :pending
  test "ignorance is strength" do
    assert 2 + 2 == 5
  end
end
{% endhighlight %}

![Pending Test(/assets/pending.png)](/assets/pending.png)

From the output above you can see that the pending test was skipped.

If you actually want to run your pending tests later on, you can do this:

```bash
mix test --include pending
```

### 2. Run Only Certain Tests

If you had some WIP tests, for example, you could select _only_ those to run with the `only` flag:

```bash
mix test --only wip
```

{% highlight elixir %}
defmodule SomeModuleTest do
  use ExUnit.Case

  test "some test" do
    assert 1 + 1 == 2
  end

  @tag :pending
  test "ignorance is strength" do
    assert 2 + 2 == 5
  end

  @tag :wip
  test "my great new feature" do
    assert 2 + 2 == 4
  end
end
{% endhighlight %}

![Running WIP Tests(/assets/wip.png)](/assets/wip.png)

If you read the output above, you'll see that only 1 of the three
tests ran.

In this case, only tags marked `wip` were run.

Similarly, you could _exclude_ your WIP tests this way:

```bash
mix test --exclude wip
```

### 3. Silence STDOUT
I don't like it when my test output is littered with print or log messages.

If you have Elixir version 1.1+ then you can use `ExUnit.CaptureIO` to keep
things clean (<a href="http://elixir-lang.org/docs/v1.0/ex_unit/ExUnit.CaptureIO.html#capture_io/3" target="_blank">among other uses</a>):

{% highlight elixir %}
defmodule SomeModuleTest do
  use ExUnit.Case

  import ExUnit.CaptureIO

  test "some noisy test" do
    capture_io fn ->
      IO.puts "RAWR"
    end

    assert 1 + 1 == 2
  end
end
{% endhighlight %}

If you wrap the noise producing parts in a function and pass it
to `capture_io`, then the output will be suppressed.

In the above example "RAWR" will _not_ end up with the test results.

Don't fret If your version of Elixir doesn't have CaptureIO; <a href="https://github.com/whatyouhide/redix/blob/master/test/test_helper.exs#L10-L18" target="blank">Other techniques may still work</a>.

### 4. Ensure a Service is Running

This isn't ExUnit specific, per se, but if your test suite depends on a particular service running (e.g Redis), then you can
test for it when your test helper gets loaded.

That way, if the service _isn't_ available then you have the opportunity to abort the test
run and print a friendly error message:

{% highlight elixir %}
# test_helper.exs
ExUnit.start()

case :gen_tcp.connect('localhost', 6379, []) do
  {:ok, socket} ->
    :gen_tcp.close(socket)
  {:error, reason} ->
    Mix.raise "Cannot connect to Redis" <>
              " (http://localhost:6379):" <>
              " #{:inet.format_error(reason)}"
end
{% endhighlight %}

This <a href="https://github.com/whatyouhide/redix/blob/master/test/test_helper.exs#L3-L8" target="_blank">example</a> comes from <a href="https://github.com/whatyouhide" target="_blank">Andrea Leopardi's</a> great <a href="https://github.com/whatyouhide/redix" target="_blank">Redix</a> project.

He uses `:get_tcp` to check that he can connect to port 6379. If the connection is successful, he closes the socket and the test run continues. If the connection is not successful, he uses `Mix.raise` with a helpful error message to shut things down.

Pretty cool.

### 5. Load Support Modules

It's not a problem to create support modules directly in your test files, but if they get too big then
you may want to move them to their own directory.

This technique comes from <a href="https://github.com/mikepack" target="_blank">Mike Pack's</a> <a href="https://github.com/mikepack/poolboy_queue" target="_blank">Poolboy Queue</a> library:

{% highlight elixir %}
# test_helper.exs
ExUnit.start()

{:ok, files} = File.ls("./test/support")

Enum.each files, fn(file) ->
  Code.require_file "support/#{file}", __DIR__
end
{% endhighlight %}

He uses `File.ls` to fetch the names of all of the files living under `test/support,` and then applies `Code.require_file` to each one.

Once again, this isn't ExUnit specific, but this simple technique pops up fairly frequently in my own projects.

### 6. Workhorse Assertions

The best source on ExUnit assertions is still <a href="http://elixir-lang.org/docs/stable/ex_unit/ExUnit.Assertions.html" target="_blank">the docs at elixir-lang.org</a>.

However, I use the following 3 assertions roughly <a href="https://en.wikipedia.org/wiki/Pareto_principle" target="_blank">80% of the time</a>:

- <a href="http://elixir-lang.org/docs/v1.0/ex_unit/ExUnit.Assertions.html#assert/1" target="_blank">assert</a>
- <a href="http://elixir-lang.org/docs/v1.0/ex_unit/ExUnit.Assertions.html#refute/1" target="_blank">refute</a>
- <a href="http://elixir-lang.org/docs/v1.0/ex_unit/ExUnit.Assertions.html#assert_receive/3" target="_blank">assert_receive</a>

They are mercifully self-explanatory:

{% highlight elixir %}
# assert
test "a simple assertion" do
  assert 2 + 2 == 4
end

# refute
test "a simple refutation" do
  refute 2 + 2 == 5
end

# assert_receive
test "some service calls me back" do
  SomeService.work(self())

  assert_receive {:result, "some-result"}, 100
end
{% endhighlight %}

`assert_receive` and its cousin <a href="http://elixir-lang.org/docs/v1.0/ex_unit/ExUnit.Assertions.html#assert_received/2" target="_blank">assert_received</a> are pretty crucial
given that message passing is at the heart of Elixir and Erlang.

Note that `assert_receive` takes a timeout as the second parameter, which is helpful if you need to wait a bit for a message to arrive at the mailbox.


### 7. Setup and Other Callbacks

I really can't do any better than the great explanation at the <a href="http://elixir-lang.org/docs/stable/ex_unit/ExUnit.Callbacks.html" target="_blank">elixir-lang.org</a> site,
but in case it's not obvious:

- `setup` runs before each test
- `setup_all` is only run once
- `on_exit` lives in a setup block and takes a function as an argument


### 8. Starting or Stopping the Application

By default, your application will be started while your tests run.

If you don't want that, then you could stop it in the test helper:

{% highlight elixir %}
ExUnit.start()

Application.stop(:your_app_name)
{% endhighlight %}

In case you don't remember, the name of your app is listed in your mixfile:

{% highlight elixir %}
# mix.exs
def project do
    [app: :your_app_name,
     # ...
    ]
end
{% endhighlight %}

To get things started again, you only need to call 

{% highlight elixir %}
Application.start(:your_app_name)
{% endhighlight %}

### 9. VCR and Mocking

If you're coming from the Ruby world and would like something similar to <a href="https://github.com/vcr/vcr" target="_blank">VCR</a> to fake HTTP responses, then check out <a href="https://github.com/parroty/exvcr" target="_blank">ExVCR</a>.

Similarly, you can use <a href="https://github.com/eproxus/meck" target="_blank">Meck</a> if you want a nice mocking library.

I have used both and they work very well, but you'll need to take care that you're not straying too far from <a href="http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/" target="_blank">the Elixir way</a> if you use them.

### 10. Using the Seed and Intermittent Errors

Intermittent errors are probably unavoidable, especially if you're writing concurrent programs.

To help fight this, ExUnit tests are run in a random order for via a seed integer.

If a particular run results in errors that you don't normally see, you can run subsequent tests
in the _exact same order_ using that seed as a flag:

```bash
mix test --seed <the-seed-that-caused-errors>
```

![Using the Seed(/assets/seed.png)](/assets/seed.png)

If you look at the output above, you'll see that we're passing the integer 668506 to mix.

No matter how many times we run `mix test --seed 668506` we'll always get the tests run in the exact same order.

Getting consistent results goes a long way in squashing pesky errors that only occur some of the time.

# Wrapup

I wrote this post for myself so that I can easily lookup the techniques I use frequently but still manage to forget.

I hope you will find at least some use from it.

Here are some other articles you might be interested in:

- <a href="http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/" target="_blank">Mocks and Explicit Contracts</a>
- <a href="https://gist.github.com/timruffles/036b9782472e5dd0844d" target="_blank">Approaches to Dependency Injection in Elixir</a>
- <a href="http://elixir-lang.org/docs/master/ex_unit/ExUnit.html" target="_blank">The official ExUnit docs</a>
- <a href="/2016/01/04/easy-mocking-in-elixir-with-meck.html">Easy Mocking in Elixir with Meck<a/>
- <a href="/2014/08/14/binary-fixtures-with-wireshark.html">Binary Fixtures with Wireshark</a>
