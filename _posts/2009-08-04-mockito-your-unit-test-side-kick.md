---
title: "Mockito! Your unit test side-kick"

date: 2009-08-04 11:33:00 +0800
categories: [Programming]
tags: [Mockito, Java, Unit tests]

---

## Problem:

Your try to write a unit test to ComplexClass , but there's just too much objects that needs to be passed into it , and they all have their inner-logic.

## Solution:

[Mockito][1]. 

Basically , it something that creates an instance mock of every class you've ever had , AND it mocks its function calls! What does it mean? 

lets take our ComplexClass and test it. 

```java
public class ComplexClass {   
 ...
 public ComplexClass(SomeClassA a , SomeClassB b) {   
 ...
 }    
}
```

In the old days , you'd have to create an A or a mock A manually , set it's value , and make sure it returns whatever it's supposed to. This would be time consuming , error-prone and generally annoying. 

In comes Mockito:   
Want to mock A? no problem! 

`A a = Mockito.mock(A.class);`

That's it! you have mocked class! . Now I know what you're thinking - "What is it good for?"   
Well , here comes the pure magic - you want it to return a specific value upon a function call? NP!

`Mockito.when(a.getId()).thenReturn(1);`

Still not convinced?   
You can also make sure that specific functions have been called X amount of times ( `Mockito.verify(a.times(1)).getId();`) 

So that's Mockito in a nutshell , but I really recommend visiting their [site][1] and seeing what else they have to offer - It's one of those little apps that will change the way you work (in a good way!)

 [1]: http://mockito.org/
