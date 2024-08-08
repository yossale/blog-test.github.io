---
title: 'Scala 101&ndash; Basic OOP : Writing a class'
author: yossale

 
categories:
  - Programming
  - Scala
tags:
  - Scala 101
---
This is part 3 of my Scala tutorial – read the [first part][1] and the <a href="http://www.yossale.com/?p=243" target="_blank">second part</a> for a more general Scala intro. All the examples you see here were ran via the REPL ( that’s the Scala interpreter).

## Scala’s take on OOP

Scala tends toward pure object oriented model: 

  ### There is no such thing as “primitive”.
  In Java , You have Objects and Primitives. Scala , on the other hand , takes after other language like Python, Ruby, Smalltalk (and many others) in the sense that Everything is an object. Including the integer 1 and string “Hello, World”. For many , this seems a reasonable evolution from the Object/Primitive dichotomy used in Java.
  ### Functions and Closures are also Objects. 
  This is something people with no background in functional programming sometimes find difficult to accept in the beginning.
 In Scala , a function is just another type of object , and as such it do anything an object can – Change , get sent as a parameter , be the return value of another function , and so forth. Even though the idea seems strange at first , you might recall that even in C/C++ you can pass a reference to a function , or use it as a return value from a function.
 
  ### Operators are methods – Like in C++

### Let’s get technical: 

Let’s build a simple class that will represent a fish. At first , a fish only has a name: 

```scala
scala> class Fish(var name: String) {}
defined class Fish
```

Well , something looks a bit… off… , isn’t it? There’s no constructor , no fields , no nothing! And yet , it works: 

```scala
scala> var jaws = new Fish("Jaws")
jaws: Fish = Fish@530f243b

scala> jaws.name
res2: String = Jaws

scala> jaws.name = "Rex"

scala> jaws.name
res3: String = Rex
```

So what happened here , exactly? What we’ve used here is the"Primary Constructor”. The variable we’ve passed is a property of the class , and you really don’t need any more setters and getters. 

What if I want an immutable field? Just use “val” instead of “var” :

```scala
scala> class Fish(val name: String) {}
defined class Fish

scala> var jaws = new Fish("Jaws")
jaws: Fish = Fish@2876b359

scala> jaws.name
res4: String = Jaws

scala> jaws.name = "Rex"
<console>:7: error: reassignment to val
jaws.name = "Rex"
^
```

And private fields?

```scala
scala> class Fish(private val name: String) {}
defined class Fish

scala> var jaws = new Fish("Jaws")
jaws: Fish = Fish@15664f1a

scala> jaws.name
<console>:8: error: value name cannot be accessed in Fish
jaws.name
^

scala> jaws.name = "Rex"
<console>:7: error: value name cannot be accessed in Fish
jaws.name = "Rex"
^
```

What about fields that aren’t in the constructor?

```scala
scala> class Fish(val name: String) {
val kind : String = "Shark"
}
defined class Fish

scala> val f = new Fish("Jaws")
f: Fish = Fish@69066caf

scala> f.kind
res12: String = Shark
```

Well, that’s great , but I want more then one constructor!

```scala
scala> class Fish(val name: String) {
def this() = this("SomeName")
}
defined class Fish

scala> var jaws = new Fish()
jaws: Fish = Fish@1dd0eb0b

scala> jaws.name
res6: String = SomeName
```

And something to tease you – How do you create a private primary constructor? 

```scala
scala> class Fish private (val name: String) {}
defined class Fish

scala> var jaws = new Fish("Jaws")
<console>:6: error: constructor Animal cannot be accessed in object $iw
var jaws = new Fish("Jaws")
^
```

 

<a href="http://programming-scala.labs.oreilly.com/ch05.html" target="_blank">Scala has a primary constructor and zero or more auxiliary constructors</a><em>. </em>The primary constructor is the entire body of the class.  So actually , every line written in the body of the class will be executed (not including those inside functions / methods , naturally).
 Note: In Scala , any auxiliary constructor must call another constructor of the same class as it’s first actions! 

Let’s say we want a fish to print its name when it goes up:

&nbsp;

```scala
scala> class Fish(var name: String) {
println(“I am “ + name)
}
defined class Fish

scala> var jaws = new Fish("Jaws")
I am Jaws
```

The code line “println (“I am “ + name)” , although it appears context-less , is actually part of the constructor. 

Let’s sum everything up: 

```scala
class Fish(var name: String, private var age : Int) {
   println (“A new fish is born!”);

   //This is accessible
   val kind : String = "Shark"

   //This is not accessible
   private val nickName = "Goldi"

   def this() = {
      //An auxiliary con'r MUST invoke another constructor as it's first action!
      this("Fishi", 0)
      println ("I'm an auxiliary constructor!")
   }

   def this(name: String) = this(name, 0);

   def swim() = println("Blo Blo")

   private def showUpperFin() = println("Dramatic music!")

   //If you override a function , you must declare it using the override keyword. Unless the
   //function is abstract , and then it's kind of obvious to the compiler
   override def toString = "My name is " + name

   //An operator is a method just like any other
   def + (that: Fish): Fish = return new Fish(this.name,this.age + that.age)
}
```

So what have we learned so far?

  * Scala has a Primary constructor (which is the entire body of the class) and auxiliary constructors - which serve just like Java constructors.
  * The default Scala scope is public
  * You can declare class parameters in the primary constructor
  * You can access and change non private class properties by accessing them directly

The next post will touch the getter/setters issue in Scala , limiting the scope of variables , using default values , and more . Stay tuned 😉

&nbsp;

 [1]: http://www.yossale.com/?p=227