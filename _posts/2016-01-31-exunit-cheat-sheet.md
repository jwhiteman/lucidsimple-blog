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

If you want to mark certain tests as _pending_, use `ExUnit.configure`
in your `test_helper.exs` file:

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

FYI, you would get similar behavior from running:

```bash
mix test --exclude pending
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

### 3. Silence STDOUT
I don't like it when my test output is littered with print or log messages.

If you have Elixir version 1.1+ then you can use `ExUnit.CaptureIO` to keep
things clean, or even test the printed output directly::

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

Don't fret If your version of Elixir doesn't have CaptureIO. <a href="https://github.com/whatyouhide/redix/blob/master/test/test_helper.exs#L10-L18" target="blank">Other techniques may work</a>.

### 4. Ensure a Service is Running

If your test suite depends on a particular service running (e.g Redis), then you can
test for it in the test_helper.exs.

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

He uses `:get_tcp` to check that he can connect to port 6379. If the connection is successful, he closes the socket and the test run continues. If the connection is not successful, he uses `Mix.raise` with a clear error message to shut things down.

I think that's pretty cool and it's a technique I now use myself.

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

This is a simple technique that I use all of the time now.

### Assertions

### Setup vs Setup All

### Starting or Stopping the Application

### ExVCR and Meck
http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/

### Using the Seed

Here are some other articles you might be interested in:

- <a href="http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/" target="_blank">Mocks and Explicit Contracts</a>
- <a href="https://gist.github.com/timruffles/036b9782472e5dd0844d" target="_blank">Approaches to Dependency Injection in Elixir</a>
- <a href="http://elixir-lang.org/docs/master/ex_unit/ExUnit.html" target="_blank">The official ExUnit docs</a>
- <a href="/2016/01/04/easy-mocking-in-elixir-with-meck.html">Easy Mocking in Elixir with Meck<a/>
- <a href="/2014/08/14/binary-fixtures-with-wireshark.html">Binary Fixtures with Wireshark</a>
