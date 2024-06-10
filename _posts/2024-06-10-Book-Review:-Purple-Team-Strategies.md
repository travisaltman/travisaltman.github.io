---
title: "Book Review: Purple Team Strategies"
date: "2024-06-10"
layout: post
---

Recently got done reading [Purple Team Strategies](https://www.packtpub.com/product/purple-team-strategies/9781801074292) and wanted to capture my thoughts and takeaways.  The concept of Purple Teaming in cybersecurity comes from the military use of [War Gaming](https://en.wikipedia.org/wiki/Sigma_I-67_and_II-67_war_games) where they pitted Red teams against Blue teams.  Within cybersecuity we combine these teams, hence Purple, so that defenders can learn from attackers and vice versa.


Who is this book for?  I think the audience has a broad specrum from those wanting to learn about certain aspects of Purple Teaming to an organization trying to implement a whole service dedicated to Purple Teaming.  The book is fairly comprehensive as it goes into each aspect of the various services that make up a traditional purple Team.  Below are the three main services the book describes as making up a purple team.
1. CTI (Cyber Threat Intelligence)
2. Blue Team
3. Red Team
There are plenty of books, write ups, and training dedicated to each of those services.  The book goes into detail about each service so if your organization already has those teams at some level of maturity then those sections of the book might not be as important.  For example there are three chapters on the blue team collecting telemetry, detecting attacks, and correlating all that data.

I'd argue your organization doesn't need to worry about purple teaming if you don't have robust blue team that can detect, prevent, and respond to what threat actors are throwing at you.  I'd say the same thing about the Red Team meaning if you haven't been proficient at performing attacks against yourself then get that in order before diving into the purple space.

Having built and ran purple teaming within organizations I'll go into aspects of the book that resonated with me.

Test People, Processes, and Technologies

IMHO the main objective of purple teaming is to understand the risk across all three and purple is the service that can reach across all three.  This might seem obvious but where red and blue bleed together some might get confused because the intention isn't to use the individuals services (e.g., social engineering, intel, threat actor emulation) but to combine them together.  This concept is called out early in the book of which I appreciate.

Blind Purple Teaming

On page 30 they get into the concept of "blind" purple teaming.  Most people categorize red teaming as being covert and trying to sneak by the blue team defenders.  So having red teamers keeping their activity blind from blue teamers isn't anything new but they call out in this section of the book that red team activity can be blind or fully transparent.  I like the concept of a more transparent red team engagement where red and blue are baking out the objectives of each engagement to whereas traditional red team engagements have a lower degree of input from the blue side.

Maturity Model

Once again early in the book (page 35) they address maturity modeling.  When you're building out a service it's always a great execise to establish where you might be, where you want to go, and how to get there.  Where I might have some pushback is the book suggests three levels of maturity to whereas most maturity models in the industry have five.  One good example here is https://www.redteammaturity.com/.
