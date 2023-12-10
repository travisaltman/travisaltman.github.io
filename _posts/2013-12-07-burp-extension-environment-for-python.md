---
title: "Burp extension environment for Python"
date: "2013-12-07"
---

This post will explain how to setup Burp so that you can use Python to write Burp extensions. Burp has an API that allows for [extensions](http://portswigger.net/burp/extender/) which add to the functionality of Burp. The Burp suite itself is written in Java so Burp natively supports Java extensions but through Jython you can now use Python scripts to build extensions. This comes in handy if you are more comfortable using Python day to day.


The first thing you'll need to do is download [Jython](http://www.jython.org/downloads.html), I downloaded the traditional installer which will end in a JAR extension. In order for the installer to work you'll need to have already installed the [Java runtime environment](http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html) (JRE). Now double click the JAR file to install.

![](/assets/12-2-2013-10-39-19-PM.png "12-2-2013 10-39-19 PM")

I chose the standard installation type.

![](/assets/12-2-2013-10-39-54-PM.png "12-2-2013 10-39-54 PM")

Next it should hopefully recognize where your JRE is installed.

![](/assets/12-2-2013-10-40-42-PM.png "12-2-2013 10-40-42 PM")

Hopefully you get to the last window during the install process that says congratulations.

![](/assets/12-2-2013-10-40-42-PM1.png "12-2-2013 10-40-42 PM")

Now that Jython is installed correctly we need to fire up Burp and configure it to use Jython for our Python scripts. Once in Burp go to the Extender tab then the Options. There you will see a section labeled "Python Environment", simply point to the location of your Jython JAR file. I accept the defaults during install and my location was C:\\jython2.5.4rc1\\jython.jar. See screen shot below.

![](/assets/12-2-2013-10-44-07-PM.png "12-2-2013 10-44-07 PM")

After this we are ready to load our first Python extension to Burp. Go back to the Burp [extension page](http://portswigger.net/burp/extender/) and download the HelloWorld zip fie which contains a Python example. Under the Extensions tab you can click "Add", choose the Python extension type and simply pick the HelloWorld.py example. After loading the extension you should see the window below.

![](/assets/12-7-2013-3-58-16-AM.png "12-7-2013 3-58-16 AM")

You'll also see some errors generated in the Errors tab.

![](/assets/12-7-2013-3-58-34-AM.png "12-7-2013 3-58-34 AM")

This is normal as the HelloWorld.py example is meant to show you what errors would look like as well. There you have it you have just loaded your first Python extension in Burp. Hopefully I will follow up with extensions I find useful and how they can help in performing application security assessments. Feel free to contact me or leave feedback.
