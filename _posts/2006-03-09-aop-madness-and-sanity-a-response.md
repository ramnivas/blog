---
layout: post
title: 'AOP Madness and Sanity: A Response'
date: 2006-03-09 06:07:04.000000000 -08:00
categories:
- Technology
tags: []
status: publish
type: post
published: true
color: navy
---
Graham Hamilton has blogged about [AOP: Madness and Sanity](http://weblogs.java.net/blog/kgh/archive/2006/03/aop_madness_and.html). The central theme of the blog seems to be based on a notion that any language that runs on the Java _platform_ must follow the JLS and it is okay for a J2EE container, but not for aspects, to affect program semantics.

> Yes, J2EE has been using AOP all along!

This is a myth. The reality is that J2EE has been helping with _certain_ crosscutting concerns such as transaction management and security _in a rigid, pre-defined way_. AOP is a much more general approach, which works not only for J2EE-like applications, but also for Swing/SWT UI applications and real-time applications etc. Aspects are useful even along with J2EE containers, where the container falls short in meeting project's needs, as witnessed by the success of the [Spring Framework](http://www.springframework.org) using AOP to improve on J2EE.

> As far as possible we have tried to follow the principle that "what you get is what you see".

This might have been a reasonable argument before Java 1.3. With the addition of dynamic proxies, this argument now works against Java. If I see a method call to an object, I cannot say anything unless I know the proxies in which the object is wrapped. On the other hand, aspects along with tools, very directly show you that there is an advice being applied there. See screenshots in [New AJDT releases ease AOP development](http://www.ibm.com/developerworks/java/library/j-aopwork9/index.html) for examples of these.

The code-that-you-write is the code-that-gets-executed is a thing of a distant past. There is so much post-compilation byte code modification that there is a competition among byte code engineering tools! With JVMTI in Java 5, there is even a blessed mechanism to modify the code while being fed to the VM. I don't think this "what you get..." argument is even worth putting forward anymore.

> The container can intercept incoming calls and add additional semantics. An example of this approach is the way that Java EE containers can add transactional semantics to EJB components by intercepting incoming calls.

And how does JLS come into the picture here (your next point)? Why are these containers more blessed than others? What is the difference between (the presumed good) additional semantics and (the presumed evil) _changed_ semantics? What if I don't have a blessed container that meets my needs? For example, what should I do as a poor Swing application developer, who is trying to write a responsive application using multiple threads and end up with a ton of duplicated boilerplate code to route calls to the event dispatcher thread? Would it be so bad if I can write a few aspects that make me more productive immediately or should I wait for some blessed container (or its third version!) to help me?

> Some uses of AOP rely on using runtime mechanisms to modify semantics that are specified by the Java Language Specification (JLS). For example, side files might specify that various modifications are to be made to target classes to invoke additional code when various operations are performed. Some uses of AOP allow extra code to be invoked when fields are accessed or when methods are called.

AOP systems have their own semantics and language specifications. For example, AspectJ has semantics of interaction between classes and aspects. One needs to understand those semantics associated with the aspects to understand the overall system behavior. In an AOP system, aspects are a first-class participant, not a "side file" at all.

Understanding a Java-based AOP system only in terms of Java is similar to understanding C++ in terms of C semantics. You can't make the required leap if you limit yourself to expressing classes in terms of structures and method dispatch in terms of invoking an array of function pointers. Similarly, thinking about AOP in terms of JLS and code modification won't help in understanding the real value of AOP. Indeed, thinking about Java semantics by considering machine code instructions is similarly not helpful: the additional abstractions provide software engineering benefits. The use of byte code modification or the use of proxies is simply an implementation mechanism for AOP.

This criticism of AOP is similar to how procedural crowd responded to OOP. Sure, OOP did obscure the program flow, but we ended up liking the "obscurity". See my response to myth [Aspects obscure program flow](http://www.ibm.com/developerworks/java/library/j-aopwork15/#N10241) for more thoughts.

The real value of AOP is in directly expressing design intents in programming constructs. If my security requirement is to check permission before any publicly accessible method in a class in banking or its subpackage, AOP allows me to map this requirement directly in an aspect such the following. I can now review the implementation and check its adherence to the requirements. If any requirements change, I can simply modify the aspect.

{% highlight java %}
public aspect BankingSecurity {
    pointcut bankingOperation() : execution(* banking..*.*(..));

    before() : bankingOperation() {
        AccessController.checkPermission();
    }
}
{% endhighlight %}

Whether this aspect modifies byte code of the selected methods, or a proxy is created that implements the advice, or a VM takes care of dispatching the advice is a secondary consideration.

> The JLS carefully specifies the precise semantics of the Java language.

And so does &lt;An-AOP-system&gt;LS. It is just different than the JLS. Fortunately for the Java community, Sun has been supporting alternative languages for the Java platform.

> To take an extreme example, if a developer has written a statement "a = b;" then they should be able to rely on the value of the field "b" being assigned to the field "a", with no side effects. Some AOP toolkits allow arbitrary extra code to be executed when this statement runs, so that there can be arbitrary extra side effects as part of this assignment or even so that an arbitrary different value may be assigned to "a".

The join point models of AOP systems are pretty restrictive on what you can advise, based on writing maintainable code. In fact, that is how it stands apart from generic byte code modification tools and metaprogramming. AOP's disciplined approach prevents developers from writing hard-to-maintain programs (within reason, of course, since a determined individual can always write bad program in any language or system).

So let's consider the "`a=b;`" example. First of all, you can advise such a statement only if it represents an exposed join point. So for AspectJ, "`a`" or "`b`" must be a field of a class: local variables won't do. Second, is it so arbitrary if I have a "side effect" of notifying observers of state-modification implemented through an aspect? Or is it so arbitrary if I mark the object holding "`a`" as dirty? In each of these cases, I can modularize observer and dirty-tracking functionality into an aspect rather than scattering it in all the places that modify the state of an object.

> Yikes! Similarly, some uses of AOP permit extra code to be inserted on a method call from one method to another, even when the JLS clearly specifies that that method call will occur directly. This enables arbitrary extra code to be executed, with potentially arbitrary side effects, in situations where a Java developer should expect there will be no side effects.

So it is okay for a container to invoke extra code to implement security and transaction management (which may very well use byte-code modification as an implementation technique), but it is not okay for an aspect to do the same thing? Once again, what gives a container a special status? Not the JLS.

"Yikes" would be a response of a Java programmer unaware of an AOP system being used, but it would be a perfectly normal behavior to, say, an AspectJ developer.

> In J2SE 5.0, the concept of annotations was added to the Java language. In many ways, annotations were designed to meet similar goals to AOP. The concept of annotations is that developers may apply special markers to target method, fields, or classes in order to designate that they should be processed specially by tools and/or runtime libraries.

So will it be okay if that "tools and/or runtime libraries" is an AOP compiler (like AspectJ) or an AOP system (like Spring)?

See discussion of myth [Annotations obviate AOP](http://www.ibm.com/developerworks/java/library/j-aopwork15/#N10224) for a quick response. There is a lot to be said here about AOP's use of metadata. See "AOP and metadata: A perfect match", [Part 1](http://www.ibm.com/developerworks/java/library/j-aopwork3/index.html) and [Part 2](http://www.ibm.com/developerworks/java/library/j-aopwork4/index.html).

> The use of container based AOP seems to provide a reasonable mechanism for supporting AOP "cross cutting concerns" within Java environments, without breaking developers' expectations of how Java source code behaves.

Knowing the JLS is not sufficient to understand the behavior of a class in a container. For example, if I see an EJB with security attributes specified in its deployment descriptor, would a Java developer expect its methods to throw an authorization exception? See my more detailed response to myth [Application frameworks obviate AOP](http://www.ibm.com/developerworks/java/library/j-aopwork15/#N101F5).

Java has been a great language and platform. However, it is coming to a point where some serious innovation is needed to ensure that it remains the front runner. AOP is one such innovation that exemplifies the capability of the core Java platform. AOP addresses crosscutting concerns without having to wait for someone to provide a framework and learn each framework and its versions anew. However, exploiting AOP subject to a strict adherence to the limited approaches of the past is unhelpful.
