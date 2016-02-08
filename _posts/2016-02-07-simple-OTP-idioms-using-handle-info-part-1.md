---
layout: post
title:  "Simple OTP Idioms: using handle_info (Part 1)"
date:   2016-02-07 11:58:00
categories: Elixir
comments: true
---

> This week's post should be less than a 5 minute read.

This is the first of a series of posts on simple OTP idioms, patterns and tricks.

We'll start with a few posts on <a href="http://erldocs.com/18.0/stdlib/gen_server.html#handle_info/2">handle_info</a>, each week covering some
some easy pattern that can be covered in a few minutes.

This week we'll talk about how to use handle_info to slightly defer the initialization of an expensive to set up server.

## A Simple Example

You're writing an application in Elixir and you've got a <a href="http://elixir-lang.org/docs/v1.1/elixir/GenServer.html">GenServer</a> that is costly to initialize:

{% highlight elixir %}
defmodule MyServer do
  use GenServer

  def init([]) do
    state = something_slow()

    {:ok, state}
  end
end
{% endhighlight %}

The problem with this is that 'MyServer.start_link' has to block until 'init' finishes.

If 'init' is slow, then any caller trying to create an instance of 'MyServer' will be temporarily blocked.

The issue is made worse if there are multiple servers set up this way because OTP Supervisors start each of their children serially. So if you have 10 child servers that each take 6 seconds to initialize, then the supervisor will be blocked for a minute while it tries to get each one started.

We'd like to avoid this situation if we can.

## One Possible Solution

A common workaround is to trigger a timeout from init so that <a href="http://elixir-lang.org/docs/v1.1/elixir/GenServer.html#start_link/3">start_link</a> will return immeditely. The expensive init code is then moved to a <a href="http://elixir-lang.org/docs/v1.1/elixir/GenServer.html#c:handle_info/2">handle_info</a> callback:

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

The timeout will trigger the <a href="http://erlang.org/doc/man/gen_server.html#Module:handle_cast-2">handle_info</a> callback allowing you to do your drawn out setup out of band without making everyone else wait.

I've received conflicting information on whether or not the timeout handler will be _guaranteed_ to be the first message handled by your server.

If it's not, then your server will most likely crash because its state will not have been fully initialized at that point.

This seems <a href="http://stackoverflow.com/questions/14648304/is-handle-info-guaranteed-to-executed-first-in-a-process-after-init-with-timeout">very unlikely</a>, although it's worth keeping in mind if you run into any strange bugs.


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

handle_info deals with all out of band messages sent to your GenServer process, which includes timeouts.

### Which Approach is Best?

If the state of your server needs to be periodically updated on a schedule (e.g a Cache), then setting things up with timeouts might be the way to go.

For everything else, the second approach is probably best.

You'll see several variations of these techniques in open source projects if you hunt for them (see the links below).

You don't have to use them yourself, but it can save you time if you kow how to spot them.

## Closing

Here are some relevant examples of handle_info that you might be interested in:

- <a href="https://github.com/undeadlabs/discovery/blob/master/lib/discovery/heartbeat.ex#L60-L68">heartbeat.ex in Discovery</a>
- <a href="https://github.com/aetrion/erl-dns/blob/master/src/erldns_storage.erl#L67-L73">erldns_stroage.erl in erl-dns<a>
- <a href="https://github.com/akira/exq/blob/master/lib/exq/scheduler/server.ex#L55-L58">server.ex in ExQ</a>
- <a href="https://github.com/klacke/yaws/blob/master/src/yaws_session_server.erl#L253-L255">yaws_session_server.erl in Yaws webserver</a>
- <a href="https://github.com/erlang/otp/blob/maint/lib/os_mon/src/disksup.erl#L153-L157">disksup.erl in OTP</a>

If you read through the examples, it will be worth your time to checkout how each 'init' function is setup. A few are explicitly setting timeouts, while others use different approaches.

Next week's post will cover using handle_info for looping and polling.

Thanks for following along.

-- Jim

