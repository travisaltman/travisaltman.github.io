---
title: "download latest metasploit behind restrictive firewalls"
date: "2009-11-14"
layout: post
---

Sometimes when you want to grab the bleeding edge version of software you'll need to utilize subversion (SVN). You can go and read [Wikipedia's take on SVN](http://en.wikipedia.org/wiki/Subversion_(software) "wikipedia's da shit, again") but basically SVN can be used to grab the latest snapshot of software. Grabbing Metasploit through SVN is the best way to get the latest exploits, payload, scanners, and auxiliary components. If you were to grab Metasploit from it's main page you would be missing a lot of that functionality, this is where SVN comes into play. Unfortunately I'm not able to grab the latest version of Metasploit because my organization has restrictive firewalls and proxies preventing me from using the SVN protocol. So the best way around this problem is to wrap the application, SVN in this case, inside of a tunneled proxy for transporting. The best implementation I've found for doing that is using SOCKS proxies.


The basic goal of this article is to explain to others how to tunnel an application in a SOCKS proxy that doesn't support SOCKS proxies. A SOCKS proxy is another network protocol but what's special about SOCKS is that it doesn't rely on the underlying packet to do it's routing. SOCKS handles the routing and basically just creates an envelope for whatever it's "wrapping up". SOCKS can work with lots of protocols (HTTP, FTP, SMTP, etc) and lots of applications (Firefox, Internet Explorer, OpenSSH, etc). One useful example of using a SOCKS proxy is tunneling HTTP traffic through an SSH tunnel. This can be accomplished because both Firefox and SSH have support for SOCKS proxies. Refer to my earlier article concerning [tunneling HTTP over SSH](http://travisaltman.com/tunneling-http-thru-ssh/ "Tunneling HTTP over SSH"). One application / protocol that SOCKS does not work with is SVN, so then how can you tunnel SVN. [Proxychains](http://proxychains.sourceforge.net/ "Da bomb") to the rescue.

Proxychains is the coolest thing since sliced bread. If an application doesn't support SOCKS then Proxychains will make it support SOCKS. Proxychains basically SOCKSifies applications. The main reason to SOCKSify an application is so that you can tunnel it through SSH because SSH supports SOCKS. So how do you download Metasploit through restrictive firewalls? The answer is ProxyChains + SVN + SSH = latest Metasploit. So enough with the yip yapping how does all this work, below are instructions.

**Requirements**

1. Internet facing listening SSH server
2. Linux client (client being your laptop or desktop) with SSH
3. Proxychains on client
4. SVN on client

You may could perform all of these steps in Windoze but why would you? Besides all of my instructions will be Linux based. Once you've got Proxychains installed (see proxychains INSTALL file) the next thing to do is edit it's config file proxychains.conf. In my situation all I had to modify were two lines. I first commented out the line that says dynamic\_chain as seen below.

```bash
# The option below identifies how the ProxyList is treated.
# only one option should be uncommented at time,
# otherwise the last appearing option will be accepted
#
dynamic_chain
#
```

Next we'll tell proxychains to use our localhost as the proxy and which port to connect to. At the very bottom of your conf file you'll need to add the following.

```bash
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5  127.0.0.1 4545
```

I randomly chose port 4545. I usually choose a port higher than 1024 because you don't need root privileges to use higher ports. Now your proxychains config file is set. Now let's create the ssh tunnel.

```bash
ssh username@sshServerIPaddress -D 4545
```

In my case it would be

```bash
ssh travis@74.208.13.81 -D 4545
```

The -D flag tells ssh to listen on your localhost (127.0.0.1) and forward that connection to your remote host, in my case 74.208.13.81. Now that you've got proxychains configured and your ssh tunnel is up and running you're ready to go. We don't need to configure SVN we just need to have the client installed. So now that you've got everything up and running simply issue the command below to download the latest Metasploit.

```bash
proxychains svn co https://metasploit.com/svn/framework3/trunk/
```

What this final command will do is use proxychains to wrap the SVN protocol into your ssh tunnel thus allowing you to download the latest version of Metasploit behind a restrictive firewall, pretty nifty huh.

Keep in mind this will download metasploit into whatever directory you happen to be in. If for example you wanted to download metasploit into your home directory (e.g /home/travis) then issue the following command.

```bash
proxychains svn co https://metasploit.com/svn/framework3/trunk/ /home/travis
```

Also keep in mind that in the above examples proxychains is assumed to be a recognized command and is set in your path. I installed proxychains in my /opt directory so I had to issue the proxychains command below. `/opt/proxychains-3.1/proxychains/proxychains svn co https://metasploit.com/svn/framework3/trunk/`

Happy sploiting and downloading, hope this explanation helps.
