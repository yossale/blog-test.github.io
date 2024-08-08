---
title: 'Scala 101 - Scoping'
author: yossale

 
categories:
  - Programming
  - Scala
tags:
  - Scala 101
---
This is a part of my Scala tutorial . Read the <a title="Scala 101 – Val, var, Arrays, Loops and Conditions" href="http://www.yossale.com/?p=227" target="_blank">first part</a> & <a title="Scala 101 – REPL, Sequences, Tuples, Exceptions" href="http://www.yossale.com/?p=243" target="_blank">second part</a> for a more general Scala intro. You can read <a title="Scala 101– Basic OOP : Writing a class" href="http://www.yossale.com/?p=274" target="_blank">here</a> for an overview of how to write a class in Scala.

### Scoping

Like in most OO languages, you can limit the access to certain elements of the class (fields, methods) using access level modifiers - private , public , protected are the basic ones , but you can also have the package scope in Java , and the"friend" class in C++

Scala supports something much more agile then what we've come to expect  : As usual , you have private , protected and public (which is default in Scala , unlike in Java) - But , and this is where the fun begins , you also have a variable scope . Let's see an example , as usual , with our Fish class:

Let's say we have 2 fish : smallFish and bigFish. Currently , they can access each other's private field"myName" :

```scala
scala> var smallFish = new Fish("Small")
smallFish: Fish = Fish@5cb27de5
scala> var bigFish = new Fish("Big")
bigFish: Fish = Fish@78bdf2a

scala> bigFish.sayHello(smallFish)
hello, Small
scala> smallFish.sayHello(bigFish)
hello, Big
```

This is unavoidable in Java - The"private" modifier is valid for the class , not each object. It's like a small community - everyone who is like you (i.e object of the same class) can access everything you have.

Let's see how Scala solves this :

```scala
private[this] var myName = fishName
```

What we've done is added the"[this]" to the"private" access modifier - and that means that this variable is only usable from this object only. Let's see what happens when we try to compile the class with this small change:
```scala
class Fish (var fishName: String){
   private[this] var myName = fishName
   def name = myName
   def name_= (newName: String) = myName = newName
   def sayHello(otherFish : Fish) = println ("hello, " + otherFish.myName)
}
```

<console>:11: error: value myName is not a member of Fish
def sayHello(otherFish : Fish) = println ("hello, " + otherFish.myName)
```

So the class won't even compile - exactly like it won't compile if you try to access a private field from outside the class.

What else can you write in the [scope] ?  
<a title="Chapter 5. Basic Object-Oriented Programming in Scala" href="http://programming-scala.labs.oreilly.com/ch05.html" target="_blank">Here is what Oreilly has to say about it :</a>

> Visibility is limited to [scope], which can be a package, type, or this (meaning the same instance, when applied to members, or the enclosing package, when applied to types).

&nbsp;