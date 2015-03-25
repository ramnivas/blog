---
layout: post
title: AspectWerkz 2.0 RC1 is released
date: 2004-11-16 11:07:17.000000000 -08:00
categories: Technology
status: publish
published: true
color: navy
---
[AspectWerkz](http://aspectwerkz.codehaus.org) team has just [released the 2.0 version](http://aspectwerkz.codehaus.org/new_features_in_2_0.html). The new version contains many good features from the [AspectJ](http://eclipse.org/aspectj) users' perspective :

*   AspectWerkz is now an extensible aspect container. AspectJ excels at providing a language-level support for AOP concepts and AspectWerkz excels at its integration with application servers. It seems that one can now write aspects in AspectJ and deploy using AspectWerkz's deployment technology.
*   AspectWerkz's AOP support is now more closely aligned to that implemented in AspectJ:

    *   variations of the after() advice
    *   percflow() aspect association
    *   this(), target(), and args() pointcuts to capture join point context

The AspectWerkz team plans to address some of the remaining differences from AspectJ (cflow(), cflowbelow() etc.) in a future version. It seems like AspectJ and AspectWerkz will become _nearly_ isomorphic -- you can write program in AspectJ or AspectWerkz and translate between them without much loss of information.
<p>Of course, there are many AspectWerkz-specific improvements such as 20x improvement in performance and hot deployment/redeployment that got to make it more useful as an enterprise AOP solution.

This is exciting! Congratulation to the AspectWerkz team.
