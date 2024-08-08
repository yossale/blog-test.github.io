---
title: When will hadoop NN recognize a failure?

date: 2008-11-16 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
I've been trying to cause the NN to recognize that it is inconsistent and turn to the SNN for data.

<span style="font-weight: bold;">Scenario 1:</span> Will the NN Recognize that its data is not up-to-date? (No) 

  1. Started with NN on server A , SNN on B , DN on C
  2. Enter 3 files (testFileX.test)
  3. Wait for an SNN image to be written (usually happens after 5 minutes)
  4. kill NN, SNN , DN .
  5. Start the NN on B with -importCheckpoint , start SNN on B , start DN on C
  6. Enter 3 new files (failbackFileX.test)
  7. stop-dfs.sh
  8. Restart the original structure (NN on A , SNN on B , DN on C)
  9. what files does hadoop recognize? - Only the testFileX.test files (the first ones).

Result : Hadoop NN doesn't recognize that its data is not updated.

<span style="font-weight: bold;">Scenario 2:</span> Force it to load with importCheckpoint

Same scenario as above until 7 (include)  
8. hadoop namenode -importCheckpoint :  
&#8230; <span style="font-weight: bold;">NameNode already contains an image in /usr/apps/hadoop/name</span> &#8230;

That didn't work either.

<span style="font-weight: bold;">Scenario 3:</span> Corrupt the VERSION file:

Hadoop recognizes the version file is corrupted , but just fails - no access to the SNN. Doesn't work.

<span style="font-weight: bold;">Scenario 4:</span> Corrupt the fsimage file:

ERROR fs.FSNamesystem (FSNamesystem.java:<init>(277)) - FSNamesystem initialization failed.

No SNN involvment.

<span style="font-weight: bold;">Conclusion: </span>  
I have no idea when hadoop decides to turn to the SNN - it seems it should have done that in any of the above scenarios , but it won't.