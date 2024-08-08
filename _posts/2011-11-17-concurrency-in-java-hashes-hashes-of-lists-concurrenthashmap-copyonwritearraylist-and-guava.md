---
title: "Concurrency in Java &ndash; Hashes, hashes of lists, ConcurrentHashMap, CopyOnWriteArrayList and Guava"
author: yossale

Hide SexyBookmarks: 
  - 0
Hide OgTags: 
  - 0
wpsd_autopost: 
  - 1
categories: 
  - Uncategorized
published: true
---


&nbsp;

This post is mainly about using hash and hash-like structures in a multi threaded app. You might want to consider reading about Callable and other approaches too.

First thing you should know about concurrency – it’s not simple. It’s never simple, and although the built-in structures might help you, you’ll still need to make sure that your solution is robust.

My problem was this: I needed to have a hash of String –> List<String> , and update the list. Something like this:

```java
Map<String, List<String>> map = new HashMap<String, List<String>>();
for (Doc d : docs) {
  List<Doc> l = map.get(doc.getId());
  if (l == null) {
    l = new ArrayList<Doc>();
    map.put(doc.getId(),l);
  }
  l.add(d);
}
```

Naturally , this will only work in single threaded environment. But my environment is a multithreaded environment. So what do we do?

Well , the first, simplest solution is to just use  a <a href="http://download.oracle.com/javase/6/docs/api/java/util/Collections.html#synchronizedMap(java.util.Map)" target="_blank">SynchronizedMap</a>. This will work, but it will slow down your application immensely – It’s basically wrapping your map with a lock, and limiting access based on the lock. Which is a good enough solution if speed is not an issue, but otherwise it’s a bit crippling.

The second simplest solution is to use <a href="http://download.oracle.com/javase/1,5,0/docs/api/java/util/concurrent/ConcurrentHashMap.html" target="_blank">ConcurrentMap</a>. So we just replace our map with a ConcurrentHashMap.  
We’ll ConcurrentHashMap is basically a map with a lot of locks (default is 16). That way, the buckets in the map are distributed to super-bucket themselves , and each super-bucket is protected via a lock. That way, you get much more access to the hash then in the Synchronized case.

```java
Map<String, List<String>> map = new ConcurrentHashMap<String, List<String>>();
for (Doc d : docs) {
  List<Doc> l = map.get(doc.getId())
  if (l == null) {
    l = new ArrayList<Doc>()
    map.put(doc.getId(),l)
  }
  l.add(d)
}
```

However, we are still at risk of losing data due to a race condition: if we have 2 threaded that are trying to insert different document (d1, d2) with the same id to the hash in the same time , and the id is not yet there , then both of them might access the hash, see that there is value for that id, create a new, empty ArrayList, and then commit one right after the other – which means that we will lose one of the documents.

So, what do we do?

We’re going to use a very important function provided called “<a href="http://download.oracle.com/javase/6/docs/api/java/util/concurrent/ConcurrentMap.html#putIfAbsent(K, V)" target="_blank">putIfAbsent</a>”. It basically just does this :

```java
if (!map.containsKey(key))
       return map.put(key, value);
   else
       return map.get(key);
```

but does it atomically. So if we’d change our code to reflect that:

```java
Map<String, List<String>> map = new ConcurrentHashMap<String, List<String>>();
for (Doc d : docs) {

  List<Doc> l = new ArrayList<Doc>();
  List<Doc> cur = map.putIfAbsent(doc.getId(),l);

  if (cur == null) {
    l.add(d)
  } else {
    cur.add(d)
  }
}
```

What “putIfAbsent” returns is “what was in the list when I got there” . If it returns “null” , then the list we’ve provided , l, was placed in the hash a the value of our Id. Otherwise, there was already a list there, and we will have to use it (cur).

O.K, so that solves the problem of several threads overriding each other when creating a new list.

However, there’s a big chance that you’ll fail due to ConcurrentModificationException. Why is that?  
The reason for that is because we are constantly changing the values in the ArrayList - and if you're going to iterate over the list at that point or another while some threads are still working on it, you're most definitely going to have ConcurrentModificationException.

So, what to do?

Well, I guess there are multiple ways to take care of it, but in my case , since the number of Docs per id is limited to few dozens, I choose to work with the[ CopyOnWriteArrayList][1] . Basically what it does is this : every time you create an iterator of the list, the iterator has a"snapshot" of the list-state on the moment of it's creation. That way, you will never have a  ConcurrentModificationException. So the new code is :

```java
Map<String, CopyOnWriteArrayList<String>> map = new ConcurrentHashMap<String, CopyOnWriteArrayList<String>>();
for (Doc d : docs) {

    List<Doc> l = new CopyOnWriteArrayList<Doc>()
    List<Doc> cur = map.putIfAbsent(doc.getId(),l)

    if (cur == null) {
        l.add(d)
    } else {
        cur.add(d)
    }
}
```

Two last notes about concurrency in Java:

The first is that if you find yourself buried deep in concurrency issues, you might want to think about changing your strategy from synchronizing and concurrency to actors and callables.

The second is this - if you're already creating specialized structures for your application - start thinking about using [Guava][2] - google's open source concurrency package.


 [1]: http://download.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/CopyOnWriteArrayList.html
 [2]: http://code.google.com/p/guava-libraries/
