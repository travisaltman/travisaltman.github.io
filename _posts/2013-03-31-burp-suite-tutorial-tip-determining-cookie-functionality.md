---
title: "Burp suite tutorial / tip: determining cookie functionality"
date: "2013-03-31"
categories: 
  - "web-security"
---

When testing web applications you may come across an application that passes a ton of cookies along with each request. Cookies are used to maintain state within the application and typically only one cookie is needed within the application. There are times when other cookies are used as well and when testing web applications it may be difficult to determine what cookie is associated with session and functionality. Hopefully my technique of determining cookie functionality will also help others as well. Let's get started with an example. I'm going to take a look at ubuntu forums as an example.


![](/assets/UbuntuForumHomePage.png "UbuntuForumHomePage")

So configure burp to capture traffic and make a request to Ubuntu forums. Below is the screen shot of Burp making the first couple of requests to ubuntuforums.org.

![](/assets/BurpUbuntuFirstRequest2.png "BurpUbuntuFirstRequest2")

Here we see that simply going to the forum home page without authenticating we get two cookies "bb\_lastvisit" and bb\_lastactivity". We're lucky in some sense as these cookies are fairly descriptive, often times cookies have nondescript names which makes it even more difficult to understand their functionality. Now let's authenticate and see what other cookies come into play.

![](/assets/AuthUbuntuForum.png "AuthUbuntuForum")

Now we have two additional cookies (bb\_sessionhash and IDstack) that get submitted with each request. At this point it's a safe bet to say that either IDstack or bb\_sessionhash is responsible for handling session to ubuntu forums. One of the quick and easy ways to determine which cookie is truly used for session is to intercept a request that requires authentication, manually delete that cookie and see if you get kicked out of the application. Below is a screenshot of me performing that action.

![](/assets/BurpIntProfileUbuntuForum.png "BurpIntProfileUbuntuForum")

In this example I clicked on my profile because some portions of the profile require authentication to view. So I deleted the IDstack cookie to see if it had any affect on session. After forwarding the request it did indeed bring me to my profile so the IDstack cookie isn't responsible for handing the session. We can see this in the browser as shown in the screenshot below.

![](/assets/UbuntuForumProfileBeforeDelCookie.png "UbuntuForumProfileBeforeDelCookie")

Next let's try the getting rid of the bb\_sessionhash cookie via the same method.

![](/assets/BurpRemBBsessionCookie.png "BurpRemBBsessionCookie")

After the bb\_sessionhash cookie is removed we do indeed loose the authenticated features of the "My Profile" page as seen below.

![](/assets/UbuntuForumProfileAfterDeletingCookie.png "UbuntuForumProfileAfterDeletingCookie")

So now we have identified the cookie that maintains session for this application. It's also a good idea to delete all the cookies except for bb\_sessionhash or your particular cookie in question. I did go ahead and delete all the cookies except for bb\_sessionhash and I maintained session so the other cookies have nothing to do with session in this instance. In this scenario you can wipe your hands clean knowing that you've correctly identified the cookie that properly handles the session but sometimes other cookies will appear within the same application when you stumble across other functionality. One example of this is when a web application has some reporting piece to it. I've seen where the reporting functionality may be a bolted on third party application that uses it's on session handling which would call another cookie. Because of this I like to only pass the cookie that handled the initial authentication and see if any other functionality, a la reporting, gets broken with only the one cookie being passed. So what I like to do is set up a intercept rule that will only pass the initial authentication cookie and monitor what gets broken as I walk the application. To setup this rule go into the Options tab within the Proxy tab and create a "Match and Replace" rule to perform this. I add a rule with type of "Request header" and then paste the entire cookie line into the "Match" field then I only place the session handling cookie in the "Replace" field which in this case is the bb\_sessionhash cookie. A screenshot of this rule can be seen below.

![](/assets/MatchAndReplaceCookie-300x145.png "MatchAndReplaceCookie")

After this every single request will only contain the cookie you've identified as the one belonging to the main session. If you look in the history tab and view a request after you've made the rule you'll see where this comes into play as seen in the screenshot below.

![](/assets/AutoModMatchReplaceRequestForCookie.png "AutoModMatchReplaceRequestForCookie")

Now you've successfully isolated one cookie that deals with initial authentication and while you're walking the app and you stumble across some broken functionality it may be because another cookie got introduced into the application. Hopefully this rule will help you identify other cookies that get introduced into the application. Also hopefully this will keep you mind open to other rules that you can make regarding sessions within the application.
