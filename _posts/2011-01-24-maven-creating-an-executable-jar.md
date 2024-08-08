---
title: 'Maven - Creating an executable jar'
author: yossale

 
categories:
  - Frameworks
  - Programming
tags:
  - jar
  - Maven
---
I wanted to create an executable jar in maven , s.t whenever I run"mvn install" , it will generate both the regular one and the executable one.

all the data here is taken from this [stackoverflow question][1] and this [maven example page][2].

In your pom.xml file , add this :

```xml
<build>
	<finalName>JarFilename</finalName>
	......
	<plugins>
		<plugin>
			<artifactId>maven-assembly-plugin</artifactId>
			<configuration>
				<archive>
					<manifest>
						<mainClass>fully.qualified.ClassName</mainClass>
					</manifest>
				</archive>
				<descriptorRefs>
					<descriptorRef>jar-with-dependencies</descriptorRef>
				</descriptorRefs>
			</configuration>
			<executions>
				<execution>
					<id>make-assembly</id>
                                        <phase>package</phase>
					<goals>
						<goal>single</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>

```

**Short explanation : **  
The artifactId is the plugin name (no packageId is required , because it's from the default maven package).  
In the configuration we define the location of the main class , and which plugin we're running  
And then , the execution : that tells the plugin when to run. In this case , it will run in the packaging phase. 

**How to run**  
All you need to do is run 

```shell
mvn install 
```

if you want only to create the executable jar , you can run 

```shell
mvn assembly:assembly 
```

 [1]: http://stackoverflow.com/questions/574594/how-can-i-create-an-executable-jar-with-dependencies-using-maven
 [2]: http://maven.apache.org/plugins/maven-assembly-plugin/usage.html