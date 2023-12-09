---
title: "Honeypot / honeyd tutorial part 5, email alerts"
date: "2012-02-14"
layout: post
---

So this is the final article in this series of honeypots and honeyd and before I wrap it up I've gotta give big shout outs to [Neils Provos](http://www.citi.umich.edu/u/provos/) the creator of honeyd. Neils has done an excellent job with the honeyd program and his book [Virtual Honeypots](http://www.amazon.com/Virtual-Honeypots-Tracking-Intrusion-Detection/dp/0321336321) is hands down the best book about honeypots and I highly recommend picking up a copy. While writing some of these tutorials Neils was even kind enough to answer some of my emails.


Up to this point you hopefully have an understanding on how to get honeyd up and running on your preferred hardware while having the ability to run multiple honeypots in either a static or dhcp environment. Now that everything is running smoothly and you've successfully tested all connectivity you'll probably want to start getting alerts from some of the honeypots you've setup and deployed. I've written a small python script that accomplishes this for me so hopefully my explanation of the setup will get you receiving email alerts as well.

So out of the box honeyd doesn't natively support getting emails sent to you when your device is port scanned. I really wanted this feature but had to figure out the best way of determining when my honeypot was being scanned. Honeyd has the option of creating a log file so my first idea was to parse this file for items of interest. Below is the command I would use to launch honeyd with the logging option.

```bash
honeyd -d -f honeyd.conf -l /tmp/logfile
```

You should see similar output as below after running the above command.

```bash
Honeyd V1.5c Copyright (c) 2002-2007 Niels Provos
honeyd[12073]: started with -d -f honeyd.conf -l /tmp/logfile
Warning: Impossible SI range in Class fingerprint "IBM OS/400 V4R2M0"
Warning: Impossible SI range in Class fingerprint "Microsoft Windows NT 4.0 SP3"
honeyd[12073]: listening promiscuously on eth1: (arp or ip proto 47 or (udp and src port 67 and dst port 68) or (ip )) and not ether src 00:0c:29:11:1e:53
honeyd[12073]: [eth1] trying DHCP
honeyd[12073]: Demoting process privileges to uid 65534, gid 65534
honeyd[12073]: [eth1] got DHCP offer: 192.168.134.147
honeyd[12073]: Updating ARP binding: 00:00:24:54:9e:06 -> 192.168.134.147
honeyd[12073]: arp reply 192.168.134.147 is-at 00:00:24:54:9e:06
honeyd[12073]: Sending ICMP Echo Reply: 192.168.134.147 -> 192.168.134.254
honeyd[12073]: arp_send: who-has 192.168.134.254 tell 192.168.134.147
honeyd[12073]: arp_recv_cb: 192.168.134.254 at 00:50:56:e8:c9:74
```

So this just created a honeypot with the IP address of 192.168.134.147. From another machine let's port scan our honeypot, to keep the output simple I'm only going to scan one port.

```bash
root@tht:~# nmap -p 135 192.168.134.147

Starting Nmap 5.21 ( http://nmap.org ) at 2012-02-10 19:37 PST
Nmap scan report for 192.168.134.147
Host is up (0.0013s latency).
PORT STATE SERVICE
135/tcp open msrpc
MAC Address: 00:00:24:54:9E:06 (Connect AS)

Nmap done: 1 IP address (1 host up) scanned in 0.14 seconds
```

So port 135 is open so we know our configuration is working properly. Also honeyd will output information to the terminal letting you know that connections are being made to your honeypot. The information below was appended to the output from when we started honeyd.

