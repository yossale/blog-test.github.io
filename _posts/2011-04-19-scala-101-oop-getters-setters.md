---
title: 'Scala 101 - OOP : Getters &#038; Setters'
author: yossale

 
categories:
  - Programming
  - Scala
  - Uncategorized
tags:
  - Scala 101
---
This is a part of my Scala tutorial . Read the <a title="Scala 101 – Val, var, Arrays, Loops and Conditions" href="http://www.yossale.com/?p=227" target="_blank">first part</a> & <a title="Scala 101 – REPL, Sequences, Tuples, Exceptions" href="http://www.yossale.com/?p=243" target="_blank">second part</a> for a more general Scala intro. You can read <a title="Scala 101– Basic OOP : Writing a class" href="http://www.yossale.com/?p=274" target="_blank">here</a> for an overview of how to write a class in Scala.

Let's say we have the following class:

```scala
class Fish {
   var name = "Default Name"
}

scala> var jaws = new Fish
jaws: Fish = Fish@8191a42
scala> jaws.name
res29: java.lang.String = Default Name
scala> jaws.name = "Jaws"
scala> jaws.name
res30: java.lang.String = Jaws
```

Now , let's say you've decided you need to limit access to the name parameter vis a setter and a getter. How would those functions look like? Well , if you come from Java , like me , you would probably do something like this (after adding the"private" modifier to the variable):

And then you can use it like this:

```scala
scala> var jaws = new Fishjaws: Fish = Fish@1f619137
scala> jaws.getName
res32: java.lang.String = Default
scala> jaws.setName("Jaws")
scala> jaws.getName
res34: java.lang.String = Jaws
```

Problem Solved ? Well , this solution will work , but it's crappy. Why is it crappy?

  * You broke your api
  * If you want to avoid breaking the api , you have to have getters/setters from step one - which is a lot of unnecessary code , and it's really not very elegant.
  * It's much less convenient then the previous way.

So here comes Scala to the rescue: First , lets change the field name from"name" to"myName". Now , the Getter : lets create a function called"name" that will return the value of"myName". It fairly easy:

```scala
//Getter
def name = myName
```

You don't have to use"return" in Scala , and if it's a one-liner , you can drop the {}  , so we get a lovely little function that we can use like this :

```scala
scala> jaws.name
res30: java.lang.String = Jaws
```

Now for the setter. What we would like to do is keep the convenient field access - if we can get the name by using"jaws.name" , it would be great if we could have a function that will enable us to do this :

```scala
jaws.name = "Jaws"

```

  I bet you're thinking "Yeah , and grandma can fly" . Well, she can! You see , in Scala , you can have a space in the name of the function. Let me repeat that  <strong>a space in the function name</strong>. So actually , we can write a function whose name is: "name =" . Stop looking a me like this, it works <img src="http://ams18.siteground.eu/~yossale4/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" /> The magic little thing is , as always in Scala , _ . So this is our setter:


```scala
//Setter
def name_= (newName: String) = myName = newName

```

  The _ stands for a space , and now we can have this class :

  and we can do this:

  ```scala
scala> var jaws = new Fishjaws: Fish = Fish@8191a42
scala> jaws.name
res29: java.lang.String = Default Name
scala> jaws.name = "Jaws"
scala> jaws.name
res30: java.lang.String = Jaws
```

  I knew you'd like it <img src="http://ams18.siteground.eu/~yossale4/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" />

