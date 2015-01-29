---
layout: post
title:  "Strong, Weak and Soft References -  Part 1"
date:   2015-01-22 11:00:00
---

## Strong Reference

A strong reference is an ordinary object reference, with nothing special about it.

##### Java:

{% highlight java %}
  Foo foo = new Foo();
{% endhighlight %}
#####  Ruby
{% highlight ruby %}
  foo = Foo.new
{% endhighlight %}


As you can see, there is nothing special about a strong reference. It's the kind object you use every day.

## Weak Reference

A weak reference is an object reference that is not strong enough to remain in memory after a [GC][gc] cycle.

How does it work?

The object is considered [weekly reachable][weakreach] when it's only reachable by a weak reference or chains of references
that include at least one weak reference. In this case, once the [GC][gc] analyzes the object and determine that it's weekly reachable, it will be deallocated.

How to create a weak reference?

##### Java:

{% highlight java %}
  WeakReference<Foo> weakFoo = new WeakReference<Foo>(new Foo());
//Returns the reference object, it might return null if the object has been deallocated.
weakFoo.get();
{% endhighlight %}

##### Ruby:
In Ruby you can use a gem called [ref][ref].

{% highlight ruby %}
  ref = Ref::WeakReference.new("hello")
#Returns the reference object or nil (if the object has been deallocated).
ref.object
{% endhighlight %}


## Soft Reference

A soft reference is almost identical to weak reference, but they can survive to a few GC cycles before to be deallocated.

#### Examples:

##### Java:

{% highlight java %}
  SoftReference<Foo> softFoo = new SoftReference<Foo>(new Foo());
//Returns the reference object, it might return null if the object has been deallocated.
softFoo.get();
{% endhighlight %}

##### Ruby

{% highlight ruby %}
  ref = Ref::SoftReference.new("hello")
#Returns the reference object or nil (if the object has been deallocated).
ref.object
{% endhighlight %}

[rc]: https://github.com/ruby-concurrency/
[ref]: https://github.com/ruby-concurrency/ref
[gc]: http://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29
[weakreach]: http://en.wikipedia.org/wiki/Unreachable_memory#weakly_reachable
