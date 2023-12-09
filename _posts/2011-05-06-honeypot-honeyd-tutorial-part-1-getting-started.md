---
title: "Honeypot / honeyd tutorial part 1, getting started"
date: "2011-05-06"
layout: post
---

If you've somehow found my obscure site then you probably already know a little bit about honeypots and their functionality, if not [here is a good breakdown](http://www.honeypots.net/). There are many different types of honeypots and these different types are explained very well in the book [Virtual Honeypots](http://www.amazon.com/Virtual-Honeypots-Tracking-Intrusion-Detection/dp/0321336321) which I highly recommend you read if you are serious about deploying a honeypot.


This series of articles will focus on honeypots using an application called [honeyd](http://www.honeyd.org/). There are a number of honeypot solutions out there but I personally feel like honeyd is a great fit because it can be relatively simple or you can start tweaking it to get a more full featured product. You may think of honeypots as internet facing and it's true that they can be configured that way but during this series of tutorials I will only be using honeyd on an internal network. Internet facing honeypots are mainly used to research and find new malware, internal honeypots are mainly used as alerting systems that would alert you when other devices / users are connecting to your honeypots. You can also use honeyd when investigating malware which I'll discuss in a later tutorial.

For this tutorial I will be using one Windows machine and one Linux machine, [Backtrack](http://www.backtrack-linux.org/) distribution to be exact. Backtrack will be the machine that is running honeyd. Honeyd is available for Windows but I highly recommend that you use honeyd on Linux. If you're half way interested in information security then I suggest that you get to know Linux as there are a lot of information security tools such as honeyd that use Linux. Sorry for the Linux rant, below is basic diagram of my setup.

![](/assets/Selection_171.png "Selection_171")

The idea here is that we'll install and configure honeyd on Backtrack then simply test that we have connectivity with our Windows machine. To see if you have honeyd installed on Backtrack (or any Linux system) simply type "honey + TAB", if "d" is shown right after honey then you know you have honeyd installed as it is an available command if you don't have honeyd installed on Backtrack run the following command

```bash
sudo apt-get install honeyd
```

This will also work for any Debian based Linux system. To install on other distributions such as Gentoo, Fedora, Slackware, etc I would check their documentation on how to install packages. After honeyd is installed the next thing we'll need to do is create a configuration file. A honeyd configuration file is the heart of your honeypot. The configuration file tells honeyd what operating system to emulate, what ports to open, what services should be ran, etc. This config file can be tweaked to emulate all sorts setups but for right now let's look at a simple setup and get that up and running. Below is my config file.

```bash
create default
set default default tcp action block
set default default udp action block
set default default icmp action block

create windows
set windows personality "Microsoft Windows XP Professional SP1"
set windows default tcp action reset
add windows tcp port 135 open
add windows tcp port 139 open
add windows tcp port 445 open

set windows ethernet "00:00:24:ab:8c:12"
dhcp windows on eth0
```

Within Backtrack you can use Kate or nano text editors to create this file. In Backtrack Kate is under the Utilities menu. The "create default" section simply tells honeyd to drop traffic unless it is defined later in the configuration file. I find this section is needed when you let your honeypot acquire an IP address via dhcp. Also it's probably a good idea to implement this section so that you only answer to network connections that you define later in the config file. Anytime you see "create" within the config file you are creating a template for a honeypot, so you can create as many honeypots as you'd like within the honed.conf config. In the windows template we are defining a number of things. First we are setting the personality, meaning when another device on the network connects to this honeypot it will appear to be a Windows XP Pro SP1 device. This is emulated via network stack fingerprints. In the windows template I'm also opening up three ports (135, 139, and 445). These are common ports that are open on a windows system. The "action reset" statement will drop traffic if it is not aimed at the open ports defined in this config. The "set windows ethernet" sets a MAC address for our honeypot.  This will be needed if you run your honeypot via dhcp. You can simply make up any MAC address you'd like, I usually keep it close to the physical MAC address that I'm running the honeypot off of. Finally the dhcp statement tells the windows template to acquire an IP address from dhcp. Now that we have our honeyd.conf file properly setup it's time to launch honeyd, below is the command I use when initially getting honeyd up and running.

```bash
honeyd  -d  -f  honeyd.conf
```

Here we use the -d so that it doesn't run in the background (or doesn't run as a daemon in Linux terms). This allow for more verbose output so that we can troubleshoot as needed. Running in this mode will also show the IP that was given to our honeypot via dhcp. Below is the type of output you should see after running the honeyd command.

```bash
Honeyd V1.5c Copyright (c) 2002-2007 Niels Provos
honeyd[1870]: started with -d -f honeyd.conf
Warning: Impossible SI range in Class fingerprint "IBM OS/400 V4R2M0"
Warning: Impossible SI range in Class fingerprint "Microsoft Windows NT 4.0 SP3"
honeyd[1870]: listening promiscuously on eth0: (arp or ip proto 47 or (udp and src ...
honeyd[1870]: [eth0] trying DHCP
honeyd[1870]: Demoting process privileges to uid 65534, gid 65534
honeyd[1870]: [eth0] got DHCP offer: 192.168.99.135
honeyd[1870]: Updating ARP binding: 00:00:24:c8:e3:34 -&gt; 192.168.99.135
```

In this verbose output we see that dhcp gave our honeypot the address of 192.168.99.135. From our windows machine let's ping that IP address and make sure that we have connectivity. You should see output on the terminal similar to below.

```bash
honeyd[1870]: arp reply 192.168.99.135 is-at 00:00:24:c8:e3:34
honeyd[1870]: Sending ICMP Echo Reply: 192.168.99.135 -&gt; 192.168.99.128
honeyd[1870]: arp_send: who-has 192.168.99.128 tell 192.168.99.135
honeyd[1870]: arp_recv_cb: 192.168.99.128 at 00:0c:29:7e:60:d0
honeyd[1870]: Sending ICMP Echo Reply: 192.168.99.135 -&gt; 192.168.99.128
honeyd[1870]: Sending ICMP Echo Reply: 192.168.99.135 -&gt; 192.168.99.128
honeyd[1870]: Sending ICMP Echo Reply: 192.168.99.135 -&gt; 192.168.99.128
```

So congrats you've successfully deployed honeyd. We can now ping our honeypot but we need to make sure the ports we've configured to be open are open. Let's us the cadillac of port scanners [nmap](http://nmap.org/) to detect open ports on our honeypot. You can scan for all 65,535 ports on our honeypot but to keep the verbose output of honeyd low let's just scan for a handful of ports. Below is the nmap command I used.

```bash
nmap -p 135,139,445,1337 192.168.99.135
```
The output of this command should look similar to below.

```bash
Starting Nmap 5.00 ( http://nmap.org ) at 2011-05-06 13:13 EDT
Interesting ports on someone (172.20.73.77):
PORT     STATE  SERVICE
135/tcp  open   msrpc
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds
1337/tcp closed waste
MAC Address: 00:00:24:26:C4:ED (Connect AS)

Nmap done: 1 IP address (1 host up) scanned in 0.37 seconds
```

So honeyd appears to be working correctly. If you've reached this point then you are on your way to doing even more with honeypots and honeyd. The main purpose of this article was to get you up and running. In the next series of articles we'll configure more honeypots, set static IP's, get alerts on devices port scanning our honeypots, investigate malware, etc.
