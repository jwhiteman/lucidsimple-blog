---
layout: post
title:  "Creating a Looping GenServer"
date:   2016-02-21 12:02:00
categories: Elixir
description: "Creating a Looping GenServer"
comments: true
---

> This week's post is short.
>
> It should be a 5 minute read.

<a href="http://elixir-lang.org/docs/v1.1/elixir/GenServer.html">GenServers</a> are very easy to setup, but it's not immediately obvious how to implement
a looping server that does work on its own.

For example, let's say we're writing a job queue and we want our GenServer worker to respond to jobs placed into Redis. We want our server to pop jobs from Redis,
process the job, and then start over again for the next incoming job.

The key to setting this all up is to remember that we can use <a href="http://blog.lucidsimple.com/2016/02/07/simple-OTP-idioms-using-handle-info-part-1.html">handle_info</a>
as a swiss army knife for "out of band" messages _and_ that we can kick these off from the initializer:

{% highlight elixir %}
defmodule NaivePoller do
  use GenServer

  def start_link(queue_name) do
    GenServer.start_link __MODULE__, queue_name, name: __MODULE__
  end

  def init(queue_name) do
    {:ok, conn} = Redix.start_link

    schedule_poller()

    {:ok, %{conn: conn, queue: queue_name}}
  end

  def handle_info(:poll, state = %{conn: conn, queue: queue_name}) do
    job = dequeue_job(conn, queue_name)

    SomeWorker.handle(job)

    schedule_poller()

    {:noreply, state}
  end

  defp schedule_poller() do
    send self(), :poll
  end

  defp dequeue_job(conn, queue_name) do
    Redix.command!(conn, ["BRPOP", queue_name, 0], timeout: :infinity)
  end
end
{% endhighlight %}

Let's walk through this code.

In **init** we use <a href="https://github.com/whatyouhide/redix">Redix</a> to initialize our Redis client. And then on the following line we call **schedule_poller** which
shoves the message **:poll** into our mailbox.

The initializer completes normally, setting up a simple map for the server state which wraps the Redis connection along with the name of the queue we're interested in:

{% highlight elixir %}
def init(queue_name) do
  # setup up our Redis client
  {:ok, conn} = Redix.start_link

  # shove :poll into our mailbox
  schedule_poller() # => send self(), :poll

  # return our server state
  {:ok, %{conn: conn, queue: queue_name}}
end
{% endhighlight %}

**handle_info(:poll, state)** is then immediately invoked to deal with the **:poll** message we sent ourselves in **init**.

We use this as an opportunity to call **dequeue_job** which makes a blocking call to Redis via <a href="http://redis.io/commands/brpop">BRPOP</a> (ignore that there are <a href="http://redis.io/commands/BRPOPLPUSH#pattern-reliable-queue">better patterns for this</a>, for now).

{% highlight elixir %}
def handle_info(:poll, state = %{conn: conn, queue: queue_name}) do
  # dequeue the job
  job = dequeue_job(conn, queue_name)

  # handle the job
  SomeWorker.handle(job)

  # set ourselves up to recur
  schedule_poller() # => send self(), :poll

  # return from handle_info
  {:noreply, state}
end
{% endhighlight %}

Basically we're just waiting around for a job to come in.

Once a job appears in Redis, **dequeue_job** will pop it off so that the hypothetical worker SomeWorker can process it.

After the job has been processed we make another call to **schedule_poller**, which shoves **:poll** back into the mailbox causing the whole process to start over again.

So, in essence, our GenServer is recurring to get work done. Pretty cool.

Here are some other posts you may be interested in:

- <a href="http://stackoverflow.com/questions/30568806/which-otp-behavior-should-i-use-for-an-endless-repetition-of-tasks/30570202#30570202">Using OTP for "endless" repition</a>
- <a href="http://stackoverflow.com/questions/32085258/how-to-run-some-code-every-few-hours-in-phoenix-framework/32097971#32097971">Scheduling code in Phoenix</a>
- <a href="http://blog.lucidsimple.com/2016/02/07/simple-OTP-idioms-using-handle-info-part-1.html">Simple OTP Idioms: using handle_info (Part 1)</a>
