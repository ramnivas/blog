---
layout: post
title: Making Eclipse snappier with KeepResident
date: 2004-12-17 09:09:02.000000000 -08:00
categories: Technology
tags: []
status: publish
type: post
published: true
color: navy
---
[KeepResident](http://suif.stanford.edu/pub/keepresident) is a very effective Eclipse plugin (for Windows) that eliminates mystery slowdowns often experienced by Eclipse users. It makes Eclipse snappy by encouraging the Windows virtual memory manager to keep the JVM process memory in RAM.

Most of my talks include several demos using Eclipse. To save time, I get Eclipse up and running with the right projects opened before going to my talks. However, when I bring up Eclipse from the minimized state, it takes about a minute before running at the full speed (and with so many people watching, a minute feels like an eternity!). Further, during the presentations, Eclipse often becomes irresponsive for a few seconds. With the KeepResident plugin, those delays are virtually gone.
