---
title: "Python script to check for vulnerable printers"
date: "2010-06-17"
layout: post
---

People often overlook printers when it comes to information security. Truth is that a ton of useful information can be found in printers. Employees will often scan sensitive documents such as social security cards, loan information, birth certificates, etc.


I've also seen important organizational information on printers such as internal memos between higher up executives. The documents I've seen in the past were never meant to be shared but a default printer will more than happily share your sensitive information. Almost any new commercial printer will come with a ton of features to store and retrieve any documentation that flows through the printer (copy, scan, and print jobs). Almost all of these new printers also give you a web interface to retrieve that documentation, an example of a [printer's web interface can be seen here](http://www.buyastrostuff.com/ftp/Rays/5100/Web-Interface.jpg). When I'm performing a [penetration test](http://en.wikipedia.org/wiki/Penetration_test) I always go for the web interface of a printer, the web interface is where I can grab all the sensitive information. These printers usually get unboxed and plugged into the network without much configuration from the default state, this means that the web interface is wide open with default usernames and passwords. Usually admin access to these printers will give you more access and it's this admin access that I check for.

When you've only got a limited amount of time during a penetration test you want to get the best bang for your buck so I created a python script that will go and check for default usernames and passwords on certain models of printers. Below is the python script.

```python
import urllib2
import sys

target = open(sys.argv[1])
eachIPinList = target.readlines(); target.close()
output = open(sys.argv[2], 'w')

for string in eachIPinList:
  try:
    print 'Trying ' + string.rstrip()

    theurl = 'http://' + string.rstrip() + '/index.html'
    username = 'root'
    password = ''

    passman = urllib2.HTTPPasswordMgrWithDefaultRealm()
    passman.add_password(None, theurl, username, password)
    authhandler =  urllib2.HTTPBasicAuthHandler(passman)
    opener = urllib2.build_opener(authhandler)
    urllib2.install_opener(opener)
    pagehandle =  urllib2.urlopen(theurl)
    if pagehandle.getcode() == 200:
      output.write(string)
  except:
    pass
```

Usage:  at the command line type the following

```python
python nameOfScript.py IPlist.txt output.txt
```

So this script takes two arguments, 1) A list of IP's you'll want to test against, 2) Name of an output file where successful attempts are logged. If you're having troubles running the script read my [other post about running a python script](http://travisaltman.com/password-dictionary-generator/). The output.txt will contain a list of IP's that the script was able to log into. There are three variables that you'll have to modify for your particular printer model that you are trying to scan for on your network, they are listed below.

```python
theurl = 'http://' + string.rstrip()  + '/index.html'
username = 'root'
password = ''
```

Username and password variables should be obvious, simply put in the default username and password of the printer on your network. The only thing you'll have to change in 'theurl' variable is the last quoted string. In my case it was '/index.html', in your case it could be '/auth/login.html'. Variable 'theurl' builds the http request used to log into your printer's web interface. A full example is below.

```
http://192.168.1.5/index.html
```

This script is doing nothing more than trying to log into the web interface of a printer, that's it. So the script is not limited to printers, it can be used against any web application that takes a username and password. Although this script can be used against any web application there is a limitation.  This script authenticates to the printer using Basic Access Authentication. There are three main ways to authenticate to a web application.

1. HTTP Basic Access Authentication
2. HTTP Digest Access Authentication
3. HTML Form-based Authentication

So this script will not work if your web application (printer in this case) is using the second or third option. How would you know which one your printer or web application is using? Turns out OWASP has a nice write up on [how to test which type of authentication](http://www.owasp.org/index.php/Testing_for_Brute_Force_%28OWASP-AT-004%29#Black_Box_testing_and_example) your web application is using. Turns out that no one really uses one and two because they are not as secure as HTML Form-based Authentication wrapped inside SSL. Of course some printers use Basic Authentication because they are poorly built. Basic Authentication actually passes your username and password essentially in [plaintext](http://en.wikipedia.org/wiki/Plaintext), the only way it tries to hide your username and password is by [base64](http://en.wikipedia.org/wiki/Base64) encoding them which is easily transformed back into plaintext. I don't want to get lost in the weeds to much but just knowing that your printer is using Basic Authentication is bad enough. Even if you set a strong username and password anyone [sniffing network traffic](http://en.wikipedia.org/wiki/Packet_analyzer) would be able to determine your credentials.

I kicked this script over to [Dave Huggins](http://davehuggins.com/blog/) who has tons of experience developing Python applications and he quickly improved upon it by adding the functionality of IP ranges instead of a file. His enhancements can be seen below.

```python
def IPRange(octets, func=""):
  if func == "":
    def func():
      pass

  octets = (octets.split('.'))
  ranges = []
  loop = 0
  for octet in octets:
    if octet.find('-') != -1:
      spot = octet.find('-') + 1
      octets[loop] = int(octet[:octet.find('-')])
      ranges.append(int(octet[spot:]) + 1)
    else:
      octets[loop] = int(octet)
      ranges.append(int(octet) + 1)
      loop += 1
  CurrentAddress = ""
  loop = 0
  output = []
  for one in range(octets[0], ranges[0]):
    for two in range(octets[1], ranges[1]):
      for three in range(octets[2], ranges[2]):
        for four in range(octets[3], ranges[3]):
          for item in (one, two, three, four):
            CurrentAddress += str \
                ((one, two, three, four)[loop]) + "."
              loop += 1
          CurrentAddress = CurrentAddress[:-1]
          output.append(func(CurrentAddress))
          CurrentAddress = ""
          loop = 0
  return output

if __name__ == '__main__':
  import os, sys, urllib2

  def defaultPrinter(ipAddress):
    try:
      print 'Trying ' + ipAddress
      theurl = 'http://' + ipAddress + '/indexConf.html'
      username = 'root'
      password = ''

      passman = urllib2.HTTPPasswordMgrWithDefaultRealm()
      passman.add_password(None, theurl, username, password)
      authhandler =  urllib2.HTTPBasicAuthHandler(passman)
      opener = urllib2.build_opener(authhandler)
      urllib2.install_opener(opener)
      pagehandle =  urllib2.urlopen(theurl)
      if pagehandle.getcode() == 200:
        output.write(ipAddress)
    except:
      pass

  output = open(sys.argv[2], 'w')
  IPRange(sys.argv[1], defaultPrinter)
  ```

Happy printer hunting.