```bash
honeyd[12073]: arp reply 192.168.134.147 is-at 00:00:24:54:9e:06
honeyd[12073]: arp reply 192.168.134.147 is-at 00:00:24:54:9e:06
honeyd[12073]: Connection request: tcp (192.168.134.143:49677 - 192.168.134.147:135)
honeyd[12073]: arp_send: who-has 192.168.134.143 tell 192.168.134.147
honeyd[12073]: arp_recv_cb: 192.168.134.143 at 00:0c:29:e3:2a:39
honeyd[12073]: Connection dropped by reset: tcp (192.168.134.143:49677 - 192.168.134.147:135)
honeyd[12073]: arp_recv_cb: 192.168.134.254 at 00:50:56:e8:c9:74
honeyd[12073]: arp_recv_cb: 192.168.134.254 at 00:50:56:e8:c9:74
```

On line three we see a connection request from 192.168.134.143 to our honeypot at 192.168.134.147 and the ":135" after the IP address is the destination port so from this output we can verify everything is working and we're getting the correct response from our port scan. You also see on line six that the connection from 192.168.134.143 to 192.168.134.147 was dropped. Now let's take a look at our log file to see what kind of information about our port scan shows up. We'll use the tail command in Linux, by default the tail command only shows the last 10 lines of a file although this can be adjusted.

```
root@bt:~# tail /tmp/logfile
2012-02-10-22:37:42.9749 udp(17) - 192.168.134.1 61712 224.0.0.252 5355: 60
2012-02-10-22:37:48.0504 udp(17) - 192.168.134.143 44907 192.168.134.2 53: 74
2012-02-10-22:37:48.0751 udp(17) - 192.168.134.2 53 192.168.134.143 44907: 74
2012-02-10-22:37:48.0799 tcp(6) S 192.168.134.143 49677 192.168.134.147 135
2012-02-10-22:37:48.0814 tcp(6) E 192.168.134.143 49677 192.168.134.147 135: 0 0
2012-02-10-22:38:00.0874 udp(17) - 192.168.134.1 57479 224.0.0.252 5355: 55
2012-02-10-22:38:02.9374 udp(17) - 192.168.134.1 64855 224.0.0.252 5355: 55
2012-02-10-22:38:09.9825 udp(17) - 192.168.134.1 57141 224.0.0.252 5355: 57
2012-02-10-22:38:13.2114 udp(17) - 192.168.134.1 60692 224.0.0.252 5355: 56
2012-02-10-22:38:16.0751 udp(17) - 192.168.134.1 57282 224.0.0.252 5355: 56
```

In the log file we see very similar information on lines 5 and 6 as we did in the standard output of the terminal. Turns out there's a third location of output that we could tap into. All processes in Linux will probably display some kind of information into one of two system log files. Either the /var/log/messages or the /var/log/syslog file. Different [distros](http://distrowatch.com/) of Linux will put this information into different locations but for the backtrack distro I know it goes into /var/log/syslog. Let's tail this file to see what kind of information it holds.

```
Feb 10 22:36:53 bt honeyd[12073]: arp reply 192.168.134.147 is-at 00:00:24:54:9e:06
Feb 10 22:36:53 bt honeyd[12073]: Sending ICMP Echo Reply: 192.168.134.147 -> 192.168.134.254
Feb 10 22:36:53 bt honeyd[12073]: arp_send: who-has 192.168.134.254 tell 192.168.134.147
Feb 10 22:36:53 bt honeyd[12073]: arp_recv_cb: 192.168.134.254 at 00:50:56:e8:c9:74
Feb 10 22:37:48 bt honeyd[12073]: arp reply 192.168.134.147 is-at 00:00:24:54:9e:06
Feb 10 22:37:48 bt honeyd[12073]: arp reply 192.168.134.147 is-at 00:00:24:54:9e:06
Feb 10 22:37:48 bt honeyd[12073]: Connection request: tcp (192.168.134.143:49677 - 192.168.134.147:135)
Feb 10 22:37:48 bt honeyd[12073]: arp_send: who-has 192.168.134.143 tell 192.168.134.147
Feb 10 22:37:48 bt honeyd[12073]: arp_recv_cb: 192.168.134.143 at 00:0c:29:e3:2a:39
Feb 10 22:37:48 bt honeyd[12073]: Connection dropped by reset: tcp (192.168.134.143:49677 - 192.168.134.147:135)
```

