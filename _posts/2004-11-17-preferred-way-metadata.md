---
layout: post
title: Preferred way of specifying metadata
date: 2004-11-17 13:50:29.000000000 -08:00
categories: Technology
tags: []
status: publish
type: post
published: true
color: navy
---
I share [Dion's concern about metadata hell](http://www.almaer.com/blog/archives/000546.html). I think the key to using annotations correctly is to realize that annotations supply metadata i.e. additional data _about the element being annotated_. In other words, annotations should describe the element’s characteristics, and not what should happen before/after/around those elements. Here are some examples to illustrate the point.

In the first example, the apparent intention is to show the wait cursor around methods with the @WaitCursor annotation (using annotation processor or AOP etc.):

{% highlight java %}
// Not preferred
@WaitCursor
public void foo() {
}
{% endhighlight %}

The problem is that the method is carrying too implementation-specific metadata. A preferred way is to specify the expected time of execution for the method (many fine details omitted; see [AspectJ language support for metadata](http://ramnivas.com/blog/index.php?p=10) for more detailed examples):

{% highlight java %}
// Preferred
@Timing(avg=50)
public void foo() {
}
{% endhighlight %}

An annotation processor or an aspect may use the timing information to show the wait cursor around slow methods. The biggest difference is that the processors and aspects get to decide how to classify methods as slow and what needs to happen for the slow methods. For example, in a non-UI application, it may choose to do nothing. Further, the same information can be used for other purposes: [rate monotonic analysis (RMA)](http://www.sei.cmu.edu/str/descriptions/rma_body.html), connect with a monitoring agent to implement alerts, etc.

As a second example, the apparent intention is to introduce automatic fault tolerance through retries for retry-safe operations:

{% highlight java %}
// Not preferred
@RetryUponFailure
public void getBalance() {
}
{% endhighlight %}

A better way is to specify that operation is idempotent and let the annotation processors or aspects decide how to use this additional information:

{% highlight java %}
// Preferred
@Idempotent
public void getBalance() {
}
{% endhighlight %}

Finally, here is another example, where the programmer wishes to remove verbose code needed for [read-write lock management]( http://java.sun.com/j2se/1.5.0/docs/api/java/util/concurrent/locks/ReadWriteLock.html) through the use of annoations.

{% highlight java %}
// Not preferred
@ReadLock
public float getBalance() {
}

@WriteLock
public void credit(float amount) {
}{% endhighlight %}

A better way is to specify if methods are modifying object’s state:

{% highlight java %}
// Preferred
@ReadOperation
public float getBalance() {
}

@WriteOperation
public void credit(float amount) {
}{% endhighlight %}

That said, there are cases where the additional information makes sense only in the context of specific concerns. In those cases, the scenario becomes a little murky. Consider for example, situation where database table names are specified using annotations:

{% highlight java %}
@Persistence(table="myTable")
public class MyClass {
}
{% endhighlight %}

The `Persistence` annotation makes sense only in the context of persistence concern and not when `MyClass` is considered in isolation. The question to ponder over: Should such metadata be specified in the class itself? The issue with this example is not surprising given debate over the best place to put such annotations -- doclet tags or XML mapping document. It will be interesting to watch how the Java community ends up using metadata.
