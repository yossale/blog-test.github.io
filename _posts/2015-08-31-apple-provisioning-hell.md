---
title: Apple Provisioning Hell

---

We're developing an ionic app for iOS and android, and handling the Apple provisioning/certificates/apn profile is hell. Just pure hell. 

One of them most annoying messages is "Your account already has a valid iOS distribution certificate". If my account already has a valid iOS dist. cert, why won't you just get it and build the freaking thing? 

After wasting craploads of time on the nonsese, I finally found [Sigh](https://github.com/KrauseFx/sigh). They had me at the first sentence:
> Because you would rather spend your time building stuff than fighting provisioning

Just install it:
`sudo gem install sigh`

And then run it:
`sigh`

And that's it. It will get, fix, download and install all the provisioning profiles, and you'll save yourself hours of tweaking around with apple's weird licensing logic. 

Sigh is also a part of an open-source toolbox called fastlane, to easily facilitate iOS development, testing, installation and deployment. 


