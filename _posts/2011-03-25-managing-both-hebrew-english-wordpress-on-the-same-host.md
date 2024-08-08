---
title: 'Managing both Hebrew &#038; English wordpress on the same host'
author: yossale

 
categories:
  - Frameworks
---
I opened up a new blog about investments , and I wanted it to be in Hebrew. Problem is , WordPress is basically for ltr languages - and if you install the rtl version , you'll get everything backwards.

So I had no choice but to have 2 different WP installations on the same hosting service (I really didn't want to buy a new domain). The solution is fairly simple :

Install a new WP installation on a subdomain (www.yossale.com/finance , in my case) , and then do the following:

  1. Download the [Hebrew version][1] of WP (or your own localized version of WP)
  2. Unzip it
  3. Upload the wp-content/languages folder (with all it's content) from your computer to the same path on your host (usually www/YourSubDomain/wp-content)
  4. In your wp-config.php file : 
      1. Locate the line <pre class="brush:php">define ('WPLANG', );
```
        
        * *</li> 
        
          * Add your language identifier (&#8220;he-IL" , in my case) to the line - so you get* 
            
            <pre class="brush:php">define ('WPLANG', "he-IL");
```
            
            </em>* ** *** **</li> </ol> 
            
            ** **</li> 
            
            ** **
            
            ** **
            
            ** **
            
              * Refresh
            ** **** **** **</ol> 
            
            and you're done!
            
            You now have 2 different installations of WP - one in English , and the other in Hebrew
            
            ** **
            
            ** **
            
            ** **
            
            ** **

 [1]: http://he.wordpress.org/