---
title: "Maven: Building a Java-Groovy mixed project"
author: yossale

wpsd_autopost: 
  - 1
categories: 
  - Frameworks
  - Java
tags: 
  - Groovy
  - Maven
published: true
---

I have a project that mixes Java and Groovy, and the main problem I ran into was that the jar was nicely built, but it only contained the Java classes, and none of the groovy ones.

When I ran the groovy compilation plugin, I got only the Groovy classes, and none of the Java ones.

So, what you need to do to compile both of them and have a normally built jar it this:

In your pom file, after all the dependencies and properties, just add this build section:

```xml
<build>
  <sourceDirectory>src/main/groovy</sourceDirectory>
  <testSourceDirectory>src/test/groovy</testSourceDirectory>
  <resources>
    <resource>
      <directory>${project.basedir}/src/main/resources</directory>
    </resource>
  </resources>
  <plugins>
    <plugin>
      <groupId>org.codehaus.gmaven</groupId>
      <artifactId>gmaven-plugin</artifactId>
      <version>${gmaven-version}</version>
      <executions>
        <execution>
          <id>compile-groovy-classes</id>
          <goals>
            <goal>compile</goal>
          </goals>
          <phase>compile</phase>
          <configuration>
            <sources>
              <fileset>
                <directory>
                ${project.basedir}/src/main/groovy</directory>
                <includes>
                  <include>**/*.groovy</include>
                </includes>
              </fileset>
            </sources>
          </configuration>
        </execution>
        <execution>
          <id>compile-groovy-tests</id>
          <goals>
            <goal>testCompile</goal>
          </goals>
          <phase>test-compile</phase>
          <configuration>
            <sources>
              <fileset>
                <directory>
                ${project.basedir}/src/test/groovy</directory>
                <includes>
                  <include>**/*.groovy</include>
                </includes>
              </fileset>
            </sources>
          </configuration>
        </execution>
      </executions>
      <configuration>
        <providerSelection>1.7</providerSelection>
        <source />
      </configuration>
      <dependencies>
        <dependency>
          <groupId>org.codehaus.gmaven.runtime</groupId>
          <artifactId>gmaven-runtime-1.7</artifactId>
          <version>${gmaven-version}</version>
          <exclusions>
            <exclusion>
              <groupId>org.codehaus.groovy</groupId>
              <artifactId>groovy-all</artifactId>
            </exclusion>
          </exclusions>
        </dependency>
        <dependency>
          <groupId>org.codehaus.groovy</groupId>
          <artifactId>groovy-all</artifactId>
          <version>${groovy-version}</version>
        </dependency>
      </dependencies>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>2.8.1</version>
      <configuration>
        <useFile>true</useFile>
        <redirectTestOutputToFile>true</redirectTestOutputToFile>
        <reportFormat>plain</reportFormat>
        <failIfNoTests>true</failIfNoTests>
      </configuration>
    </plugin>
  </plugins>
</build>
```
