---
layout: post
title: An example of Scala literal types 
date: 2020-02-05 15:00:00.000000000 -08:00
categories: Technology
status: publish
type: post
published: true
color: navy
---
One of the new features in [Scala 2.13](https://github.com/scala/scala/releases/tag/v2.13.0) is [literal types](https://docs.scala-lang.org/sips/42.type.html). In this blog, I will show a simple example to appreciate the usefulness of this feature.

# Literal Types in short
A type describes the category of a related set of objects. For example, the `Int` type describes integers such as 0, 1, 42, -30 and so on. The type of such a value is, well, the type.

```scala
val i: Int = 1
```

With literal types, the type of value is the specific literal used to initialize it. In the following, we specify the type to be the literal `1` and not `Int`.

```scala
val one: 1 = 1
```

You can use any primitive value as well as any String as a literal type.

```scala
val hello: "hello" = "hello"
```

You can assign values with literal types to their "regular" types.

```scala
val num: Int = one
val str: String = hello
```

But you may not do it the other way round:

```scala
val two: 2 = 1 // ERROR 

val second: Int = 2
val three:3 = second // ERROR
// Not even...
val s: 2 = second // ERROR
```

At first sight, this feature feels odd and not very useful. After all, the only value you can ever assign to a value with literal type is the value itself. So what's the point?

The motivating examples listed in the [SIP]((https://docs.scala-lang.org/sips/42.type.html)) illustrate the usefulness of this feature. However, those examples require a bit of head-scratching to understand and give a feeling that this feature is useful only for advanced type-level programming provided by Shapeless and such. However, that is not so. 

# A simple example of literal types
Let's look at an example inspired by [TypeScript](https://www.typescriptlang.org/docs/handbook/advanced-types.html#string-literal-types) (which has been supporting literal types for a few years now). 

In the Scala.js DOM API, we have the [createElement](https://www.scala-js.org/api/scalajs-dom/0.9.5/index.html#org.scalajs.dom.raw.Document@createElement(tagName:String):org.scalajs.dom.raw.Element) method to create a DOM elements:

```scala
def createElement(tagName: String): Element
```

The typical use of this API is as follows (this example is lifted directly from our code base for [LearnRaga](https://learnraga.com)).

```scala
val script = document.createElement("script").asInstanceOf[HTMLScriptElement]
script.src = src
script.async = true
```

Here the knowledge that if we supply "script" as the argument, we get back an `HTMLScriptElement` is something not enforced by the code and requires casting before we set the element's attributes. If we were to use literal types, we can avoid the typecast, simplifying use code as follows.

```scala
val script = document.createElement("script")
script.src = src
script.async = true
```

How can this be? With literal types, we can create overridden versions of `createElement` to return the specific type of the created element.

```scala
def createElement(tagName: "script"): HTMLScriptElement = js.native
def createElement(tagName: "a"): HTMLLinkElement = js.native
... more such createElement with literal types

// Fallback for any specific tag we may not have considered
def createElement(tagName: String): Element = js.native
```

That’s it. We now have a simpler API that expresses the relationship between the tag supplied and the type of the element produced. And we know a simpler motivating example of the literal type feature!