On lines 7 and 10 we see similar information as we've seen in other areas. So when I decided to build an email alert script I had three sources of information I could pull from. I eventually went with combing through /var/log/syslog but I could have easily went a different route. For me /var/log/syslog seem to have more verbosity and better keywords but then again that's just my opinion. My next step was to write a script that would parse /var/log/syslog then generate an email alert when it saw connections to my honeypot. I wrote my script in [Python](http://python.org/) because I'm most familiar with that language and Python comes installed by default on most Linux distributions so whatever distro you decide to run honeyd on it's more than likely Python will already be installed on that same operating system.

Before you implement any scripted email solution it's a good idea to test email functionality with a small email script just to ensure you can properly communicate and receive email alerts. To do this you'll need to know the name ([FQDN](http://en.wikipedia.org/wiki/Fully_qualified_domain_name)) of your [SMTP](http://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) email server. You can usually find them in your email client. For Outlook you can go to File > Account Settings > Account Settings > Email tab > click on Change, there you'll see the name of your organizations smtp server. Typically it'll be something simple, if the email of your organization ends in example.com then your smtp server will likely be smtp.example.com. SMTP servers are usually configured to send emails in one of two ways, authenticated or unauthenticated. We'll look at examples for both. The first example below is a python script of sending authentication credentials along with the request, in this particular I'm sending the alert to my gmail account.

```python
import smtplib
import time

From = 'someuser@gmail.com'
To = 'travisaltman@gmail.com'
Date = time.ctime()
Subject = 'test'
Text = 'test'

Message = ('From: %s\nTo: %s\nDate: %s\nSubject: %s\n%s\n' % (From, To, Date, Subject, Text))
username = 'someuser'
password = 'somepassword'

print ('Connecting to server')
s = smtplib.SMTP ('smtp.gmail.com')
s.starttls()
s.login(username,password)
sendMail = s.sendmail(From, To, Message)
s.quit

if sendMail:
  print ('error')
else:
  print ('great success')
```

Hopefully some of this script is easy to figure out, the main thing you need to be concerned with is lines 11,12, and 15. Lines 11 and 12 hold the username and password you'll need to authenticate to your smtp server and line 15 is the name of your smtp server. More than likely your internal smtp server will not require authentication if that's the case simply remove lines 11,12,16, and 17 from the script above. If you're not sure first send the test email without authentication and if you get the error message "smtplib.SMTPSenderRefused" then more than likely you'll need to provide credentials. If everything goes smooth running your test email script then you should see the output below, here I've named my script test.py.

```bash
root@tht:~# python test.py
Connecting to server
great success
```

Next you can implement the full email alerting script. Simply copy and paste the script below into your text editor of choice and name the script, I've named my alert.py.

```python
import re
import time
import smtplib
import os

try:
  os.remove('/root/outputHoney')
except:
  pass

log = "/var/log/syslog"
output = 'outputHoney'

systemTime = time.ctime()
loggedDate = systemTime[4:10]
loggedYear = systemTime[20:24]
hostFile = '/etc/hostname'
readHostFile = open(hostFile, 'rU').readline();
extractHostName1 = readHostFile.split('"')
hostname = extractHostName1[0].rstrip()

file = open(log, 'rU').readlines();
for line in file:
if re.search(loggedDate, line):
  if re.search("192.168.1.55", line): #This if for any IP(s) you want to exclude
    ignore = ''
  elif re.search("Connection request", line):
    loggedTime = line[7:15]
    timeString = loggedDate + ' ' + loggedYear + ' ' + loggedTime
    timeTuple = time.strptime(timeString, '%b %d %Y %H:%M:%S')
    epochLogTime = int(time.mktime(timeTuple))
    epochSystemTime = int(time.time())
    if epochSystemTime <= epochLogTime+300:
      lineSplit1 = line.split('(')
      lineSplit2 = lineSplit1[1]
      lineSplit3 = lineSplit2.split(':')
      lineSplit4 = lineSplit2.split('-')
      lineSplit5 = lineSplit4[1]
      lineSplit6 = lineSplit5.split(':')
      destinationIP = lineSplit6[0]
      sourceIP = lineSplit3[0]
      srcAndDest = sourceIP + ' connected to ' + destinationIP
      file = open(output, 'w')
      file.writelines(srcAndDest)
      file.close
if os.path.exists('/root/outputHoney'):
  From = hostname+'@example.xxx'
  # To = ['user1@example.xxx','user2@example.xxx']
  To = 'travisaltman@gmail.com'
  Date = time.ctime()
  Subject = 'honeypot alert'
  username = 'someusername'
  password = 'somepassword'
  # IPSconnecting = open(output, 'r').read()
  Text = "A device at " + sourceIP + " is port scanning our honeypot at " \
  + destinationIP + ". This honeypot is being emulated on device " \
  + hostname + "."
  Message = ('From: %s\nTo: %s\nDate: %s\nSubject: %s\n%s\n' % (From, To, Date, Subject, Text))
  s = smtplib.SMTP ('smtp.gmail.com')
  s.starttls()
  s.login(username,password)
  sendMail = s.sendmail(From, To, Message)
  s.quit
```

I'm not a developer and I always mangle my scripts together so this isn't the prettiest code. I'll give you a run down of what this script does, if you have any specific questions please feel free to leave a comment I generally respond to comments fairly quickly. Basically the script combs through /var/log/syslog looking for the string "Connection request". I've also confirmed that this script works just as well combing through /var/log/messages, you'll have to verify which log your Linux distro is dumping this information. To test the script to make sure it works I would first port scan your honeypot from another device then simply run the python script to see if you get no errors and hopefully you get an email in your inbox, just run the following command after you've port scanned your honeypot.

```bash
python alert.py
```

This command should only run for maybe 10 seconds then return you to the command line. If you get any errors then you'll have to trouble shoot the script, contact me if you need help with that. Just to give you an idea of what the email alert would look like I've got a screen shot of an alert I got sent to my gmail address below.

![](/assets/Gmail-honeypotAlert-travisaltman@gmail.png "Gmail-honeypotAlert-travisaltman@gmail")

Currently the script only looks for "Connection request" in the last five minutes of log files so you'll need to combo that up with running the script every five minutes, this can be done with Linux's [crontab](http://ss64.com/bash/crontab.html). [Cron](http://en.wikipedia.org/wiki/Cron) can schedule programs to be run at certain frequencies. To set up alert.py to run every five minutes use the crontab command.

```bash
crontab -e
```

This opens up a text file in the terminal where you enter a specific syntax to tell cron which program you want to run and how often. Enter the text below to tell cron to run alert.py every five minutes.

```bash
*/5 \* \* \* \* /root/alert.py
```

Then use control+w to save your work and control+x to quit the text editor. So in the alert.py script the every five minutes comes from line 33 where 300 is seconds which equals five minutes so if you wanted to modify the time you could do it on that line. Another line in the script you may want to change is line 25, here you can add any IP's that you want to ignore for whatever reason. You'll definitely want to change line 55 that has the text of the email you'll be receiving and customize that to your hearts content. Don't forget to modify smtp server information and also remove or change the authentication piece as needed. Also in the screenshot you'll notice that the device is "bt" which is short for backtrack. I implemented this feature because you may want to run honeyd on multiple devices throughout your network and you'll want to know what device is sending you the email. The name of the device is determined in lines 17-20. You may have to modify that code because not all distros of Linux keep their hostname in that location and you may have to parse the text file that holds that hostname in a different manner. There's more information that I could go into about the script but hopefully I've hit the major points.
