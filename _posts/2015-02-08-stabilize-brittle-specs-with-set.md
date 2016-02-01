---
layout: post
title:  "Stabilize Brittle Specs with Set"
date:   2015-02-08 18:56:27
categories: Ruby
comments: true
---

### Overview

_This is a beginner-to-intermediate level tutorial._

_Time-to-complete is probably around 5 minutes._

### The Problem

You want to test equality on an array or an array-like object with *unique* elements, but
the order of the result you are testing is non-deterministic. You're doing some sort of integration spec
where you don't want to stub out the service call - perhaps something like
a feature spec that queries your Redis instance.

A simplified example:

{% highlight ruby %}
describe Service do
  it 'has the results I expect' do
    results = Service.get('fubar:members')

    expect(results).to eq(['foo:1', 'foo:7', 'foo:42'])
  end
end
{% endhighlight %}

This works fine, except for when the Service occasionally returns the results in a different order.

If the results from `Service.get` happen to return `['foo:7', 'foo:1', 'foo:42']`
then your spec would fail.

To get around this, you'll see test cases that have this shape:

{% highlight ruby %}
describe Service do
  it 'has the results i expect' do
    results = Service.get('fubar:members')

    expect(results).to include('foo:1')
    expect(results).to include('foo:7')
    expect(results).to include('foo:42')
  end
end
{% endhighlight %}

This approach works, but if you're like me,
you try to [stick to one assertion per spec](http://blog.jayfields.com/2008/01/testing-one-expectation-per-test.html)

### Solution

If you're not concerned with order and you're only interested in testing for
membership, then Ruby's [Set](http://www.ruby-doc.org/stdlib-2.2.0/libdoc/set/rdoc/Set.html)
will do the trick.

You can initialize a set with an array using `Set.new(your_array)`.

And as you can see, while the ordering of elements does determine equality
in an Array, it does not for a Set.

{% highlight ruby %}
# order matters to an array
%w(a b c) == %w(c b a)
# => false

# but not to a set
require 'set'
Set.new(%w(a b c)) == Set.new(%w(c b a))
# => true
{% endhighlight %}

We can rewrite our sample spec like this:

{% highlight ruby %}
describe Service do
  it 'has the results I expect' do
    raw_results = Service.get('fubar:members')

    results     = Set.new(raw_results)
    expectation = Set.new(['foo:1', 'foo:7', 'foo:42'])

    expect(results).to eq(expectation)
  end
end
{% endhighlight %}

Given our problem (unique elements with non-deterministic ordering), this
version of our spec solves our problem.

### Closing

A great point can be made of not hitting _any_ sort of live service from your
tests to begin with.

If your test data is coming from a fixture (stub, [vcr](https://github.com/vcr/vcr), [factory girl](https://github.com/thoughtbot/factory_girl),
[fakeredis](https://github.com/guilleiguaran/fakeredis), etc etc), then the order is
determined to begin with, and these sorts of non-deterministic spec problems are a non-issue.

Another point is that, admittedly, this solution only really works for when you're
wrapping results sets that you expect to have unique elements. Calling `Set.new([1,1,1])`
is somewhat akin to calling `[1, 1, 1].uniq`, so you'll want to make sure that a
Set really is a good fit for your data.

Still, it's a great tool to have in your toolkit. You'll sometimes see these
issues pop up in certain types of integration specs or when your lib code is internally
using Sets.

Obviously, they are also helpul in regular application code when the order of your data
doesn't matter, and you only want to ingest unique elements.

In these cases and quite a few others, you'll be glad to have [Set](http://www.ruby-doc.org/stdlib-2.2.0/libdoc/set/rdoc/Set.html)
at your fingertips.

### Update (12/28/2015)

As some people <a href="https://www.reddit.com/r/ruby/comments/2v9nis/using_rubys_set_class_to_stabilize_brittle_specs/">pointed out</a>, RSpec comes with <a href="http://www.rubydoc.info/github/rspec/rspec-expectations/RSpec/Matchers:match_array">match_array</a>, which would negate the need for using Set.

I probably made a mistake saying fix brittle _specs_, as my intention was to be a reminder of what you can do with the <a href="http://ruby-doc.org/stdlib-2.3.0/">RSL</a> rather than being a tutorial on RSpec.

Still, `match_array` is pretty handy when you are writing specs. I've used it at least once since writing this post.

In the case when you want a framework agnostic way to achieve the same results, or if you're writing your own matcher, then Set would be useful.

-- Jim

<div class="cta">Did you find this post helpful? If so, <a href="/subscribe" target="_blank">you may want to consider subscribing</a>.</div>
