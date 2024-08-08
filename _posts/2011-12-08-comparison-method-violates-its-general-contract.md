---
title: "Comparison method violates its general contract!"
author: yossale

Hide SexyBookmarks: 
  - 0
Hide OgTags: 
  - 0
wpsd_autopost: 
  - 1
categories: 
  - Java
  - Programming
published: true
---

```java
java.lang.IllegalArgumentException: Comparison method violates its general contract!
```

Well, this is definitely the weirdest Java exception I've had so far. I got it on a Comparator I wrote, which returned either -1 or 1 (no == check was performed , and I never returned 0) .

This will happen to you usually when switching to Java 7 from older versions of Java. (Happened  to me when I switched from 6 to 7)

Long story short , the solution was to add the following line (Which is generally a good idea) to your Comparator :

```java
if (o1 == o2) return 0;
```

Why does it happens ? Well, according to [Java 7 change set][1], they replaced the merge sort implementation , and from now on

> The new sort implementation may throw an IllegalArgumentException if it detects a Comparable that violates the Comparable contract.


Why is that important to them? I have no idea, but I'm guessing maybe the new merge sort works better when some elements are equal (i.e , the compartor returns 0), and having a"bad" comprator hinders performances. .

What I do fail to understand is why this contract validity test done on runtime and not on compilation time!

 [1]: http://www.oracle.com/technetwork/java/javase/compatibility-417013.html
