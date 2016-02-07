---
layout: post
title:  "Simple OTP Idioms: using handle_info to defer initialization"
date:   2016-02-07 11:58:00
categories: Elixir
comments: true
---

> This is the first of an (eventual) series on simple OTP idioms, patterns and tricks.
>
> This weeks post should be 3 or 4 minute read.

## A Simple Example

You're writing an application in Elixir and you've got a <a href="http://elixir-lang.org/docs/v1.1/elixir/GenServer.html">GenServer</a> that is expensive to initialize:

{% highlight elixir %}
defmodule MyServer do
  use GenServer

  def init([]) do
    state = something_slow()

    {:ok, state}
  end
end
{% endhighlight %}

One possible issue with this is that calls to `MyServer.start_link` will block until the init function finishes. If you have a slow init function, then the caller will just have to wait.

If you have a multiple servers setup this way, then their <a href="http://elixir-lang.org/docs/v1.0/elixir/Supervisor.html">Supervisor</a>  will be slowed down considerably as it starts each child serially.

A common workaround is to trigger a timeout from init so that <a href="http://elixir-lang.org/docs/v1.1/elixir/GenServer.html#start_link/3">start_link</a> will return immeditely. The expensive setup code is then moved to a <a href="http://elixir-lang.org/docs/v1.1/elixir/GenServer.html#c:handle_info/2">handle_info</a> callback:

{% highlight elixir %}
defmodule MyServer do
  use GenServer

  def init([]) do
    {:ok, [], 0}
  end

  def handle_info(:timeout, _state) do
    state = something_slow()

    {:noreply, state}
  end
end
{% endhighlight %}

Note that the return tuple from init has a 3rd element, which is `0.` In this case, `0` simply means "timeout immediately".

The timeout will then trigger the `handle_info` callback allowing you to do your slow setup without making everyone else wait.

As a bonus, you can be guaranteed that the `handle_info(:timeout)` message will be the first message that your server process receive.

You won't have to worry about other processes querying your server before its state has been fully setup.

##  A Similar Approach

It's worth noting that you don't _have_ to use timeout to achieve similar results: 

{% highlight elixir %}
defmodule MyServer do
  use GenServer

  def init([]) do
    send self(), :my_fancy_setup

    {:ok, []}
  end

  def handle_info(:my_fancy_setup, _state) do
    state = something_slow()

    {:noreply, state}
  end
end
{% endhighlight %}

As you can see from above, the real mechanism for deferred initialization isn't the timeout - it's the use of handle_info.

If the state of your server needs to be periodically updated (e.g a Cache), then setting things up with timeouts makes sense.

If the state of your server is expensive, but will only need to be set once, then the second approach would probably be clearer.

## Closing

Here are some examples of handle_info that you might be interested in:

- <a href="https://github.com/undeadlabs/discovery/blob/master/lib/discovery/heartbeat.ex#L60-L68">heartbeat.ex in Discovery</a>
- <a href="https://github.com/aetrion/erl-dns/blob/master/src/erldns_storage.erl#L67-L73">erldns_stroage.erl in erl-dns<a>
- <a href="https://github.com/akira/exq/blob/master/lib/exq/scheduler/server.ex#L55-L58">server.ex in ExQ</a>
- <a href="https://github.com/klacke/yaws/blob/master/src/yaws_session_server.erl#L253-L255">yaws_session_server.erl in Yaws webserver</a>
- <a href="https://github.com/erlang/otp/blob/maint/lib/os_mon/src/disksup.erl#L153-L157">disksup.erl in OTP</a>

If you read through the examples, it will be worth your time to checkout how each `init` function is working.

Next week I'll write about a fairly common pattern I've seen using handle_info for polling and server loops.

