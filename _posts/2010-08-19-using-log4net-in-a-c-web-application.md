---
title: 'Using Log4Net in a C# web application'
author: yossale

 
categories:
  - Frameworks
  - 'Java 2 C#'
  - Programming
tags:
  - 'C#'
  - Log4Net
---
I came to Log4Net because I really loved Log4j , but I must say the documentation on this project is simply crappy, and half of the links in the project page are broken (conveniently , to all the examples&#8230;) .

After I've lost countless hairs on this should-have-been trivial thing , here's what I've got:

(Note: I've tried to add the configuration to web.config , and later to just another App.config file with it , but to no avail - Log4Net ignored it . If you managed , please tell me how!)

<div id="_mcePaste">
  <ol>
    <li>
      <strong>Create a configuration file (Lets call it My_Log4Net.config).<br /> </strong>This is a very basic configuration:</p> <div class="codecolorer-container html4strict default" style="overflow:auto;white-space:nowrap;width:650px;height:300px;">
        <div class="html4strict codecolorer">
          <span class="sc2"><?xml <span class="kw3">version</span><span class="sy0">=</span><span class="st0">"1.0"</span> encoding<span class="sy0">=</span><span class="st0">"utf-8"</span> ?></span><br /> <span class="sc2"><configuration></span><br /> &nbsp; <span class="sc2"><configSections></span><br /> &nbsp; &nbsp; <span class="sc2"><section <span class="kw3">name</span><span class="sy0">=</span><span class="st0">"log4net"</span> <span class="kw3">type</span><span class="sy0">=</span><span class="st0">"log4net.Config.Log4NetConfigurationSectionHandler, log4net"</span> <span class="sy0">/</span>></span><br /> &nbsp; <span class="sc2"><<span class="sy0">/</span>configSections></span> &nbsp;<br /> &nbsp; <span class="sc2"><log4net></span><br /> &nbsp; &nbsp; <span class="sc2"><root></span><br /> &nbsp; &nbsp; &nbsp; <span class="sc2"><level <span class="kw3">value</span><span class="sy0">=</span><span class="st0">"ALL"</span> <span class="sy0">/</span>></span><br /> &nbsp; &nbsp; &nbsp; <span class="sc2"><appender-ref ref<span class="sy0">=</span><span class="st0">"LogFileAppender"</span> <span class="sy0">/</span>></span> &nbsp; &nbsp; &nbsp;<br /> &nbsp; &nbsp; <span class="sc2"><<span class="sy0">/</span>root></span> &nbsp; &nbsp;<br /> <br /> &nbsp; &nbsp; <span class="sc2"><appender <span class="kw3">name</span><span class="sy0">=</span><span class="st0">"LogFileAppender"</span> <span class="kw3">type</span><span class="sy0">=</span><span class="st0">"log4net.Appender.FileAppender"</span>></span><br /> &nbsp; &nbsp; &nbsp; <span class="sc2"><<a href="http://december.com/html/4/element/param.html"><span class="kw2">param</span></a> <span class="kw3">name</span><span class="sy0">=</span><span class="st0">"File"</span> <span class="kw3">value</span><span class="sy0">=</span><span class="st0">"BCThresholdUpdater.log"</span> <span class="sy0">/</span>></span><br /> &nbsp; &nbsp; &nbsp; <span class="sc2"><<a href="http://december.com/html/4/element/param.html"><span class="kw2">param</span></a> <span class="kw3">name</span><span class="sy0">=</span><span class="st0">"AppendToFile"</span> <span class="kw3">value</span><span class="sy0">=</span><span class="st0">"true"</span> <span class="sy0">/</span>></span><br /> &nbsp; &nbsp; &nbsp; <span class="sc2"><layout <span class="kw3">type</span><span class="sy0">=</span><span class="st0">"log4net.Layout.PatternLayout"</span>></span><br /> &nbsp; &nbsp; &nbsp; &nbsp; <span class="sc2"><<a href="http://december.com/html/4/element/param.html"><span class="kw2">param</span></a> <span class="kw3">name</span><span class="sy0">=</span><span class="st0">"Header"</span> <span class="kw3">value</span><span class="sy0">=</span><span class="st0">""</span> <span class="sy0">/</span>></span><br /> &nbsp; &nbsp; &nbsp; &nbsp; <span class="sc2"><<a href="http://december.com/html/4/element/param.html"><span class="kw2">param</span></a> <span class="kw3">name</span><span class="sy0">=</span><span class="st0">"Footer"</span> <span class="kw3">value</span><span class="sy0">=</span><span class="st0">""</span> <span class="sy0">/</span>></span><br /> &nbsp; &nbsp; &nbsp; &nbsp; <span class="sc2"><<a href="http://december.com/html/4/element/param.html"><span class="kw2">param</span></a> <span class="kw3">name</span><span class="sy0">=</span><span class="st0">"ConversionPattern"</span> <span class="kw3">value</span><span class="sy0">=</span><span class="st0">"%d [%t] %-5p %m%n"</span> <span class="sy0">/</span>></span><br /> &nbsp; &nbsp; &nbsp; <span class="sc2"><<span class="sy0">/</span>layout></span><br /> &nbsp; &nbsp; <span class="sc2"><<span class="sy0">/</span>appender></span> &nbsp;<br /> &nbsp; <span class="sc2"><<span class="sy0">/</span>log4net></span> &nbsp;<br /> <span class="sc2"><<span class="sy0">/</span>configuration></span>
        </div>
      </div>
    </li>
    
    <li>
      <strong>Add the reference to the configuration file in the assembly file:</strong><br /> Add this line to the  AssemblyInfo.cs file:</p> <div class="codecolorer-container properties default" style="overflow:auto;white-space:nowrap;width:650px;">
        <div class="properties codecolorer">
          <span class="br0">&#91;</span>assembly: log4net.Config.XmlConfigurator<span class="br0">&#40;</span>ConfigFile <span class="sy0">=</span> <span class="st0">"My_Log4Net.config"</span>, Watch <span class="sy0">=</span><span class="re1"> true<span class="br0">&#41;</span><span class="br0">&#93;</span></span>
        </div>
      </div>
    </li>
    
    <li>
      <strong>All you have to do now is use the logger in your file . I usually use it like this:</strong> <div class="codecolorer-container csharp default" style="overflow:auto;white-space:nowrap;width:650px;">
        <div class="csharp codecolorer">
          <span class="kw1">private</span> <span class="kw1">static</span> <span class="kw1">readonly</span> ILog Logger <span class="sy0">=</span> LogManager<span class="sy0">.</span><span class="me1">GetLogger</span><span class="br0">&#40;</span><a href="http://www.google.com/search?q=typeof+msdn.microsoft.com"><span class="kw3">typeof</span></a><span class="br0">&#40;</span><span class="br0">&#123;</span>NAME_OF_CURRENT_CLASS<span class="br0">&#125;</span><span class="br0">&#41;</span><span class="br0">&#41;</span><span class="sy0">;</span>
        </div>
      </div>
    </li>
  </ol>
</div>

Log away <img src="http://ams18.siteground.eu/~yossale4/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" />