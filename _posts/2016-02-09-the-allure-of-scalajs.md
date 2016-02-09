---
layout: post
title: The allure of Scala.js
date: 2016-02-09 11:00:00.000000000 -08:00
categories: Technology
status: publish
type: post
published: true
color: navy
---
Recently when I started working on my startup, I needed to choose a set of technologies that will make our small team effective. From a high-level architectural perspective, our product is a fairly typical modern app: it has an API backend, a single-page web app, and mobile apps for iOS and Android. Having used Scala successfully for many years, the choice on the backend was obvious. But when it came to the frontend (web and mobile apps), we decided to use Scala through [Scala.js](http://scala-js.org) as well. Given that Scala.js is a fairly new technology (the current version is 0.6.6), I had some trepidations on choosing it. A few months in, I am more than happy to have made that choice. So wanted to share what I find so attractive about Scala.js.

# Scala.js in short
Scala.js offers a way to compile Scala code to JavaScript. You can use most of the Scala language (approximately all language features except most of the runtime reflection). You can integrate existing JavaScript libraries and frameworks with ease (we use [React](https://facebook.github.io/react/), through [scalajs-react](https://github.com/japgolly/scalajs-react), for example). While developing, you compile your project in the "fast optimization" mode, which favors faster build time over smaller code size. For the production build, you use the "full optimization" mode, which takes a bit longer, but produces code that favors smaller output code size (the JS file produced for our app, as it stands today, is about 180 KB gzipped, excluding external libraries such as React). Given that you write code in Scala, you gain all the benefits of Scala: static typing, functional programming, nice collections library, sane ways to deal with async computation, and so on.

# Things going for Scala.js
If a tool were to become successful, the target community must be already craving for the benefits offered. Otherwise, it might be a right tool at a wrong time. A Scala developer faced with a task of developing frontend will, without too much prodding, see Scala.js as an obvious choice. But a real interesting question is will a JavaScript developer see Scala.js as a better way to implement frontend. To that end, I am sensing a shift in how JavaScript developers positively perceive many attributes that make Scala attractive in the first place. Since Scala.js is essentially Scala on JS, this may be a very timely development.

## Static typing is being embraced by JavaScript
As single-page apps gain prominence, there is a growing recognition that dynamic typing in JavaScript may not be the best approach to crafting large apps in a maintainable fashion. I am seeing this recognition in many places.

- React allows [expressing types for properties](https://facebook.github.io/react/docs/reusable-components.html). This feature is more like runtime assertions based on types. The guarantees offered by this feature is fairly weak and checking is late (during the actual execution in development mode), but it is a step in the direction that favors static typing.
* [Flow](http://flowtype.org) offers static analysis through annotated and inferred types to show type errors during build time. Type annotations you can provide aren't as expressive as in Scala, but it can save you from a few hard to debug type errors.
* Angular 2.0 is using [TypeScript instead of JavaScript](http://blogs.msdn.com/b/typescript/archive/2015/03/05/angular-2-0-built-on-typescript.aspx) so that code can include and framework can utilize types.
* [SoundScript](https://developers.google.com/v8/experiments) aims to improve upon ideas in Typescript and Flow  with a major focus on types.
* Even when not including types in the code itself, there is a tendency of being type aware at least in the documentation. For example, [Falcor](http://netflix.github.io/falcor/) documentation [describes types in great details](http://netflix.github.io/falcor/doc/Observable.html).

Even looking at languages in general, it is clear that the need for static typing is being felt: [Typed Clojure](http://typedclojure.org/), [Typed Lua](https://github.com/andremm/typedlua) etc. "Walks like a Duck, Quacks like a Duck, it must be a Duck" has changed to "If it is a Duck, just say so!".

All these point to a clear recognition that some form of static typing is helpful when developing complex apps. Scala's type system is far more expressive and battle-hardened, so it should allow leap-frogging other approaches. On a flip side: a few will look at Scala's type system as too expressive and/or too complex! And this isn't a facetious point. Static languages, Scala included, are still trying to find the right mix of ease of use and safety.

During our product development, we see benefits of static typing every day. For example, recently, we had to  change a tree-like entity by merging a few levels and adding a few other. With Scala.js compiler watching my back, I could refactor it quite easily. Basically, I changed the model as needed and then kept dealing with errors emitted by the compiler. By the time I got the whole app to compile, I also had the app working again. I remember a chill along my spine when I had to do similar tasks when using JavaScript alone.

Static typing also is helping us with confidence in what we have built. As it is testing is hard. Testing frontend is even more so. While no substitute for proper testing, static typing has been helping enormously to offer some assurance even before we test the app.

## Functional programming is gaining even more support in JS
Functional programming, in general, is gaining prominence; languages that have them are rising in popularity and languages that don't have are in the process of adding some support towards it. While JavaScript includes some support for functional programming, ES6 takes it a step further to bring in a simpler function syntax and add commonly expected collection operations such as `map` and `forEach` without needing an external library such as [underscore](http://underscorejs.org/) or [lodash](https://lodash.com/).

Scala's functional programming support is far more comprehensive. One area of particular interest in frontend development--async programming--becomes very natural using `Future`s and `Promise`s in Scala. The mental model of mapping and flatmapping over futures is way more natural than callbacks all over the code base. Here too, JS world is moving in the right direction with APIs such as [Fetch](https://fetch.spec.whatwg.org).

Scala is a bit uniquely positioned here in that it also combines object-orientation with functional programming. With ES6 including a proper support for classes, developers should get that familiar feel.

We have a lot of code that is best represented as data transformation and lends well to functional programming. Overall functional programming concepts have kept things clean, testable, and maintainable.

## Immutability is starting to be recognized as a good thing
A few years back, frontend development meant mutating objects and responding to the changes. AngularJS 1.0, for example, relied on observing mutations in data and DOM and keeping them in sync through a two-way binding mechanism. This mutability-oriented thinking led to the `Object.observe` proposal in ES7.

All this is changing. React is promoting immutability on all fronts. Angular 2.0 too is [promoting immutable objects](http://victorsavkin.com/post/110170125256/change-detection-in-angular-2). With efforts such as [immutable.js](http://facebook.github.io/immutable-js), it is easy for JavaScript developers to work with the immutable data structure. This reduces friction in adopting Scala by developers who are already favoring immutability. And the `Object.observe` ES7 proposal [may not happen after all](https://mail.mozilla.org/pipermail/es-discuss/2015-November/044684.html).

In our product, we use a modified [Flux pattern](https://facebook.github.io/flux/docs/overview.html) and limit mutability to a few stores (we are considering switching to [Diode](https://github.com/ochrons/diode)). This reduces the cognitive burden tremendously, since we know where to look when things change or don't change as expected. As an immediate benefit of using immutable model, we get undo/redo functionality at virtually no cost.

## JavaScript developers already use build tools

One of the benefits of using plain JS is the rapid change-save-reload model. But once you start to use any compile-to-js languages (including ES6), you have to add a build step using [gulp](http://gulpjs.com/), [webpack](https://webpack.github.io/) etc. Realistically, you need a build step even when you develop using plain JavaScript to deal with the asset pipeline etc. Scala being a compiled language obviously needs a build tool (for Scala.js, the most friction-free tool is [sbt](http://www.scala-sbt.org/)). Since you would need a build tool no matter which direction you go in, throwing in sbt to compile Scala code doesn't feel as a big a deal.

For us, thanks to the incremental build support in Scala.js, each change takes about 5 seconds to build (on a year old Macbook Pro). While not too bad, this is something where Scala.js could improve upon and there are a few ideas being considered such an incremental linker that should cut down this time to a smaller number.

## Scala.js already supports the isomorphism property
Isomorphism, in the context of webapps, implies having a choice of performing some work on server or client. For example, rendering the initial page on the server and then enhancing it in the client (with events handlers etc.) is a common way to improve the initial load time.

We utilize isomorphism to run the several algorithms on either server or client depending on the context. This allowed us to write a few fairly complex algorithms only once and have them run wherever they fit best--on the JVM backend or the JS frontend.

## Scala ecosystem is behind Scala.js
There seems palpable energy in Scala community around supporting Scala.js. Basically, the Scala community is starting to see JS as the second platform for Scala. A few libraries specifically target Scala.js  ([upickle](https://github.com/lihaoyi/upickle-pprint), [boopickle](https://github.com/ochrons/boopickle), [autowire](https://github.com/lihaoyi/autowire), etc.). Those that focused initially on the only platform available earlier--JVM--are changing to make them work on the JS platform ([Scalatest](http://www.scalatest.org/), [scalaz](https://github.com/japgolly/scalaz), [shapeless](http://milessabin.com/blog/2015/05/27/shapeless-2.2.0/), [monocle](https://github.com/japgolly/Monocle), etc.) where appropriate. New libraries see supporting Scala.js as a common requirement ([cats](https://github.com/typelevel/cats#scala-and-scala-js), [lenses](https://github.com/trueaccord/Lenses), [circe](https://github.com/travisbrown/circe), etc.). This energy around Scala.js helps to apply the same set of libraries and techniques that worked well on the server side to the client side.

# Is it the right time?
New programming languages, frameworks, tools, and approaches--however great--catch on only when the target audience is already ready for the goodies offered by them. For Scala.js, UI programming community seems primed to embrace many aspects available in Scala for quite some time now. All these factors make me feel that Scala.js could prove to be a formidable candidate when it comes to frontend development. Of course, there are many other factors that will play a role in deciding if Scala.js is going to actually succeed. It seems like a no-brainer for a developer already familiar with Scala to give a shot in trying Scala.js, but only time will tell if other developers will embrace Scala to use Scala.js.

As far as we are concerned, using Scala.js has been working wonderfully!
