---
layout: post
title: 'AOP and metadata: A perfect match, Part 1 published'
date: 2005-03-09 08:21:30.000000000 -08:00
categories:
- Technology
tags: []
status: publish
type: post
published: true
color: navy
---
IBM developerWorks has published the [first part of my two-part article series "AOP and metadata: A perfect match"](http://www.ibm.com/developerworks/java/library/j-aopwork3/). This article is the third in the year-long [AOP@Work](http://www.ibm.com/developerworks/views/java/libraryview.jsp?search_by=AOP@work:) series. In the first two articles in this series, [Mik Kersten](http://kerstens.org/mik) presented a thorough comparison of various AOP tools ([part1](http://www.ibm.com/developerworks/java/library/j-aopwork1), [part2](http://www.ibm.com/developerworks/java/library/j-aopwork2)). In my articles, I examine the concepts and mechanics in using metadata with AOP.

> The new Java metadata facility, a part of J2SE 5.0, is perhaps the most significant addition to the Java language to date. By providing a standard way to attach additional data to program elements, the metadata facility has the potential to simplify and improve many areas of application development, including configuration management, framework implementation, and code generation. The facility will also have a particularly significant impact on aspect-oriented programming, or AOP.
>
> The combination of metadata and AOP raises important questions, including:
>
> *   What is the impact of the metadata facility on AOP?
> *   Is metadata-fortified AOP optional, or is it necessary?
> *   What are the guidelines for using metadata effectively with AOP?
> *   How will this combination affect the adoption of AOP?
>
> I'll begin to answer these questions in this two-part article, the second in the new AOP@Work series. In this first half of the article I'll start with a conceptual overview of metadata and the new Java metadata facility. I'll also explain the difference between supplying metadata and consuming it, and provide some common programming scenarios that invite the use of metadata annotation. Next, I'll quickly review the basics of AOP's join point model and explain where it would benefit from metadata fortification. I'll conclude with a practical example, evolving a design through five stages using metadata-fortified AOP. In Part 2, I'll demonstrate a novel way to view metadata as a signature in a multidimensional concern space, talk about the impact of metadata on AOP adoption, and conclude with some guidelines for effectively combining AOP and metadata.
