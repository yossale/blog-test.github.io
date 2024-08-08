---
title: Hadoop failover (and hopefully failback)

date: 2008-11-13 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
We've decided to test using linux HeartBeat together with hadoop , to enavle failover (and failback) capacbilities.

Infrastructure: Take 3 Servers : A is the NN , B is the SNN and will be later used as NN , and a datanode (On C). The hadoop-site.xml file in borh A and B use THE SAME LOCATION as their SNN.

3 servers:  
A - Hadoop NameNode - fs.checkpoint.dir: is configured to be on server B under fs.checkpoint.dir  
B - Hadoop SNN - fs.checkpoint.dir: is configured to be local , under fs.checkpoint.dir  
C - Hadoop DN

<span style="font-weight: bold;">Scenarion 1: Failover</span> 

  1. Run the regular (above) configuration.
  2. Insert some files
  3. Kill the NN on A.
  4. Stop the DN on C (this is only required becuase we don't use the Heartbeat yet).
  5. (create a /usr/apps/hadoop/name dir on B and updtae the hadoop-site files on B and C)
  6. Start the NN on B with the flag: haddop namenode -importCheckpoint
  7. Start DN on C.
  8. Check if all the relevent files exist.

<span style="font-weight: bold;">Status: Works</p> 

<p>
  </span><span style="font-weight: bold;">Scenarion 2: Failback</span><br />continue from the previous scenario: (NN and SNN on B , DN on C , Nothing on A) 
  
  <ol>
    <li>
      Insert some files
    </li>
    <li>
      Kill the NN on B.
    </li>
    <li>
      Stop the DN on C (this is only required becuase we don't use the Heartbeat yet).
    </li>
    <li>
      (updtae the hadoop-site files on B and C)
    </li>
    <li>
      Start the NN on A with the flag: haddop namenode -importCheckpoint
    </li>
    <li>
      Start DN on C.
    </li>
    <li>
      Check if all the relevent files exist.
    </li>
  </ol>
  
  <p>
    <span style="font-weight: bold;">Status: Fail (not back , just fail) </span> .<br />The NN on A doesn't failback .It claims it already has a valid image file in it's local location - but then detects errors and shuts down.
  </p>