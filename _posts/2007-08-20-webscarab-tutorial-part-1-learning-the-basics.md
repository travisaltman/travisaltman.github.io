---
title: "Webscarab Tutorial Part 1 (learning the basics)"
date: "2007-08-20"
layout: post
---

This tutorial is designed to walk you through the basics of using a HTTP proxy. A HTTP proxy is very useful when it comes to web application vulnerability assessment. A proxy will allow you to record all of your transactions while using the web application producing a history of pages you have visited and links you have clicked. A proxy also allows you to see the HTTP request and responses, basically you'll see what is being sent behind the scenes. This document will go into more detail about what a HTTP proxy can do as we step through some exercises on analyzing traffic from a web application.


This tutorial is going to focus on Webscarab, although there are other numerous useful tools on the market (e.g. Paros, Burp). The first thing we'll need to do is obtain Webscarab, I like to use the version signed by Rogan Dawes, which can be found [here](http://www.owasp.org/index.php/Category:OWASP_WebScarab_Project). Go to the downloads section and make sure you get the Java Web Start version signed by Rogan Dawes. The second thing we'll need to do is start up Webscarab. By default Webscarab listens on port 8008 but this can be easily changed to any port. These settings can be seen in Figure 1.

![](/assets/webscarabproxysettings.JPG)

Figure 1: Webscarab proxy settings

We'll also need to configure our browser so that our communication is pointed through the proxy. In recent versions of Firefox the path should be Tools >> Options >> Advanced Tab >> Network Tab >> Settings. Once there you'll need to highlight "Manual proxy configuration", then for "HTTP Proxy" type in "localhost" and for port use 8008. You'll also need to do this for the SSL proxy if the web application uses SSL. These settings can be seen in Figure 2.

![](/assets/firefoxproxysettings.JPG)

Figure 2: Firefox proxy settings

The path to change IE settings: Tools >> Internet Options >> Connections tab >> LAN settings. Here you'll need to check the box that says "Use a proxy server for your LAN", this can be seen in Figure 3.

![](/assets/ieproxysettings.JPG)
Figure 3: IE proxy settings

This tutorial is going to show how Webscarab can walk through and assess the Hacme Casino web application provided by Foundstone, Figure 4 shows the login page for this application.

![](/assets/hacmecasinologinpage.PNG)

Figure 4: Hacme Casino login page

I have already created an account within the application with the username "hacker" and a password of "passwd". So with Webscarab already running in the background I am going to login to Hacme Casino. If you are on the summary tab within Webscarab you will notice requests and responses filling up rows in the bottom pane. Webscarab is logging all communication between you and the web server, this includes all images, CSS files, Javascript files, parameters, etc... The top pane of the summary tab shows you a directory structure of your history through the web application. This summary tab can be seen in Figure 5.

![](/assets/summaryhacmeloginviawebscarab.PNG)

Figure 5: Webscarab summary tab

Now a summary of your history is neat but that only scratches the surface of Webscarab's functionality. One of the best functions of a HTTP proxy is the ability to intercept requests on the fly or replay those requests at a later time. In order to intercept requests / responses make sure you have checked the "Intercept requests" / "Intercept responses" checkboxes in the Proxy >> Manual Edit tab. These settings can be seen in Figure 6.

![](/assets/webscarabinterceptsettings.JPG)

Figure 6: Webscarab intercept settings

You may be wondering why you would want to intercept or repeat a HTTP request / response. The simple answer is to learn more about what a website is doing with your input (e.g. SSN, credit card, personal information). Application security folks, developers, or curious people may want to understand more about the web application they're using. Intercepting a request / response will allow you to see and manipulate communication being sent back and forth. Application security analysts like to replay requests over and over again with different inputs to see what the application will allow as input. This will give security analysts an idea of how secure the application is. Had we intercepted the login process you would have seen the inputs for username and password being sent to the web server. A screen shot of this can be seen in Figure 7.

![](/assets/intercepthacmelogincredentials.PNG)

Figure 7: Interception of the login process for Hacme Casino

You can see in Figure 7 that Webscarab has intercepted both the username "hacker" and password "passwd". A HTTP proxy is able to see the password even though each character was replaced by an asterisk within the application. At this point you could accept the request or manipulate the parameters. You could try to login as someone at this point even though you initially typed in a different username and password. With a HTTP proxy you could manipulate any request / response not just the login process.

This covers Part 1 of the tutorial on Webscarab. OWASP also has a great write up, called [Getting Started](http://www.owasp.org/index.php/WebScarab_Getting_Started), going over basically what I have covered here. So if you ever wanted to know more about a web application Webscarab is a great tool that can help you learn more. In Part 2 of this series we'll analyze how an application maintains state by using the "SessionID Analysis" functionality of Webscarab.
