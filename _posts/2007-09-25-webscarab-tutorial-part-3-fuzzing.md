---
title: "Webscarab Tutorial Part 3 (fuzzing)"
date: "2007-09-25"
layout: post
---

Part 2 covered the neat functionality of session ID analysis within Webscarab. Now we'll focus on another great function within Webscarab, fuzzing. I define fuzzing as testing the input of an application by trying various parameters that the input may not expect. These parameters don't have to be random, in my opinion it's best when you tailor your parameters depending on the application. When fuzzing you typically want to inject "command & control" parameters into the input to find the most serious vulnerability. For example if a web application is expecting a social security number I may inject html parameters such as " < / > " to manipulate the look, feel, and operation of a web application. I don't want to delve a whole lot into fuzzing because there are books out there that talk about this one subject. This tutorial is going to focus on using Webscarab to fuzz web applications and find vulnerabilities. Hopefully by the end of this tutorial you will better understand the technical aspects of fuzzing as oppose to the concept of fuzzing, but more reading on fuzzing web applications may be required.


This tutorial will once again be targeting Foundstone's Hacme Casino which intentionally has vulnerabilities built into the application. Fuzzing can focus on different types of vulnerabilities and parameters within web applications (e.g. XSS, SQL injection, queries, directory paths, etc...), although this tutorial will focus on parameters vulnerable to SQL injection. Foudstone's documentation lets us know that the username input is vulnerable to SQL injection so we can try fuzzing it with Webscarab to find other possible injections. First we'll try and login with the username 'test' and password 'test'. This can be seen in Figure 1.

