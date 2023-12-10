---
title: "Defeating MDM: Enrolling a jailbroken device into a mobile device management system"
date: "2015-04-30"
layout: post
---

[TLDR](http://en.wikipedia.org/wiki/Wikipedia:Too_long;_didn%27t_read):  I was able to enroll a jail broken device on a "major" MDM provider.  Any vendor that says they can prevent jailbroken devices from enrolling in a MDM solution is not being 100% honest.  Any resourceful person can get around jailbreak detections.  Because of the client side nature of this problem it's very difficult to control the end user, as always it's a cat and mouse game.


MDM or [mobile device management](http://en.wikipedia.org/wiki/Mobile_device_management) is a way for organizations to control, push configurations, set policies, monitor, etc on phones and tablets.  Mobile operating systems today offer some MDM capabilities but the reason why MDM solutions are popping out of the woodwork is that there are features lacking especially in iPhone / iOS and Android.  Google has recently come out with [Android for Work](https://www.google.com/work/android/) (AFW) which looks to close the gap and add a lot of MDM capabilities which will greatly help organizations adopt the Android platform, especially in a [BYOD](http://en.wikipedia.org/wiki/Bring_your_own_device) environment.  With that intro out of the way let's get to how one can break MDM.

This will focus on iOS.  Enrolling a jailbroken device can very from quite simple to reversing the MDM app on how it checks for jailbreak detection.  I got enrolled a jailbroken device via the simple method.  [Joe Demesy](https://www.linkedin.com/pub/joseph-demesy/74/816/440) and the guys at BishopFox did all the hard work and created an app, named [iSpy](https://github.com/BishopFox/iSpy), to make all of this easier.  Follow the instructions on how to get everything setup and once done you will be able to perform actions such as code injection, runtime modification, debugging, disable SSL certificate pinning, and bypassing jailbreak detection.  Once installed go to Settings > iSpy and there you can enable "Bypass Common JailBreak Checks", click on the screen shot below for the full size image.

![iSpy](/assets/iSpy-225x300.png)

To get around this "major" commercial MDM vendor jailbreak detection all I had to do was enable the feature inside of iSpy.  You will also have to enable "Inject iSpy into Apps" for the MDM mobile app.  Next open up the MDM app and you should briefly see in the top left of the app that iSpy is successfully injected into the app.  Below is an example of iSpy injected into OneNote.

![OneNote](/assets/OneNote-261x300.png)

After I did these two steps I was able to successfully enroll my jailbroken device inside the commercial MDM solution.  The purpose of MDM's blocking jailbroken devices is that they have full root access to the device which allows an attacker or malicious user to modify the intentional features of a MDM solution.  So allowing a jailbroken device is somewhat of a big deal as you don't have the same protections as you would with a non-jailbroken device.

Especially in iOS this cat and mouse game of jailbreak detection will continue.  Anytime you have a MDM iOS app the only thing it can do is what's allowed by the iOS framework which is somewhat limiting.  Also a jailbroken device can inject itself into the mobile MDM app and modify it to trick the MDM app that it isn't jailbroken.  So there you go, thanks to Joe Demesy and others it's that simple to get around a MDM solution.

For reference below are some links on techniques MDM vendors use to detect a jailbroken device which in turn attackers will try and get around.

[https://www.trustwave.com/Resources/SpiderLabs-Blog/Jailbreak-Detection-Methods/](https://www.trustwave.com/Resources/SpiderLabs-Blog/Jailbreak-Detection-Methods/)

[https://reverse.put.as/2013/06/30/gone-in-59-seconds-tips-and-tricks-to-bypass-appminders-jailbreak-detection/](https://reverse.put.as/2013/06/30/gone-in-59-seconds-tips-and-tricks-to-bypass-appminders-jailbreak-detection/)

[http://resources.infosecinstitute.com/ios-application-security-part-23-jailbreak-detection-evasion/](http://resources.infosecinstitute.com/ios-application-security-part-23-jailbreak-detection-evasion/)

[http://www.bishopfox.com/news/2014/09/misti-itac-2014-mobile-application-security-testing-code-review/](http://www.bishopfox.com/news/2014/09/misti-itac-2014-mobile-application-security-testing-code-review/)
