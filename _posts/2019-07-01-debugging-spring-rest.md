---
title: Debugging Spring Rest Requests
---

We've been working with Spring Boot for a while now, and it gets the job done nicely. 
However, sometimes, somewhere between the model annotations, the default converters, the specific converters and the Swagger annotations, a rest request just fails, and you have no idea why. 

So to save yourself hours of fiddeling with JSON requests, take the 20 minutes and watch Baeldung's ["The Lifecycle of a Request"](https://courses.baeldung.com/courses/rest-with-spring-starter/lectures/425828). It will really make your life so much easier. 

And if you're not into the 20 minutes, try breakpointing these classes:functions - you'll find out exactly what went wrong in no time.

Thank me later :)

Your problem most likely happens at 
>```org.springframework.web.method.support.InvocableHandlerMethod:getMethodArgumentValues```

This is the function that is responsible of actually getting the actual Java object from the request, and in my experience, that's where you ususlly fail. If this works for you, you can start following the REST lifecycle to start narrowing down the options:

1. `org.springframework.web.servlet.DispatcherServlet:doDispatch`
1. `org.springframework.web.servlet.DispatcherServlet:getHandler`
2. `org.springframework.web.servlet.handler.AbstractHandlerMapping:getHandlerExecutionChain`
3. `org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodArgumentResolver:readWithMessageConverters`
4. `org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor:writeWithMessageConverters`

