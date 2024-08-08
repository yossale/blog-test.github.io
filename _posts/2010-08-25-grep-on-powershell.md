---
title: Grep on PowerShell
author: yossale

 
categories:
  - Uncategorized
---
I must admit that I find the windows PowerShell a really-way-too-late pale substitution for the full capacity of the unix shell , but since I sometimes need to work on windows , there ain't much I can do about it.

So , grep :  
the equivalent of"grep" in PS is"Select-String" .  
The most basic usage is :

<div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:650px;">
  <div class="bash codecolorer">
    Select-String <span class="sy0"><</span>Some string<span class="sy0">></span> <span class="sy0"><</span>List of files<span class="sy0">></span>
  </div>
</div>

i.e

<div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:650px;">
  <div class="bash codecolorer">
    Select-String DocumentsCount <span class="sy0">*</span>.txt
  </div>
</div>

Almost all other grep functionality is supported ,and you can find more about it  <a href="http://www.computerperformance.co.uk/powershell/powershell_select_string.htm" target="_blank">here </a>