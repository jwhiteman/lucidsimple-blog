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
process the job, and then return to listening to handle the next incoming job.

The key to setting this all up is to remember that we can use <a href="http://blog.lucidsimple.com/2016/02/07/simple-OTP-idioms-using-handle-info-part-1.html">handle_info</a>
as a swiss army knife to handle "out of band" messages _and_ that we can kick these off from the initializer:

{% highlight elixir %}
defmodule NaivePoller do
  use GenServer

  def start_link(queue) do
    GenServer.start_link __MODULE__, queue, name: __MODULE__
  end

  def init(queue) do
    {:ok, conn} = Redix.start_link

    schedule_poller()

    {ok, %{conn: conn, queue: queue}}
  end

  def handle_info(:poll, state = %{conn: conn, queue: queue}) do
    job = dequeue_job(conn, queue)

    SomeWorker.handle(job)

    schedule_poller()

    {:noreply, state}
  end

  defp schedule_poller() do
    send :poll, self()
  end

  defp dequeue_job(conn, queue) do
    Redix.command!(conn, ["BRPOP", queue, 0], timeout: :infinity)
  end
end
{% endhighlight %}

Let's walk through this code.

In **init** we use <a href="https://github.com/whatyouhide/redix">Redix</a> to initialize our Redis client. And then on the following line we call **schedule_poller** which
shoves the message **:poll** into our mailbox.

The initializer completes normally, setting up a simple map for the server state which wraps the Redis connection along with the name of the queue we're interested in:

{% highlight elixir %}
def init(queue) do
  # setup up our Redis client
  {:ok, conn} = Redix.start_link

  # shove :poll into our mailbox
  schedule_poller() # => send :poll, self()

  # return our server state
  {ok, %{conn: conn, queue: queue}}
end
{% endhighlight %}

**handle_info(:poll, state)** is then immediately invoked to deal with the **:poll** message we sent ourselves in **init**.

We use this as an opportunity to call **dequeue_job** which makes a blocking call to Redis via <a href="http://redis.io/commands/brpop">BRPOP</a> (ignore that there are <a href="http://redis.io/commands/BRPOPLPUSH#pattern-reliable-queue">better patterns for this</a>, for now).

{% highlight elixir %}
def handle_info(:poll, state = %{conn: conn, queue: queue}) do
  # dequeue the job
  job = dequeue_job(conn, queue)

  # handle the job
  SomeWorker.handle(job)

  # set ourselves up to recur
  schedule_poller() # => send :poll, self()

  # return from handle_info
  {:noreply, state}
end
{% endhighlight %}

Basically we're just waiting around for a job to come in.

Once a job appears **dequeue_job** returns with a job, that gets handled by a hypothetical worker named SomeWorker.

After the job has been processed we make another call to **schedule_poller**, which shoves **:poll** back into the mailbox which causes the whole process to start over again.

So, in essence, our GenServer is recurring to get work done. Pretty cool.
