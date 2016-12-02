---
layout: post
title:  "Multicore Day Presentation"
date:   2016-12-02 19:17:13 +0200 
categories: work
comments: true
---

Tobias, my advisor, gave a presentation at
the
[SICS Software Week 2016's Multicore Day](https://www.sics.se/events/multicore-day-2016) in
Stockholm. What I like about our group is that the range of work we span is
quite wide &mdash; it ranges from the development of type systems down to the
nasty details of optimising garbage collection.

In the talk, he starts by briefly covering some of the more recent work I've done
on [Spencer]({{site.baseurl}}/spencer).

After that, he also talks about Kappa, a type system for preventing data races
in OO programs that my colleague [Elias Castegren]() has been working on
(published at
ECOOP'16 [preprint](http://www.it.uu.se/katalog/elica697/ECOOP2016.pdf) and
IWACO'16, [preprint](http://palez.github.io/IWACO2016/castegren-iwaco2016.pdf)).
Kappa is currently being integrated
into [Encore]({{site.baseurl}}/Encore-Glimpse), and I'm really excited about it.

Another thing Tobias mentions is work that our group,
mainly [Albert Yang](http://www.it.uu.se/katalog/albya111), is doing in
collaboration with [Sophia Drossopoulou](https://wp.doc.ic.ac.uk/sd/)
and [Juliana Franco](https://www.doc.ic.ac.uk/~jvicent1/) at Imperial College,
London that tries to exploit type information for giving objects more efficient
memory layouts at runtime, and to use the information a type system gives for
(much) more efficient garbage collection. Although this work hasn't been
published yet (and therefore, I will neither fully trust, nor quote performance
numbers), I believe that this line of work could be one of the necessary steps
towards the better future in which correctness and performance are aligned
goals.

<iframe width="560" height="315" src="https://www.youtube.com/embed/RnXXQCH8yUg" frameborder="0" allowfullscreen></iframe>
