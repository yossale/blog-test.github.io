---
title: 'Scala 101 - Val, var, Arrays, Loops and Conditions'
author: yossale

 
categories:
  - Programming
  - Scala
tags:
  - Scala 101
---
Today I've started a 3 days course (sponsored by my work place ) in Scala. The prerequisites  are just"Experienced in Java" .

you can find a lot of material on Scala across the web , and you don't need me for that - So I'll just give you the highlights and some links

  * **Compiles to JVM bytecode** (like groovy) and integrates with Java seamlessly (But also to compiles to CLR) - So you can start using it right now on your Java bio-sphere (You can call Java code from Scala and vice versa )
  * **Performance are ~= Java**
  * **Combines both OOP aspects and functional programming** - You can use it to do either one , but the real strength is achieved when you combines both approaches
  * **Statically Typed** - but very very strong typing mechanism
  * Supports DSLs
  * **Superb support for concurrency** (Using Actors)

Scala is much more flexible then Java , much less verbose (thank god for that) and much better suited for concurrency.

Download Scala from [here][1] , unzip it , and run the interpreter (scala.bat)

Lets start coding:

```scala
println "Hello, World"
```

Well , that was fairly easy.

O.K , so we got over the"Hello world" part. Let's see something more interesting :

```scala
val msg = "Hello, World" 
println(msg)
```

val means"this is a value" . A value , unlike a variable , can not be changed.  You'd also notice that there's no type to the msg value - Scala just inferred that it's String.

lets try to change the value :

```scala
msg = "Goodbye!"
error: reassignment to val      msg = "Goodbye!"
```

OK , so vals can not be changed. But we do need variables , mind you ..

```scala
var newMsg = "Hello, World"
newMsg = "GoodBye!"

println (newMsg)
>> Goodbye
```

O.K , so we have variables. What about functions?

```scala
def hello ( someone : String ) {
println("Hello, " + someone)
}

hello("World!")
>> Hello, World!
```

  * Declaring a function if done by"def" , like in groovy
  * Parameters are declared as paramName : ParamType
  * You don't have to declare a return type.

this would have worked too:

```scala
def hello ( someone : String ) : Unit = {
println("Hello, " + someone)
}

hello("World!")
>> Hello, World!
```

  *"Unit" is a fundamental type - slightly like Void in Java. So this function returns nothing.

this would have also worked :

```scala
def hello ( someone : String ) {
"Hello, " + someone
}

println hello("World!")
>> Hello, World!
```

  * You don't have to declare the function return type
  * you don't have to write"return" - the last computed value will be returned

this can also work :

```scala
def hello ( someone : String )  = { "Hello, " + someone }
```

and so will this :

```scala
def hello ( someone : String )  =  "Hello, " + someone
```

So , we get the fundamentals - what about Arrays and conditions?

```scala
val arr = Array(1,2)
def arrays(args: Array[Int]) = println (args(0)) 

arrays(arr)
>>1
```

  * You can initialize an array without declaring the type
  * you access the index of an array using () (not []!)
  * Arrays start at zero

And ifs?

```scala
println ( if (1 > 2) " what?! " else "that makes sense" )
>> that makes sense
```

  * If is a computed term (like the ?: operator in Java) - it returns a value

And a for loop?

```scala
val nums = Array(1,2)
for ( num <- nums ) println (num)
```

  * The basic use of"for" is more like"foreach" in Java then the standard for

This also works :

```scala
nums.foreach( num => println(num))
```

**  
Equality operators: **

== and != are defined for all objects. Unlike in Java  ,  == and != are Value comparisons! (not reference equality!). To check reference equality , use eq.

```scala
val l1 = List("A","B")
val l2 = List("A","B")

l1 == List("A", "B")
>> true

l1 == l2
>> true

l1.eq(l2 )
>> false
```

&nbsp;

**Other side notes**

  * You can define functions that are called"+" ,"-" and so forth. This is NOT operator overloading - Scala-wise , these are just functions like any other function. Each function that gets only 1 parameter can be called like this :  
    Console println"Something"  - which for Scala is the same as saying " a + b" .
  * Every object has a function called apply(Param) that can be called without the"apply" . So actually , when we used the array like this : arr(1) , we actually called the function arr.apply(1)

&nbsp;

 [1]: http://www.scala-lang.org/downloads