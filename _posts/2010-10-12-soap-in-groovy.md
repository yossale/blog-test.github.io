---
title: Consuming SOAP in Groovy
author: yossale

 
categories:
  - Uncategorized
---
I've started using SOAP in Groovy. As I'm new both to SOAP and to Groovy , this is a little basic. 

Note: all of the following is checked for Windows 7, jdk 1.6.0_21. , groovyws 0.5.2, eclipse 3.6 (Helios), groovy 1.7.5

**What is SOAP? **  
You can find the [w3c tutorial here][1] , but you don't need me for that.  
Basically , it solves the problem of"How the hell should I know what your web service expects me to send him?!" 

It's done in the following way: when someone publishes a SOAP web service , you publish a url that looks like"http://&#8230;..wsdl" . (like this: http://soapclient.com/xml/soapresponder.wsdl). This url is actually an xml document , that has pre-defiend nodes , that tell you what are the names of the function , what they expect to get , and what they return. 

The cool thing is that most SOAP libraries know to do everything by themselves : You just give them the wsdl address , and they already create all the objects for you , and absolve you from the annoyance of parsing the xml and building objects you really couldn't care less about. 

**More hands on:**  
First of all , a very simple & basic soap service to use when you're writing your SOAP version"Hello, World" : <http://soapclient.com/xml/soapresponder.wsdl> (Don't see anything? You're probably using Chrome , aren't you? Chrome has an issue with XML)

It has one method , called Method1(String a, String b) and return the 2 strings.

<div class="codecolorer-container groovy default" style="overflow:auto;white-space:nowrap;width:650px;">
  <div class="groovy codecolorer">
    &nbsp; &nbsp; <a href="http://www.google.de/search?q=site%3Agroovy.codehaus.org/%20import"><span class="kw2">import</span></a> <span class="co2">groovyx.net.ws.WSClient</span><br /> <br /> &nbsp; &nbsp; <a href="http://www.google.de/search?q=site%3Agroovy.codehaus.org/%20def"><span class="kw2">def</span></a> proxy <span class="sy0">=</span> <a href="http://www.google.de/search?q=site%3Agroovy.codehaus.org/%20new"><span class="kw2">new</span></a> WSClient<span class="br0">&#40;</span><span class="st0">'http://soapclient.com/xml/soapresponder.wsdl'</span>, <a href="http://www.google.de/search?q=site%3Agroovy.codehaus.org/%20this"><span class="kw2">this</span></a>.<a href="http://www.google.de/search?q=site%3Agroovy.codehaus.org/%20class"><span class="kw2">class</span></a>.<span class="me1">classLoader</span><span class="br0">&#41;</span><br /> &nbsp; &nbsp; proxy.<span class="me1">initialize</span><span class="br0">&#40;</span><span class="br0">&#41;</span><span class="sy0">;</span><br /> &nbsp; &nbsp; <a href="http://www.google.de/search?q=site%3Agroovy.codehaus.org/%20println"><span class="kw8">println</span></a> proxy.<span class="me1">Method1</span><span class="br0">&#40;</span><span class="st0">"Hello,"</span>,<span class="st0">"World"</span><span class="br0">&#41;</span><span class="sy0">;</span>
  </div>
</div>

So far , everything looks nice. But for a lot of people trying to run this , it won't work. Why?  
Because of countless dependency issues. 

1. You need to get the groovyws-0.5.2.jar (or later) for the WSClient. (It won't compile otherwise)  
2. Now it compiles , but it throws nasty errors your way , ahh? You're probably getting this error :

<div class="codecolorer-container java default" style="overflow:auto;white-space:nowrap;width:650px;">
  <div class="java codecolorer">
    <span class="st0">"Exception in thread "</span>main<span class="st0">" java.lang.NoClassDefFoundError: org/apache/cxf/endpoint/Client</span>
  </div>
</div>

It's because what you have is only an API , and you need countless other dependencies.  
3. Hopefully , you're working on a maven project (aren't you? see next comment) . Add this dependency:

<div class="codecolorer-container xml default" style="overflow:auto;white-space:nowrap;width:650px;">
  <div class="xml codecolorer">
    <span class="sc3"><span class="re1"><dependency<span class="re2">></span></span></span><br /> &nbsp; &nbsp; <span class="sc3"><span class="re1"><groupId<span class="re2">></span></span></span>org.codehaus.groovy.modules<span class="sc3"><span class="re1"></groupId<span class="re2">></span></span></span><br /> &nbsp; &nbsp; <span class="sc3"><span class="re1"><artifactId<span class="re2">></span></span></span>groovyws<span class="sc3"><span class="re1"></artifactId<span class="re2">></span></span></span><br /> &nbsp; &nbsp; <span class="sc3"><span class="re1"><version<span class="re2">></span></span></span>0.5.2<span class="sc3"><span class="re1"></version<span class="re2">></span></span></span><br /> <span class="sc3"><span class="re1"></dependency<span class="re2">></span></span></span>
  </div>
</div>

4. Build your project a new . maven will take care off all the dependencies for you.  
5. Still doesn't work? you get [this ][2]nasty error? add this dependency to the pom:

<div class="codecolorer-container xml default" style="overflow:auto;white-space:nowrap;width:650px;">
  <div class="xml codecolorer">
    <span class="sc3"><span class="re1"><dependency<span class="re2">></span></span></span><br /> &nbsp; &nbsp; <span class="sc3"><span class="re1"><groupId<span class="re2">></span></span></span>xerces<span class="sc3"><span class="re1"></groupId<span class="re2">></span></span></span><br /> &nbsp; &nbsp; <span class="sc3"><span class="re1"><artifactId<span class="re2">></span></span></span>xercesImpl<span class="sc3"><span class="re1"></artifactId<span class="re2">></span></span></span><br /> &nbsp; &nbsp; <span class="sc3"><span class="re1"><version<span class="re2">></span></span></span>2.8.1<span class="sc3"><span class="re1"></version<span class="re2">></span></span></span><br /> <span class="sc3"><span class="re1"></dependency<span class="re2">></span></span></span>
  </div>
</div>

Now you can finally run!

**Don't use a maven project? **  
You have 2 options:  
1. Somewhere in the WWW there is a groovyws-standalone-0.5.2.jar , which includes all the  
dependencies. I , unfortunately , couldn't find it anywhere.  
2. Make a pom.xml as mentioned above , with the dependencies , and build it . All the  
dependencies will be in your repository , and you can use them from there.

 [1]: http://www.w3schools.com/soap/soap_intro.asp
 [2]: http://www.jroller.com/gmazza/entry/abstractmethoderror_on_org_apache_xerces