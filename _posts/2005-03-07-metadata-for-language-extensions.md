---
layout: post
title: Metadata for language extensions
date: 2005-03-07 22:41:07.000000000 -08:00
categories:
- Technology
tags: []
status: publish
type: post
published: true
color: navy
---
My two-part article series "[AOP and metadata: A perfect match](http://www.ibm.com/developerworks/java/library/j-aopwork3/)" _[Update: Link added, since the article is now published]_, a part of the [AOP@Work](http://www.ibm.com/developerworks/views/java/libraryview.jsp?search_by=AOP@work:) series, is about to go live on [developerWorks](http://www.ibm.com/developerworks). The first part of the series includes a small section on using metadata to extend the underlying language. However, this topic needs a more elaborate discussion, and hence this blog.

The [new metadata facility in Java 5.0](http://www.ibm.com/developerworks/java/library/j-annotate1/index.html) can be used to extend the Java language. In this usage model, an annotation processor alters the semantics of program elements marked with special annotations. Such usage of metadata brings interesting possibilities and a few concerns.

The idea of using metadata to extend the language is not new. AOP systems such as [AspectWerkz](http://aspectwerkz.codehaus.org), [JBoss AOP](http://www.jboss.org/developers/projects/jboss/aop), and now the metadata-driven syntax in [AspectJ5 ](http://dev.eclipse.org/viewcvs/indextech.cgi/~checkout~/aspectj-home/doc/ajdk15notebook/ataspectj.html)(informally called @AspectJ syntax) use this idea to provide aspect-oriented extensions to Java, in which ordinary program elements are reinterpreted as AOP constructs. For example, you can mark a class to function like an aspect, a method as a pointcut or advice, and so on.

## Extension classification

We can classify language extensions using metadata into three categories based on how they affect the programs: compile-time extensions, structural modifications, and behavioral modifications.

### Compile-time extensions

These extensions make the compiler perform additional tasks but do not affect the compiled code. Java 5.0 already includes examples of this with [`@Override`](http://java.sun.com/j2se/1.5.0/docs/api/java/lang/Override.html), [`@Deprecated`](http://java.sun.com/j2se/1.5.0/docs/api/java/lang/Deprecated.html) and [`@SuppressWarnings`](http://java.sun.com/j2se/1.5.0/docs/api/java/lang/SuppressWarnings.html) annotations (albeit as a standard feature, rather than an extension).

Generalizing this idea, you can imagine extending the compiler to issue custom errors and warnings upon detecting certain usage patterns. For example, we can use a `Const` annotation type to implement the functionality of the [C++ "const" keyword](http://www.parashift.com/c++-faq-lite/const-correctness.html) in Java. Consider the following annotation type:

{% highlight java %}
public @interface Const {
}
{% endhighlight %}

Now consider the following class that uses this annotation:

{% highlight java %}
public class ShoppingCart {
    ...

    public void addItem(@Const Item item) {
        // can't directly modify item or call methods that could modify item
    }

    public void removeItem(@Const Item item) {
        // can't directly modify item or call methods that could modify item
    }

    @Const
    public float getTotal() {
        // can't directly modify 'this' or call methods that could modify 'this'
        return 0;
    }
}
{% endhighlight %}

The annotation processor would consume the `@Const` annotations to produce compile-time errors for any attempts to modify an item while adding or removing from a shopping cart or any attempts to modify the state of shopping cart while querying for the total.

Okay, the language extension using `@Const` is only a _theoretical possibility_, since I don't think it will be of much practical value unless the core Java classes and existing libraries adopt this annotation type and its semantics. Nevertheless, it is an _interesting theoretical possibility_.

### Structural modifications

These extensions modify the structure of the program without modifying the behavior. For example, we could use a `@Property` annotation to mark fields of a class as properties. An annotation processor then processes the annotations to add a getter method and/or a setter method for each property based on whether it is a read-write, read-only, or write-only property.  Essentially, with this extension, you get the property feature support similar to the one in [Groovy](http://groovy.codehaus.org/Groovy+Beans) and [C#](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/csspec/html/vclrfcsharpspec_10_6.asp).

Consider the following annotation type and the associated enumeration type:

{% highlight java %}
public @interface Property {
    public PropertyKind value() default PropertyKind.ReadWrite;
}
{% endhighlight %}

{% highlight java %}
public enum PropertyKind {
    ReadWrite, ReadOnly, WriteOnly;
}
{% endhighlight %}

Now consider the following class:

{% highlight java %}
public class Person {
    @Property(ReadOnly) private long id;
    @Property private String name;
}
{% endhighlight %}

The annotation processor would consume the `@Property` to translate this class into a byte-code equivalent of the following snippet:

{% highlight java %}
public class Person {
    @Property(ReadOnly) private long id;
    @Property private String name;

    public long getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
{% endhighlight %}

If a getter or setter already existed for a property, the annotation processor would leave it untouched.

We can use the same idea to support certain structural design patterns directly into the extended language. For example, we can implement the [singleton pattern](http://www.javaworld.com/javaworld/jw-04-2003/jw-0425-designpatterns.html) by allowing classes to be marked with an annotation. Consider the following annotation and enumeration types:

{% highlight java %}
public @interface Singleton {
    public SingletonKind value() default SingletonKind.Lazy;
}
{% endhighlight %}

{% highlight java %}
public enum SingletonKind {
    Lazy,   // create the singleton instance upon first use
    Eager;  // create the singleton instance as soon as possible
}
{% endhighlight %}

Now consider a class marked with a `@Singleton` annotation:

{% highlight java %}
@Singleton(Lazy)
public class Service {
    public void serve() {
        //...
    }
}
{% endhighlight %}

The annotation processor will translate this class into a byte-code equivalent of the following snippet:

{% highlight java %}
@Singleton(Lazy)
public class Service {
    private static Service _instance;

    public static Service instance() {
        if(_instance == null) {
            _instance = new Service();
        }
        return _instance;
    }

    private Service() {
    }

    public void serve() {
        ...
    }
}
{% endhighlight %}

The annotation processor could also modify compile-time behavior to flag the presence of unsupportable structures such as non-zero arguments constructors.

### Behavior modifications

These extensions alter the program behavior. For example, they may add additional code to implement security feature in all the program elements with a `@Secure` annotation. EJB 3.0 annotations essentially extend the Java language in the same vein. Another good example of this kind of extension is [ContractJ](http://www.contract4j.org) that implements DBC in Java.

While implementing behavior modifications through language extension can be useful in some situations, aspect-oriented programming is a much better way to do the job. When implemented using AOP, the behavior logic doesn't disappear into code in an annotation processor, the code expressed remains much more readable, and the debugging process remains natural.

## Extension implementation

While some of the features described here can be implemented through a pre-processor (generating code) or post processor (modifying byte-code), a more convenient way would be to allow plugins to the compiler. It would not be surprising if a future tool's JSR proposes a standard way to extend the compiler (perhaps called "complet", in the same spirit as "doclet" to use with Javadoc). Perhaps the Pluggable Annotation Processing API (JSR 269) might be the one to standardize such a facility.

## Impact on Java

This possibility allows incorporating new features in the language without having to wait for them to make it into the standard. However, abusing this possibility can wreak havoc on the comprehensibility of the programs. Using this possibility to create principled extensions will make Java a more expressive language. AOP systems using annotations to add aspect-oriented features to Java is a good example of principled extensions (in that there is a system behind it -- aspect-oriented programming). Using this possibility to add ad hoc extensions, on the other hand, will make programs hard to follow without the context of the associated annotation processing.

How the community uses the metadata feature in general, and the language-extension use case in particular, will be interesting to watch.
