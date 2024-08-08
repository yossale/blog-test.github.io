---
title: How to add another web-app to tomcat

date: 2009-04-01 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [tomcat]
---
&#160;

  1. Copy the .war file to /tomcat/wars folder (i.e /tomcat/wars) 
  2. In tomcat/apps create a new folder to where the war will be opened : **tomcat/apps/myapps   
    **
  3. In tomcat/conf/server.xml , add the reference to the new project :
<Host name="**myapp.mydomain.com**" appBase="**tomcat/apps/myapps**"   
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; unpackWARs="true" autoDeploy="true"> 

  4. Create a folder in tomcat/conf/Catalina/ called "myapp.mydomain.com" (this is how the tomcat will know who to relate it to) 
  5. In /conf/Catalina/myapp.mydomain.com add a context.xml file - this file will include all the context variable for your app. You also need to define in it the path to your app as the first object:   
    <Context path=/MyApp" reloadable="**true**">

Your war will be opened totomcat/apps/myapps/MyApp . Notice that letters case are crucial! 

Start tomcat.

To reach you app , use http://myapp.mydomain.com/MyApp/{your strut/filter path}