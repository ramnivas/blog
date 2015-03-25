---
layout: post
title: AspectJ language support for metadata
date: 2004-11-08 10:02:27.000000000 -08:00
categories: Technology
status: publish
type: post
published: true
color: navy
---
Metadata and aspect-oriented programming (AOP) is an [interesting combination]( http://javaoneonline.mentorware.net/servlet/mware.servlets.StudentServlet?mwaction=showDescr&class_id=27775&fromtopic=By%20Topic&subsysid=2000&topic=technical&avail_frames=true). I am particularly interested in how the metadata facility will impact the [AspectJ language](http://eclipse.org/aspectj). So I gathered all my thoughts in form of a proposal to modify the AspectJ language to support metadata.

At a high level, metadata support in AspectJ requires:

*   Language support for consuming annotations
    * Selecting join points based on annotations associated with program elements
    * Capturing the annotation instances associated with the captured join points using
        1.  pointcuts
        2.  reflective APIs
*   Language support for supplying annotations
    * Attaching annotations to join points in a crosscutting manner

Language extension to support metadata in AspectJ should be fully backwards compatible and flow well with the current language. This requires following two principles:

*   Introduce no new keywords, if possible.
*   Align with the existing pointcuts and APIs as much as possible.

The proposed syntax can be divided into four parts: extending the signature syntax to allow specifying annotations, adding new pointcuts to allow selecting join points based on annotation associated with them as well as capturing annotation instances as pointcut context, syntax to supply annotations, and modification to reflection APIs to access annotations.

## Extending the signature syntax

Adrian Coyler, the AspectJ team lead, has written [a design note]( https://bugs.eclipse.org/bugs/show_bug.cgi?id=72766) to extend signature patterns in AspectJ to select join points based on metadata. I have taken Adrian’s idea and extended it to cover more complex cases such as matching based on annotation properties:

Let’s dive directly into a few examples to illustrate the new syntax.

<table border="1">
<tr>
<td>Pointcut</td>
<td>Join points matched</td>
</tr>
<tr>
<td>{% highlight java %}execution(@Idempotent public * *.*(..)){% endhighlight %}</td>
<td>Execution of methods that carry an <code>@Idempotent</code> annotation (such as <code>@Idempotent public long getId()</code>)</td>
</tr>
<tr>
<td>
{% highlight java %}execution(public * *.*(@ReadOnly *, ..)){% endhighlight %}</td>
<td>
Execution of methods whose first argument carries a <code>@ReadOnly</code> annotation
  </td>
</tr>
<tr>
<td>
{% highlight java %}set(* @ContainerSupplied dataSource){% endhighlight %}</td>
<td>
Write-access to all fields named `dataSource` marked with a <code>@ContainerSupplied</code> annotation
  </td>
</tr>
<tr>
<td>{% highlight java %}within(@Secure *){% endhighlight %}</td>
<td>
All join points inside any type with a <code>@Secure</code> annotation
  </td>
</tr>
<tr>
<td>{% highlight java %}within(@Secure * && @Persistent *){% endhighlight %}</td>
<td>
All join points within lexical scope of all types that have a <code>@Secure</code> and a <code>@Persistent</code> annotation
  </td>
</tr>
<tr>
<td>{% highlight java %}within(@Secure *..*){% endhighlight %}</td>
<td>
All join points within lexical scope of all types that have a direct or indirect parent package with a <code>@Secure</code> annotation
  </td>
</tr>
<tr>
<td>{% highlight java %}within(@Secure *.@Sensitive *..*){% endhighlight %}</td>
<td>
All join points within lexical scope of all types that have a direct or indirect parent package with a <code>@Secure</code> annotation whose child package carries a <code>@Sensitive</code> annotation
  </td>
</tr>
</table>

I think `within()` with just annotation type as type pattern such as `within(@Secure)` should be a compile-time error, since it does not make sense to capture anything in the annotation type itself -- compiler generated code for property access in annotation types is irrelevant from user perspective.

Next, we need syntax for matching join points based on annotation property values and not just annotation types. Note that while the proposal to capture annotation instances using pointcuts can achieve a similar effect, those pointcuts will probably fall into dynamically determinable pointcut category. Here are a few examples that capture join points based on annotation properties.

The following pointcut matches all methods that carry a `@Transactional` annotation:
{% highlight java %}
    execution(@Transactional *.*(..))
{% endhighlight %}

The following pointcut restricts the selection to only methods that have the `kind` property set to `Required`:
{% highlight java %}
    execution(@Transactional(kind==Required) *.*(..))
{% endhighlight %}

The following pointcut restricts the selection to only methods that have the `kind` property set to either `Required` or `RequiredNew`:
{% highlight java %}
   execution(@Transactional(kind==Required || kind==RequiredNew) *.*(..))
{% endhighlight %}

In short, one may specify any Java boolean expression that uses properties and constants to filter the join point selection. Here is a pointcut indicative of complex pointcuts one may write :-):

{% highlight java %}
    pointcut longRunningTransactionalRequiredOps()
        : execution(@Transactional(kind==Required || kind==RequiredNew)
                    @Timing((min > 30 || max > 100 || ((max-min) > 50))
                             || (avg > 40 && variance > 0.35)) *.*(..))
{% endhighlight %}

For wildcard treatment, annotation types should be considered just like other types. Note that since the Java language does not permit inheritance with annotation types, the ‘+’ wildcard won’t make sense (I think it should be a compile-time error to specify annotation type with the ‘+’ wildcard).

## New pointcuts

New proposed pointcuts allow select join points based on annotations as well as capturing annotation instances as pointcut context by extending the current `this()`, `target()`, and `args()` pointcuts.

* `@interface(AnnotationTypeOrIdentifier)` to capture annotations attached to the program element corresponding to the captured join points themselves. (Note to those attended/viewed my JavaOne presentation: I used `annotation(AnnotationTypeOrIdentifier)` in that presentation. After more thought, it seems a good idea to avoid a new keyword. While `annotation()` seems clearer, once Java developers get used to defining annotation types using `@interface` syntax, the `@interface()` pointcut may feel more natural.)
* `@this(AnnotationTypeOrIdentifier)` to capture annotations attached to the `this` object associated with the captured join point.*   `@target(AnnotationTypeOrIdentifier)` to capture annotation attached to the target object associated with the captured join point.
* `@args(AnnotationTypeOrIdentifier1, AnnotationTypeOrIdentifier2, ...)` to capture annotation attached to the arguments associated with the captured join point.

Like `this()`, `target()`, and `args()` pointcuts, the proposed pointcuts take two forms: one that specifies a type and another that specifies an identifier. The first form is used to make join point selection only, whereas the second form is used to also collect the annotation instance as a context. For example, the following pointcut matches all the join points with `@Transactional` annotation:

{% highlight java %}
pointcut transactedOps() : @interface(Transactional);
{% endhighlight %}

whereas the following also collects the annotation instance to use inside advice (just like any other context)

{% highlight java %}
pointcut transactedOps(Transactional tx)
    : @interface(tx);
{% endhighlight %}

The context collected in this fashion may be used in an `if()` pointcut. For example, the following pointcut captures only join points that have `@Transactional` annotation with the kind property set to `Required`.

{% highlight java %}
pointcut transactedRequiredNewOps(Transactional tx)
    : @interface(tx) &amp;&amp; if(tx.kind() == RequiredNew);
{% endhighlight %}

Of course, an advice may use collected annotation instances just like any other collected context:

{% highlight java %}
Object around(Transactional tx) : transactedOps(tx) {
    TransactionalKind txKind = tx.value();
    ...
}
{% endhighlight %}

Since an element may carry more than one type of annotations (but never two annotations of the same type), the proposed syntax allows using `@interface`, `@this`, `@target`, and `@args` multiple times in the same pointcut. The following pointcut captures join point with both `@Transactional` and `@Idempotent` annotations:

{% highlight java %}
pointcut idempotentTransactional()
    : @interface(Transactional)
      && @interface(Idempotent)
{% endhighlight %}

## New declare annotations syntax

This new syntax allows attaching annotations to program elements in crosscutting fashion. This syntax resembles the existing `declare soft` syntax. [Adrian Coyler has blogged about this syntax]( http://www.aspectprogrammer.org/blogs/adrian/2004/08/when_is_a_pojo.html). I have also presented the same idea in JavaOne ‘04. The only difference in the syntax proposed here is switching places between the annotation type and the pointcut to better match the `declare soft` syntax:

{% highlight java %}
declare annotations: <AnnotationInstance> : <pointcut expression>;
{% endhighlight %}

As an example, the following declaration associated a new `@Transactional` annotation with property `kind` set to `Required` to the `Account.debit()` method:

{% highlight java %}
declare annotations: @Transactional(kind=Required)
    : execution(public void Account.debit(..))
      || execution(public void Account.credit(..));
{% endhighlight %}

It should be probably a compile-time error if declare annotations statement would result in multiple annotations of same type for a program elements (much in line with declare parents producing errors if multiple inheritance of classes would result).

## Modifications to reflection API

Add new methods in `org.aspectj.lang.JoinPoint` interface to access annotation instance reflectively.

{% highlight java %}
public interface JoinPoint {
    ... current methods ...

    public <A extends Annotation> A getAnnotation(Class<A> annotationClass);
    public Annotation[] getAnnotations();

    public <A extends Annotation> A getThisAnnotation(Class<A> annotationClass);
    public Annotation[] getThisAnnotations();

    public <A extends Annotation> A getTargetAnnotation(Class<A> annotationClass);
    public Annotation[] getTargetAnnotations();

    public <A extends Annotation> A[] getArgsAnnotation(Class<A> annotationClass);
    public Annotation[][] getArgsAnnotations();
}
{% endhighlight %}

That’s it for now.
