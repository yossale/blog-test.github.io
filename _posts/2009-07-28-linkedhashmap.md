---
title: LinkedHashMap

date: 2009-07-28 11:33:00 +0800
categories: [Programming]
tags: [Java]
---
**Problem:**

You need a data structure that should be limited in size , but use a special logic to decide which element to throw away when the limit is reached. 

**Solution**

Well , naturally , you can implement such a things , but the nice people in Java already thought about it : 

The [LinkedHashMap][1] is much more then a hash map with linked lists as buckets : it is has a full Cache-Like mechanism just to help you. 

**How does it work?**

Every time you call the put/putAll function in the LinkedHashMap , this function is called:

protected boolean **removeEldestEntry**([Map.Entry][2] eldest)

All this function does is to return whether or not the oldest element should be removed from the hash. So now , all you have to do is override this function in an extending class , for example : </p> 

> public class MaxSizedLRULinkedHashMap<K,V> extends LinkedHashMap<K, V> {   
> &#160;&#160;&#160;&#160; //Some constructors   
> &#160;&#160;&#160;&#160; //&#8230;&#160;   
> &#160;&#160;&#160;&#160;&#160; private final int MAX_SIZE;&#160;   
> &#160;&#160;&#160;   
> &#160;&#160;&#160;&#160;&#160; protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {   
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return this.size() > MAX_SIZE;&#160;   
> &#160;&#160;&#160;&#160;&#160;&#160; }   
> &#160; }</p> 

and that's it , you're done. Kind of. What does it mean , "Oldest" element is removed? 

Well , **LinkedHashMap** supports 2 types of "Aging" : insertion-ordered and access-ordered . This is defined in the constructor (the default is insertion-order) - so if you want to use it like an LRU , you'd have to pass to appropriate parameter in the constructor. 

<u>One last thing :</u>   
Instead of just returning a boolean in the **removeEldestEntry** function , you can implement your own clearing logic , and MUST return false - The effects of returning <tt>true</tt> after modifying the map from within this method are unspecified.

 [1]: http://java.sun.com/j2se/1.4.2/docs/api/java/util/LinkedHashMap.html
 [2]: http://java.sun.com/j2se/1.4.2/docs/api/java/util/Map.Entry.html