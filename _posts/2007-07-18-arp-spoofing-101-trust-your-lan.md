---
title: "ARP spoofing 101: Trust your LAN?"
date: "2007-07-18"
layout: post
---

At home you're fully aware of the hosts/people that are on your network, or at least you should be. Friends and family are usually trustworthy people and you don't have to worry about them carrying out malicious activity, but what if you aren't sharing the LAN (Local Area Network) with people you can trust? This article will explain why untrusted LAN's can be dangerous and what users/admins can do to protect themselves.


If a malicious user is on the same network as you there are numerous types of attacks they could carry out to compromise your computer and identity. One common attack they might use is called ARP (Address Resolution Protocol) spoofing. Before we get into the details of ARP spoofing it's helpful to understand how this protocol came about.

The early developers of Ethernet or LAN technology designed the system based upon trust, meaning they never thought anyone using this technology would use it in a malicious way. About thirty years ago, the early stages of Ethernet development, it was hard to imagine a large scale LAN. Hardware was very expensive in those days and only large organizations could afford such technology, even then these large organizations only had a small number of computers that could communicate with one another. So it was hard to imagine that they would ever communicate with someone they didn't trust. Problem is that Ethernet exists on Layer 2 using a flat addressing scheme (e.g. MAC address). This type of addressing doesn't scale well for the internet, which uses a hierarchical addressing scheme (e.g. IP). The solution to this problem was ARP.

ARP is used for mapping IP addresses to MAC addresses. A basic diagram for ARP can be seen in figure 1 below.

![ARP protocol diagram](/assets/arpspofingdiagram.JPG)

Figure 1: ARP protocol diagram

So A wants to communicate with B:

1. A checks its local ARP table for B's info
2. A doesn't have B's info so its sends a request
3. Router closet to B adds A's info to ARP table
4. Router sends B's info to A so it can update its ARP table
5. ARP info updated, A & B can now communicate

This ARP thing seems to work out nicely, so what's the problem? In an untrusted environment how does A know that B is who he says he is? Turns out that the ARP protocol has a slew of vulnerabilities.

Vulnerabilities of ARP:

- No authentication! (you cannot be assured who you are talking to)
- ARP tables can be manipulated by anyone on the LAN
- Allows malicious users to pretend to be someone else (the SPOOF)
- Perfect protocol for launching Man in the Middle Attacks (MITM)

Once a malicious user is on your LAN they can easily manipulate ARP tables, spoof who you are trying to communicate with, and launch a MITM attack. A MITM attack allows the attacker to sniff all your traffic and pick up sensitive information (e.g. username, passwords). A diagram of a MITM attack can be seen in figure 2.

![Man in the middle diagram](/assets/maninthemiddle.JPG)

Figure 2: Man in the middle diagram

Anytime you are on an untrusted LAN (e.g. Hotel, Public WiFi, University, any large organization) there is the possibility that someone could be listening in on your communications via ARP spoofing. There are plenty of tools available that would allow even the most novice attacker to gain your personal information. So if you are going to communicate on an untrusted LAN here are some tips to stay secure.

- Use encryption (e.g. SSL, SSH, or VPN)
- If unencrypted, don't communicate private information
- Only visit trusted websites (easier said than done)

For system administrators there are two countermeasures against ARP spoofing.

1. Static ARP tables (difficult to maintain)
2. Arpwatch (will notify if someone is ARP spoofing)

The point of this article is to educate others about how protocols behind the scenes work and how insecure they can be. If you can't trust everyone on your current LAN, then use encryption and stay away from communicating sensitive information. Now you know, and knowing is half the battle. As always feel free to leave comments if you find any errors within this article.

References:

http://www.grc.com/nat/arp.htm, http://en.wikipedia.org/wiki/ARP\_spoofing
