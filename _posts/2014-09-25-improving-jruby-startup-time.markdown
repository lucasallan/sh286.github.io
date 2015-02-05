---
layout: post
title:  "Improving JRuby startup time - From 36s to 07s"
date:   2014-09-25 14:49:00
---

One of the most frequent complaints about JRuby is the startup time.
But with a few configuration flags you can decrease the JRuby startup time.

Let's start with an example application (Rack + Grape + ActiveRecord):

Running Rake to measure how much time JRuby takes to load the application.

{% highlight bash %}
>$ time rake -T
36.69s user 1.48s system 282% cpu 13.524 total
{% endhighlight %}


The result is terrible, 36.69 seconds to load a simple application is not something that you would like to deal with every day.

### InvokedDynamic
The JVM provides an optimization called InvokedDynamic that speeds up steady state operation but on the other hand it really slows down startup. This is an really important optimization, but there is no big reason to use it in your development environment.

### Tiered compilation

[Tiered compilation][tiered_compilation] is something that can really slow down the startup time due to the optimizations that is made during the compilation.

### JRuby's JIT

For short runs, [JIT][JIT] can really slows down the execution (compilation is not a free operation).

Now let's take a look in the results after disable Tiered compilation, InvokedDynamic and JIT.

{% highlight bash %}
>$ export JAVA_OPTS='-Djruby.compile.mode=OFF -XX:+TieredCompilation -XX:TieredStopAtLevel=1'
>$ export JRUBY_OPTS='-Xcompile.invokedynamic=false'
>$ rake -T
8.65s user 0.44s system 184% cpu 4.932 total
{% endhighlight %}


As you can see in the results, now the startup time is 4.24 times faster.

### jruby-launcher

If you're using the [jruby-launcher][jruby-launcher], you can simplify all these options just using the flag --dev.

### DRIP

[Drip][drip] is a command line launcher for the Java Virtual Machine that provides faster startup times than the java command.
[Here][drip-install] you can find instructions about how to configure [Drip][drip].

Now the results with Drip:
{% highlight bash %}
>$ rake -T
11.27s user 0.71s system 185% cpu 6.451 total
>$ rake -T
0.07s user 0.09s system 3% cpu 4.655 total
>$ rake -T
0.07s user 0.09s system 3% cpu 4.593 total
>$ rake -T
0.07s user 0.10s system 3% cpu 4.756 total
{% endhighlight %}

The first time you run your application is a little slower because it will start the jvm in background.
As you can see in the results, after the first time the startup time is 5.2 times faster.


[drip]: https://github.com/ninjudd/drip/wiki/JRuby
[jruby]: http://jruby.org
[tiered_compilation]: http://docs.oracle.com/javase/7/docs/technotes/guides/vm/performance-enhancements-7.html
[JIT]: http://en.wikipedia.org/wiki/Just-in-time_compilation
[jruby-launcher]: https://rubygems.org/gems/jruby-launcher
[drip-install]: https://gist.github.com/rwjblue/4582914
