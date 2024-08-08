---
title: SVN Working Copy xxx locked and cleanup failed
author: yossale

 
categories:
  - Programming
  - Tiddy Bits
---
This thing drives me insane every time . It almost always happens in the middle of a mega-update, and usually the only solution is to delete everything and re-checkout. When it's 4G , it gets \*really\* annoying. 

Apparently , the main reason for this is that some"lock" files are left inside the".svn" folders , and the svn client refuses to clean them up (what's the point of the"cleanup" function , anyway , if it won't clean unnecessary locks?!) 

So I wrote this little Perl script to delete all the"lock" files from all the subdirectories in the project. It's quick , it's dirty as hell , and it will kill anything on its way - a perfect reflection of my mood!

You might also find the [StackOverflow question about it][1] useful (I sure did)

<div class="codecolorer-container perl default" style="overflow:auto;white-space:nowrap;width:650px;">
  <div class="perl codecolorer">
    <span class="kw2">use</span> strict<span class="sy0">;</span><br /> <span class="kw2">use</span> warnings<span class="sy0">;</span><br /> <span class="kw2">use</span> Cwd<span class="sy0">;</span><br /> <br /> <span class="kw2">use</span> File<span class="sy0">::</span><span class="me2">Find</span><span class="sy0">;</span><br /> <br /> <span class="kw1">my</span> <span class="br0">&#40;</span><span class="re0">$dir</span><span class="br0">&#41;</span> <span class="sy0">=</span> <span class="sy0">@</span><span class="kw2">ARGV</span><span class="sy0">;</span><br /> <br /> find<span class="br0">&#40;</span> <span class="re0">\&wanted</span><span class="sy0">,</span> <span class="re0">$dir</span> <span class="br0">&#41;</span><span class="sy0">;</span><br /> <br /> <span class="kw2">sub</span> wanted <span class="br0">&#123;</span><br /> &nbsp; &nbsp; <a href="http://perldoc.perl.org/functions/return.html"><span class="kw3">return</span></a> <span class="kw1">unless</span> <span class="sy0">-</span>f<span class="sy0">;</span> &nbsp; <br /> &nbsp; &nbsp; <a href="http://perldoc.perl.org/functions/return.html"><span class="kw3">return</span></a> <span class="kw1">unless</span> <span class="co2">/^lock$/</span><span class="sy0">;</span> <br /> &nbsp; &nbsp; <a href="http://perldoc.perl.org/functions/printf.html"><span class="kw3">printf</span></a> cwd<span class="br0">&#40;</span><span class="br0">&#41;</span><span class="sy0">.</span><span class="st0">"/$_<span class="es0">\n</span>"</span> <span class="sy0">;</span> &nbsp; &nbsp;<br /> &nbsp; &nbsp; <a href="http://perldoc.perl.org/functions/unlink.html"><span class="kw3">unlink</span></a> <span class="co5">$_</span> <span class="kw1">or</span> <a href="http://perldoc.perl.org/functions/warn.html"><span class="kw3">warn</span></a> <span class="st0">"deleting"</span><span class="sy0">.</span>cwd<span class="br0">&#40;</span><span class="br0">&#41;</span><span class="sy0">.</span><span class="st0">"/'$_ failed: $!<span class="es0">\n</span>"</span><span class="sy0">;</span><br /> &nbsp; &nbsp; <a href="http://perldoc.perl.org/functions/return.html"><span class="kw3">return</span></a><span class="sy0">;</span><br /> <span class="br0">&#125;</span>
  </div>
</div>

 [1]: http://stackoverflow.com/questions/127932/svn-working-copy-xxx-locked-and-cleanup-failed