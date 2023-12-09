---
title: "Honeypot / honeyd tutorial part 4, hardware"
date: "2011-11-12"
layout: post
---

So up to this point you've probably only ran honeyd on your laptop or desktop machine. If you want to get the most out of honeyd then you'll probably want to run it on either a server or an embedded device. In the beginning of this series I mentioned you could run a honeypot in a number of ways. Two of the ways I mentioned was to attract malware to a vulnerable system so that you can analyze the latest and greatest malware. The other way was to attract attackers on your network. In my series I'm going to keep the focus on detecting attackers on the local network and not trying to find new malware. The [honeynet project](http://www.honeynet.org/) already does a great job of tracking down the latest and greatest malware so check that project out.


If you're going to use honeyd to detect attackers on your local network then you'll need to place your honeypot as close to your networking equipment as possible. This being the case a racked server or small device with numerous network interfaces will likely be the best solution. A racked server didn't make much sense for my solution mainly due to the cost but a small embedded device would need good specs to make a good solution. I found a couple of solutions.

First the option that I went with to implement my honeypot running honeyd. [The Soekris Net5501](http://soekris.com/products/net5501.html).

![](/assets/net5501_BC_front_overview.jpg "net5501_BC_front_overview.jpg")

![](/assets/net5501_BC_back_overview.jpg "net5501_BC_back_overview")

The great thing about the Soekris is that it has four network interfaces. This allows you access to four different vlans within your environment. Out of the box it comes with the ability to load an OS on compact flash, which is the option that I went with. You could also get a PCI extension that could be fitted with a hard drive. If your install of a honeypot would require large data storage then you'll need to think about that option. I did not care about data storage, I simply wanted an alert when honeyd saw something come across the wire. Besides even if you needed storage you could have honeyd ship off that data / logs to a centralized location. For $250 bucks this is a great solution. You won't find to many small devices like this that have four network interfaces. Now you could get a full rack system with plenty of network interfaces but then your cost goes up. More network interfaces would mean that you have access to more vlans so if that's important to you then you'll have to plan accordingly. This is setup is not meant to cover your entire organization just a handful of important vlans. Below is a diagram of a potential setup.

![](/assets/soekrisDiagram.png "soekrisDiagram")

In this setup you can place a honeyd host on four different vlans looking for any devices that connect to your honeypot when they should have no business connecting to your honeypot. Keep in mind this is not meant to be an intrusion detection replacement. This solution will ride on top of your existing intrusion detection. Besides most intrusion detection setups that I've seen don't monitor activity inside a particular vlan much less traffic between vlans. The setup I've described here is meant to monitor vlans with important assests and data. So in the scenario above you would have to connect your Soekris device to a "core" router that has the vlans you want to monitor. You could also connect the Soekris device to multiple routers if those vlans are mananged by different routers. There are numerous ways to tackle a problem that I've described but this is just one of those ways.

There is another device that I believe is very handy in these types of situations and that's the [Alix boards by PC Engines](http://pcengines.ch/alix.htm). If you want to buy one I would recomend [NetGate](http://store.netgate.com/PC-Engines-C69.aspx) which also has other options such as enclosures and lots of other wireless goodness. The Alix board plus enclosure is very small, about the same size of a home wireless router. Below is an Alix board without the enclosure.

![](/assets/ALIX_2D13.jpg "ALIX_2D13")

You can buy them with a number of configurations, the one above has three network interfaces, one compact flash, one mini pci, plus a cpu and RAM. So no hard drive but you can easily run your OS on the compact flash. Of course the OS of choice should be Linux. Hopefully this answers the question as to what hardware you might could use for your installation of a honeypot in your organization's environment. Unless you do everything at your organization this type of work will require you to work closely with your network engineering team. That's all I have, I'd love to hear from others on how they have their honeypots setup and what hardware is powering that setup. Please comment if you have a question.
