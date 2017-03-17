---
layout: post
title:  "Tracing as a Service"
date:   2016-10-29 11:41:30 +0200 
categories: work
comments: true
permalink: spencer
---

<div style="width: 35%; display: block; margin: 0 auto;">
  <img src="http://stbr.me/assets/spencer_logo_bright.svg" alt="Spencer logo"/>
</div>

<br/>A project I've been working on is called Spencer: <a target="_blank"
href="http://spencer-t.racing">http://spencer-t.racing</a>. Spencer is a set of
tools with the aim to make it trivially easy for language designers, runtime
developers and other language technologists to understand the behaviour of
"typical" programs on the JVM.

Spencer contains a tool that works as a drop in replacement for `java` -- it
starts and runs an application, but recompiles code dynamically to log
information about variable loads/stores, field loads/stores, and method
enters/exits. These events are tagged with the current thread id, a sequence
number, types of callees, and so on.

We are hosting the data as a web service for people to interact with. Users are
able to query the web service by constructing, interactively, queries that
select objects. These queries can be gradually refined using an accessible
interface, and visualised in interactive plots.

Since data sets -- once they are uploaded -- never change, we can apply caching
to these queries, which will lead (eventually) to the ability to analyse
terabytes of data interactively. There are still some performance issues (mainly
of the UI) that need solving.

# Publications

There are two papers right now:

#### "Spencer: Interactive Heap Analysis for the Masses"

This paper is the suggested introductory paper. It covers the tool, its
architecture, and two simple use cases that illustrate the capabilities of the
system. To appear (International Conference on Mining Software Repositories
2017),
[preprint (pdf)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_msr.pdf?raw=true)

#### "Mining for Safety using Interactive Trace Analysis”

This paper is an application of Spencer; it mostly uses Spencer's API to look
for evidence for static properties that are hidden in Java programs. One example
for this is looking for classes of which all instances (or none) fulfilled a
certain property like uniqueness or immutability. For such classes, the
existence of a static property might hold -- we have generated a hypothesis. To
appear (Workshop on Quantitative Aspects of Programming Languages and Systems
2017)
[preprint (pdf)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_qapl.pdf?raw=true)

# Presentations

I presented the project at a meeting with some colleagues from Uppsala
University's computer architecture group:

 - [Slides (pdf)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_UART.pdf?raw=true)
 - [Slides (keynote)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_UART.key?raw=true)

Tobias gave a presentation that includes Spencer:

 - <iframe width="280" height="160" src="https://www.youtube.com/embed/RnXXQCH8yUg" frameborder="0" allowfullscreen></iframe>
 - [More context..]({{site.baseurl}}/work/2016/12/02/multicore-day.html)

I presented an high level overview at an
[UpScale](https://upscale.project.cwi.nl/) meeting in the fall of 2016:
can find the slides below.


 - [Slides (pdf)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_slides.pdf?raw=true)
 - [Slides (keynote)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_slides.key?raw=true)

