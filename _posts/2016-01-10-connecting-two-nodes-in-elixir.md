---
layout: post
title:  "Connecting Remote Nodes in Elixir"
date:   2016-01-10 23:00:00
categories: Elixir
description: "Connecting Remote Nodes in Elixir"
comments: true
---

> Hey there - this is something I was experimenting with during the week and couldn't find that much great
> information on. If you're wanting to setup a cluster of nodes in Elixir and are completely in the dark
> then this info might useful to you.

### Video

<iframe src="https://player.vimeo.com/video/151358592" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="https://vimeo.com/151358592">Setting up a simple cluster in Elixir</a> from <a href="https://vimeo.com/user29282688">Jim Whiteman</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Notes

The <a href="http://www.erlang.org/doc/man/epmd.html" target="_blank">Erlang Port Mapper Daemon</a> uses port 4369 so you'll need that open.

You'll also need to earmark a port in your security group for each node you'll be using.

[![Security Group Example](/assets/security-groups.png)](/assets/security-groups.png)

Each node needs to be setup with a name, cookie, and kernel flag:
{% highlight bash %}
# node 1
iex --name your-fun-node-name-here@hostname-or-ip \
    --cookie some-cookie-name \
    --erl '-kernel inet_dist_listen_min <MINPORT>' \
    --erl '-kernel inet_dist_listen_max <MAXPORT>'

# node 2
iex --name some-other-node-name@some-other-hostname-or-ip
    --cookie some-cookie-name \
    --erl '-kernel inet_dist_listen_min <MINPORT>' \
    --erl '-kernel inet_dist_listen_max <MAXPORT>'
{% endhighlight %}

<div style="color: orange; font-weight: bold;">
FYI: if you mispell anything for the `--erl` flags, then it will most likely fail silently.
</div>

In the above example, there is nothing stopping `MINPORT` and `MAXPORT` from being the same (e.g 9000).

If you've set things up correctly, your nodes will end up using the ports you want, rather than selecting
an arbitrary port from the ephemeral range.

You can test this with the <a href="http://www.erlang.org/doc/man/net_adm.html" target="_blank">net_adm</a> module:

{% highlight elixir %}
:net_adm.names
# => {:ok, [{'your-fun-node-name-here@hostname', 9000}]}
{% endhighlight %}

In this example, the final integer value would represent the port your node is listening on (9000).

After that, it's just a matter of connecting the nodes and sending sending some messages between them.  You'll need each node name to communicate, which you can get from the `node` function.

{% highlight elixir %}
# node 1
Node.connect(:'some-other-node-name@some-other-hostname')

server = spawn(fn ->
  receive do
    msg -> IO.puts "I got a message! #{inspect msg}"
  end
end)

:global.register_name(:server, server)
{% endhighlight %}

{% highlight elixir %}
# node 2
:global.whereis_name(:server) |> send "hello from node 2!"
{% endhighlight %}

This would result in `I got a message! hello from node 2!` being printed to stdout in node 1.

In reality, I wouldn't use `spawn` and `:global` in this manner. But this is a convenient way to demonstrate the the nodes
are connected and can communicate.

As I stated in the video, <a href="http://stackoverflow.com/questions/13868214/what-are-security-risks-when-running-an-erlang-cluster" target="_blank">you have to be careful with security here</a>. Owning one Elixir/Erlang node generally means you own all of them. Keeping as much as you can safe and secure behind your firewall is probably the the safest bet.

### Wrapup

The promise of easier distributed programming is one of the great selling points of Elixir/Erlang, and even simple examples such as this can be quite thrilling. And while the above examples are admittedly naive, I hope you found them useful.

If I got something wrong, please let me know in the comments below or send me an email at `jimtron9000` AT `gmail.com`.

Here are some other articles you might be intesrested in:

- <a href="http://learnyousomeerlang.com/distribunomicon" target="_blank">Distribunomicon</a>
- <a href="http://thesoftjaguar.com/posts/2014/06/18/rabbitmq-cluster/" target="_blank">My trip through RabbitMQ clustering</a>
- <a href="http://elixir-lang.org/getting-started/mix-otp/distributed-tasks-and-configuration.html" target="_blank">Distributed Tasks and Configuration</a>

<div class="cta">Did you find this post helpful? If so, <a href="/subscribe" target="_blank">you may want to consider subscribing</a>.</div>
