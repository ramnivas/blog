---
layout: post
title: An AOP success story from the real world
date: 2005-10-13 11:48:36.000000000 -07:00
categories:
- Technology
tags: []
status: publish
type: post
published: true
color: navy
---
Ken Pelletier sent me an email narrating his experience in applying [AspectJ](http://eclipse.org/aspectj) to fix security vulnerabilities in an existing application. This success story illustrates the power of AOP in the real world. Not only he secured his current application in very short period, but also ensured protection against new unsafe code.

> Hello Ramnivas,
>
> You may recall that you and I spoke at the NFJS conference again this year, and you were asking what were the barriers to introducing AspectJ into my current project.
>
> I have some good news to report.
>
> I was recently tasked with doing an assessment of our web tier architecture from a security point of view, with an eye toward shoring up any vulnerabilities in preparation for an upcoming security  audit. We will be rolling out an internet-facing web application which was previously intended to be only a proof-of- concept for internal use.
>
> As a consequence, there are several places where we render data to the browser without concern for whether it has been cleansed to protect against scripting exploits. I was able to very quickly exploit the site in several ways. Additionally, we are retrieving data from several partners via web services which is also subject to injection of code which, when rendered to a browser, can cause nasty things to happen.
>
> I determined the scope of work to plug the holes was non-trivial.  We would have to change all view-rendering code to escape all illegal markup characters prior to returning responses to the browser agent.   This would be a big undertaking, and, when it was done, there would be no protection against new code being introduced that was unsafe, except for a coding standard and a policy to do code reviews.
>
> I decided to look at the problem from an aspect point of view, and within about an hour I delivered an aspect to cover all cases.  It was remarkable how, with no change to existing code whatsoever, I was able escape all dangerous markup that was bound for the browser. I used an ant task to weave this into a jar when the webapp's war is assembled for deployment, so the whole thing is virtually transparent to the development process.
>
> I think this is the camel's nose under the tent.  The solution is so small, so airtight, and so unintrusive that I think it has the whole team rethinking their initial skepticism toward using AspectJ.  When measured against the 'traditional' solution, it wins on all counts.
>
> So, I thank you for your encouragement and guidance.  I've been to 4 of your sessions at NFJS conferences over the last 3 years and read your AspectJ book thoroughly.  I felt well prepared to make the case for AspectJ and to implement the solution in very short time.
>
> - Ken Pelletier
>
> Chicago, IL

Thank you Ken for sharing your experience.