[![Try logging into Hacme Casino](images/loginhacmecasinowithusernametest.png)](http://travisaltman.com/wp-content/loginhacmecasinowithusernametest.png "Try logging into Hacme Casino")

Figure 1: Trying to login

This will not log us into the application but Webscarab will capture the login process in the summary tab. Once this has happened find the login conversation in the summary tab. After you have found the login conversation simply right click and select "Use as fuzz template", this will send the parameters and headers associated with that request / conversation to the fuzzing tab. Selection of the login request can be seen in Figure 2.

[![Right click request to use as a fuzz template](images/rightclickuseasfuzztemplateforhacmecasinologin.png)](http://travisaltman.com/wp-content/rightclickuseasfuzztemplateforhacmecasinologin.png "Right click request to use as a fuzz template")

Figure 2: Send conversation to fuzz template

Now navigate to the Fuzzer tab within Webscarab. Here you'll see all the parameters that are associated with that request / conversation. You could add parameters to the request and see how the web application reacts to different paths, value, or types. You could also delete parameters for simplicity and to also see how the application reacts with those parameters missing. Once you have determined the parameters for fuzzing you'll need to define a fuzz source. So click on the "Sources" button beside "Start" and "Stop" within the Fuzzer tab. Here you will choose a dictionary style text file that contains parameters you want to fuzz with. I chose a SQL injection dictionary because we know the "username" field is vulnerable to SQL injections. The selection of the SQL injection dictionary can be seen in Figure 3.

[![Choosing SQL injection dictionary as a fuzz source](images/pickingsqlinjectionfuzzsources.png)](http://travisaltman.com/wp-content/pickingsqlinjectionfuzzsources.png "Choosing SQL injection dictionary as a fuzz source")

Figure 3: Choosing fuzz source

In my SQL injection dictionary I have 66 items, but Webscarab does not have a limit. There are lots of SQL injection dictionaries out there, some are even dedicated towards different platforms (e.g. MySQL, MS SQL Server, DB2, etc...). I got most of my SQL attacks from Andres Andreu's website [Neurofuzz](http://www.neurofuzz.com/), the dictionary I pulled from can be found [here](http://www.neurofuzz.com/modules/software/wsfuzzer/All_attack.txt). In this tutorial we won't be trying to fuzz for XSS vulnerabilities but [ha.ckers.org](http://ha.ckers.org/) has a now infamous [XSS dictionary](http://ha.ckers.org/xss.html) which is a great resource. Once all dictionary sources are added go to the main Fuzzer tab and assign parameters a fuzz source. This can be done via a drop down menu as seen in Figure 4.

[![Choosing fuzz source for each parameter](images/choosesqlinjectionfuzzsourcefromdropdownlist.png)](http://travisaltman.com/wp-content/choosesqlinjectionfuzzsourcefromdropdownlist.png "Choosing fuzz source for each parameter")

Figure 4: Drop down menu containing fuzz source

In order to prevent a parameter from being fuzzed simply leave the "Fuzz Source" field blank or delete the parameter altogether. In this case the "user\_login" is the only parameter that will reiterate through the SQLattack dictionary. The next step is to click on "Start" and let Webscarab try all of your parameters within the dictionary. This means the value "test" will be replaced with values inside the SQL injection attack dictionary and new request is sent to the web server for every attack parameter inside your dictionary. The fuzzer in action can be seen in Figure 5.

[![Running the fuzzer](images/runningfuzzerandwatchingrequests.png)](http://travisaltman.com/wp-content/runningfuzzerandwatchingrequests.png "Running the fuzzer")

Figure 5: Running the fuzzer

Notice the "Total Requests" and the "Current Request", once the fuzzer has run through all of the parameters in the SQL injection dictionary both of these numbers will be 68. Also notice the ID number 97 on the left hand side of the table, this is the first request of the fuzzing operation. The last request will have an ID number of 164, it's important to keep track of these request ID's when reviewing results of the fuzzing operation. I have found myself reviewing requests that weren't fuzzed and accidentally identified requests as not being vulnerable when in fact they were.

Once the fuzzer has made all of the requests a review of the results is needed to see if any of the attack parameters succeeded in a SQL injection. I do this simply by going back to the summary tab and opening up the first conversation of the fuzzing process. I then manually step through every conversation involved in the fuzzing operation and look for any "interesting differences" between responses. The phrase interesting differences is in quotation marks because fuzzing and looking for SQL injections is not an exact science but knowing how an application normally deals with the input will be helpful in determining what should and should not be expected in a HTTP response. Let's take a look at some of our fuzzing conversations to get a better idea of discovering differences. Have a look at the first fuzzing request, note the value of the "user\_login" parameter in the request and the value of the "Location" in the response. This can be seen in Figure 6.

[![First fuzzed conversation](images/nosqlinjectionfuzzparameterconversation97markedup.png)](http://travisaltman.com/wp-content/nosqlinjectionfuzzparameterconversation97markedup.png "First fuzzed conversation")

Figure 6: First fuzzed conversation

Here it's seen that the first value in the attack dictionary was actually used for the username value, good to know Webscarab is functioning properly. The top of Figure 6 shows that a POST request is sent to /account/login to check the credentials of the user, since the first SQL injection is not a valid user the response is to redirect back to the login screen. Keep in mind when looking at these conversation screen shots that the top half of the figure is the request and the lower half is the response. It can be deferred from this conversation that if an invalid username is inserted into the web application the response will be a redirect to the login screen. So when looking through the other SQL injected conversations it would be a good idea to look for a redirect to another location or an error message. It's always a good idea to look for database error messages when trying to find SQL injection vulnerabilities within a web application. Stepping through the other conversations I notice something different, this can be seen in Figure 7.

[![Successful SQL injection](images/sqlinjectionfuzzparameterconversation103withredtext.png)](http://travisaltman.com/wp-content/sqlinjectionfuzzparameterconversation103withredtext.png "Successful SQL injection")

Figure 7: SQL injection changed redirect location

Looks like on conversation 103 one of the SQL injections in the attack dictionary changed the location of the redirect to /lobby/games. Let's throw the injection value back into the web application and see what the response may be. The request can be seen in Figure 8.

![](https://github.com/travisaltman/travisaltman.github.io/blob/master/assets/aftersqlinjectionviawebinterface.png)

Figure 8: SQL injection request on Hacme Casino

The response to this SQL injection can be seen below in Figure 9.

[![Successful SQL injection](images/aftersqlinjectionviawebinterface.png)](http://travisaltman.com/wp-content/aftersqlinjectionviawebinterface.png "Successful SQL injection")

Figure 9: SQL injection response (Great success!)

Looks like the SQL injection gave us access to Andy Aces' account. This occurred because we added the phrase "or 1=1" (which is always true) to the end of the SQL query that authenticates the users to Hacme Casino. The reason Andy Aces' account was hijacked is because his name is the first one in the database. Guess having the last name Altman could be bad for me as well? Looking through the other conversations there appears to be another SQL injection that worked as well, this can be seen in Figure 10.

[![Another successful SQL injection](images/sqlinjectionfuzzparameterconversation117withredtext.png)](http://travisaltman.com/wp-content/sqlinjectionfuzzparameterconversation117withredtext.png "Another successful SQL injection")

Figure 10: Another successful SQL injection

The hits just keep on coming. This may seem to easy but there are plenty of web applications out in the wild that don't validate input and let malicious users manipulate their application and backend databases. SQL injections within a web application can be a serious vulnerability depending on the data held within the database. Had this scenario been real a malicious user could have taken over Andy Aces' account and had his way inside the online casino.

The fuzzing functionality of Webscarab makes web application vulnerability assessment a more automated process. Manually entering all of those SQL injection attacks can take a very long time. There is a downside to the dictionary approach though, your dictionary may not be as creative as a malicious user. Some people believe that a fuzzer should generate random input and that you should try thousands of requests in order to properly test a web application. Thousands of random requests could be better but stepping through those requests to determine validity can make for a long day. Although if one were to take the random input approach Webscarab has a solution for stepping through those results, the Compare and Search functionality. I may dive into the Compare and Search functionality at a later date, these functions can really speed up the process of web vulnerability assessment. Also keep your eye open for a video tutorial of Webscarab coming soon, you could always subscribe to my feed for the latest and greatest.

Once again I hope this tutorial was helpful in showing you the great features of Webscarab, as always your comments and feedback are welcomed.

travis@home:~$ more references

[Owasp Webscarab fuzzing tutorial](http://www.owasp.org/index.php/Fuzzing_with_WebScarab)

[Rogan Dawes documentation for Webscarab](http://dawes.za.net/rogan/webscarab/docs/)

[Foundstone Hacme Casino](http://www.foundstone.com/us/resources/proddesc/hacmecasino.htm)
