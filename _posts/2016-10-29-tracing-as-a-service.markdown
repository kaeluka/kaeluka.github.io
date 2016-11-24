---
layout: post
title:  "Spencer: Tracing as a Service"
date:   2016-10-29 11:41:30 +0200 
categories: work
permalink: spencer
---

<div style="width: 50%; display: block; margin: 0 auto;">
  <img src="https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_screenshot.png?raw=true" alt="screenshot"/>
</div>

<br/>A project I've been working on is called Spencer. Spencer is a set of tools
with the aim to make it trivially easy for language designers, runtime
developers and other language technologists to understand the behaviour of
"typical" programs on the JVM.

Spencer contains a tool that works as a drop in replacement for `java` -- it
starts and runs an application, but recompiles code dynamically to log
information about variable loads/stores, field loads/stores, and method
enters/exits. These events are tagged with the current thread id, a sequence
number, types of callees, and so on.

We are going to host these data as a web service for people to interact with.
Users will be able to query the web service by constructing, interactively,
queries that select objects. These queries can be gradually refined using an
accessible interface, and visualised in interactive diagrams.

Since data sets -- once they are uploaded -- never change, we can apply caching
to these queries, which will lead to the ability to analyse terabytes of data
interactively.

I presented an high level overview at an
[UpScale](https://upscale.project.cwi.nl/) meeting in Oslo a few weeks back. You
can find the slides below.

# Download

 - [Slides (pdf)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_slides.pdf?raw=true)
 - [Slides (keynote)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_slides.key?raw=true)
