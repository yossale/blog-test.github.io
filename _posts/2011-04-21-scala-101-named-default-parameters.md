---
title: 'Scala 101 - Named &#038; Default parameters'
author: yossale

 
categories:
  - Programming
  - Scala
tags:
  - Scala 101
---
Another syntax sugar provided in Scala are the default values for parameters , and the named parameters. Its fairly stright forwards , so We'll just start with an example :

```scala
def f(age: Int, name: String = "MyName") = println ("Hello, " + name + " , you're "
                                                           + age + " years old")
f: (age: Int,name: String) Unit

scala> f (1)
Hello, MyName , you're 1 years old
scala> f (1,"Jaws")
Hello, Jaws , you're 1 years old
```

So we just gave the parameter a default value , and if we don't provide a value , it just takes the default. That's very nice. But what will happen in the following case?

```scala
def f(name: String = "MyName", age: Int) = println ("Hello, " + name + " , you're "  + age + " years old")
f: (name: String,age: Int) Unit

scala> f(1)
<console>:7: error: not enough arguments for method f: (name: String,age: Int)Unit.
Unspecified value parameter age.       
f(1)     
```

Well, this doesn't work . Why? because the compiler can't tell which argument you meant. One solution is to put all the default parameters at the end of the function:

```scala
def f(age: Int, name:String = "MyName") = println ("Hello, " + name + " , you're " + age + " years old")
f: (age: Int,name: String)Unit

scala> f(1)
Hello, MyName , you're 1 years old

```

Another , and better, option is to just call them by their names:

```scala
def f(name: String = "MyName", age: Int) = println ("Hello, " + name + " , you're "  + age + " years old")
f: (name: String,age: Int)Unit
scala> f(age=1)
Hello, MyName , you're 1 years old  

```

I feel like this entire thing is a syntactic sugar , but it's still a nice one <img src="http://ams18.siteground.eu/~yossale4/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" />