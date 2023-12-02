---
title: "Why your organization should be doing Breach & Attack Simulations"
date: "2019-02-01"
layout: post
---

Some would say what's old is new again when it comes to a phrase like "breach and attack simulations".  How is this different from vulnerability scanning, pentesting, or red teaming?  Really it's more of a maturation of cyber security services so if your organization doesn't currently employ a combination of vulnerability scanning, penetration testing, or red teaming then breach and attack simulations services should probably be lower on your list.


What is Breach & Attack Simulation (BAS)?

> _"It's an automated  or semi-automated emulation of threat actors TTP's (tactics, techniques, and procedures) against information systems within your organization's environment to determine how effective your current controls would protect, detect, and defend those information systems from malicious users."_

Now there are various definitions out there but think that pretty much sums up the intent of having a service like this in place.  That may sound like what red teaming and penetration testing are meant to accomplish but they aren't one to one and their objectives are completely different.  I've done this before and called it an "assumed compromise" so really it's a play on that but calling out breach and attack simulation as a separate capability or service helps more clearly define the objective you're going after.

So how do the services differ and what are the main goals of introducing something like an assumed compromise or breach simulation?  The quick breakdown on the differences below should help.

| Feature          | BAS              | Pentesting         | Red Teaming        |
|------------------|------------------|-----------------|-----------------|
| Objective      | Control and posture     |  Identify weakness     |  Bolster blue team    |
| Consistency      | High: run exact same scenario every time      | Medium: follows a framework but human driven   | Low: depends up conditions     |
| Attack elements      | Tenth entry      | Eleventh entry  | Twelfth entry   |
| Thirteenth entry | Most if not all stages | Usually focused on recon & exploitation | Varies but tends to be post exploitation |

This break down should help delineate the services and as you're trying to mature cyber security services within your organization and leverage the table above to highlight the benefit that a BAS would provide.  So as you're making the case for BAS within your organization whether that's additional head count to support the service or if it's simply a service you'd like to introduce alongside other services the above table will point your leadership in the right direction but let's dig deeper into the benefits of BAS.

**Key Benefits of BAS**

1. **Cost**:  Spinning up resources to perform penetration testing or red teaming requires a lot of cycles.  Wing to wing penetration tests and red team assessments can last anywhere from 3 - 6 months and involve numerous resources.  Having a BAS solution is more cost effective as it eliminates the need for additional resources plus automates the task of pentesting and red teaming activities.
2. **Mimics larger set of TTP's consistently**:  Penetration Testing and Red Teaming usually only care about the end game and aren't necessarily interested in testing the whole environment to whereas BAS will test across the board all the different scenarios that a threat actor might leverage.  This is key as red teams or pentesters may not run a particular scenario for a number of reasons but BAS would cover that and may find something others would miss.
3. **Agile**:  Similar to the ease of launching a vulnerability scanner against a particular target leveraging a BAS solution one should easily be able to kick off a simulation against an environment.
4. **Evaluation**:  Having the ability to run the simulations in almost any environment is very handy.  As most know the attack surface across the organization varies and if it were all the same then that'd be a whole lot easier to defend but we don't live in [rainbow land](https://www.youtube.com/watch?v=C-ztmhwkEUw).  Whether it's running BAS on a more frequent basis to test your organization's core detection capabilities or if it's testing it against a business function within your organization that has customized applications and configurations BAS allows you to quickly and easily evaluate your risk posture.

Beyond some of the key benefits and the differences in technologies or services BAS will afford your organization what you really want from any new offering is what it's going to achieve.  I've defined and highlighted features of BAS but the **_main intent is to discover any cyber security controls that may be deficient_** in your environment so to that point what are some of the main features that BAS will employ to help identify those gaps?

- Network assessment:  The multiple attack simulations will test all the various NIDS and alerts the SOC has setup to see which ones are getting notified appropriately
- Data exfiltration:  Will test various outbound techniques to see what controls are in place to defend against attackers getting information outside your boundaries
- Lateral movement:  Can perform threat actor techniques for such things as privilege escalation and lateral movement within your network and information systems
- Endpoint assessment:  Anti-virus, end point logging, and various other technologies are there to help alert or contain a threat so BAS has features to identify any gaps that might be missing with endpoint protections
- Email gateway:  Hopefully your email gateway is working like it should to protect your organization but if not BAS has the capability to test some of the various techniques threat actors are leveraging (office docs, pdfs, .Net apps, etc.) to get passed your defenses

This list could go on for quite a while but these are some of the major categories where a BAS solution could test cyber security controls that are simply not working at all or not deployed effectively.

Anytime you're trying to sell something new within your organization it's probably best to think of all the questions someone will ask whether that's being inquisitive about the solution or from a devils advocate perspective.  Leaving off the cost aspect as that will vary hopefully my points help to answer potential questions but there are some questions that you can ask to leadership or folks within your organization that might shed light on how a BAS solution can bridge those gaps.

_Can we detect simple and complex threat actor attacks?_

_Can various types of malware be transferred through our boundaries?_

_Will our alerting logic correlate certain events?_

_How secure are we from all the various TTP's?_

_Could we respond quickly enough (ransomware, worms, etc.)?_

_Can an attacker run code or bypass application controls undetected?_

_If we ran this in other environments would we see it?_

These are just some of the questions that BAS is meant to help answer.  Others within your organization may be able to either fully or partially answer these questions but no one is 100% secure and as we play this cat & mouse game with adversaries that are looking to find any crack within our armor then leveraging something like a BAS solution will help us to answer those questions more definitively and better yet put a solution in place that can be better adapted as you move forward facing even greater challenges.

References

[https://misti.com/infosec-insider/a-primer-on-breach-and-attack-simulations](https://misti.com/infosec-insider/a-primer-on-breach-and-attack-simulations)

[https://www.gartner.com/doc/3875421/utilizing-breach-attack-simulation-tools](https://www.gartner.com/doc/3875421/utilizing-breach-attack-simulation-tools)
