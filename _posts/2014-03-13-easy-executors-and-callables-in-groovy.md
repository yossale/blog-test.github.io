---
title: Easy Executors and Callables in groovy

published: true
---


Multithreading and general multi-tasking in groovy is super easy, thanks to frameworks like GPars. However, even the basic Java frameworks can be easily utilized in groovy for a rapid-no-brainer task-driven design.

Thanks to Groovy's closure coercion, running a set of callabels is as easy as this:


Basically what happened here is that every closure in groovy is executable, as we can see here:



So it's very easy to use a simple closure as a callable, and defer the actual execution of the command inside the closure to the Executor. Cool, ahh?
