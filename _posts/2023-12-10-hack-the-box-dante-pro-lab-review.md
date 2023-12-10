I'll start with my overall thoughts and takeaways then get into some tips and tricks to hopefully make you more successful if you decide to tackle this challenge.


##Overall thoughts

- It's definitely a challeng so if that's your style of learning then this is right up your alley especially if you don't want any hand holding along the way
- Challenging yes but rewarding when grabbing flags and completing the whole lab
- Doubling down on this it is a challenge not a course on penetration testing
- Overall structure of the lab is well thought out but just know it gets torn down and rebuilt everyday
- Time of this write up I had a deal of $20 / month (black friday deal) to access the lab but $50 / month is the standard
- The Intermediate classification is probably fair but with some caveats
  - The techniques used to exploit the systems are not overly complex but there are a wide range of those techniques
  - I'm fairly seasoned as someone who does adversary emulation and it still took me tens of hours to complete
- You're going to need help whether that's searching online or asking for help within HTB forums or discord

That being said would I take it again or do other HTB pro labs?  Maybe, I'd advise others that you'll need to dedicate time and energy if your goal is to complete the lab versus paying however much per month for access to a lab environment.  If your goal is to use this certification to break into the industry then I'd probably go into a different direction as it might be overwhelming as opposed to an exam based certification.  If your goal is to sharpen what you have then I'd say it's worthwhile even if you don't complete the entire lab.  Besides if it isn't what you thought you can always.  So overall that's my take.

##Tips and tricks

These are just overall tips and tricks I won't get too much into the nitty gritty but will link to other helpful resources.  Just like  other penetration tests it's a must you take extremly good notes especially since the lab refreshes daily.  When completing all 27 flags you'll need to be able to reference how you accomplished every single one.  Mentioned earlier the rating of Intermediate might be over stated but when trying to exploit a box what's usually presented is probably what you should dig further into.  For example if it's a wordpress website look for vulns for that.  If it's an FTP server try default creds or creds you've already obtained.  There are a handful of gotchas that aren't as straight forward and in those instances I'd search online or hit up the HTB communities.

From a technical standpoint when trying to achieve all the flags there are a handful of things to consider.
- Privilege escalation: Once you do get a shell you'll need to get root or admin access and for that PEAS is a great tool to start with
- Credential reuse: Whenever you find credentials add them to your dictionary and know the format certain brute force scanners use those dictionaries so that you can easly launch them once you've obtained new creds
- Pivoting: Other write ups mention this but what I'll add is sure you should be familiar with tools such as proxy chains, ligolo, etc. but certain tools work better in certain scenarios so best to have a playbook for your personal go to pivot tools
- Payloads and file transfers: This goes somewhat hand in hand with pivoting but reverse shells are a huge part of the lab so get with common ways (e.g., webshells, meterpreter payloads) to generate and use those

Was hesitant to put tooling as a bullet point as I think it's implied but be proficient with tools like metasploit, crackmapexec, john the ripper, nmap scripts especially brute force ones, netcat, impacket, evil-winrm, skipfish, burp, feroxbuster, sqlmap, proxy chains, ligolo, kerbrute, GetNPUsers.py, secretsdump, and rubeus just to name a few.  Some of those tools are redundant but I used just about all of them to complete the lab.  The more proficient you are with this tooling the faster you'll be able to capture all the flags.  It's important to build up your knowledgebase of these tools because it's better to have your intepretation of tool usage versus a standard readme which should help solidify your methodologies.
