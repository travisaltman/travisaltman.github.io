---
title: "Book Review: Purple Team Strategies"
date: "2024-06-10"
layout: post
---

Recently got done reading [Purple Team Strategies](https://www.packtpub.com/product/purple-team-strategies/9781801074292) and wanted to capture my thoughts and takeaways.  The concept of Purple Teaming in cybersecurity comes from the military use of [War Gaming](https://en.wikipedia.org/wiki/Sigma_I-67_and_II-67_war_games) where they pitted Red teams against Blue teams.  Within cybersecuity we combine these teams, hence Purple, so that defenders can learn from attackers and vice versa.


Who is this book for?  I think the audience has a broad spectrum from those wanting to learn about certain aspects of Purple Teaming to an organization trying to implement a whole service dedicated to Purple Teaming.  The book is fairly comprehensive as it goes into each aspect of the various services that make up a traditional purple Team.  Below are the three main services the book describes as making up a purple team.

1. CTI (Cyber Threat Intelligence)
2. Blue Team
3. Red Team

There are plenty of books, write ups, and training dedicated to each of those services.  The book goes into detail about each service so if your organization already has those teams at some level of maturity then those sections of the book might not be as important.  For example there are three chapters on the blue team collecting telemetry, detecting attacks, and correlating all that data.

I'd argue your organization doesn't need to worry about purple teaming if you don't have a robust blue team that can detect, prevent, and respond to what threat actors are throwing at you.  I'd say the same thing about Red Team meaning if you haven't been proficient at performing attacks against yourself then get that in order before diving into the purple space.

Having built and ran purple teaming within organizations I'll go into aspects of the book that resonated with me.

## Assess People, Processes, and Technologies

IMHO the main objective of purple teaming is to understand the risk across all three and purple is the service that can reach across all three.  This might seem obvious but where red and blue bleed together some might get confused because the intention isn't to use the individuals services (e.g., social engineering, intel, threat actor emulation) but to combine them together.  This concept is called out early in the book of which I appreciate.

## Blind Purple Teaming

On page 30 they get into the concept of "blind" purple teaming.  Most people categorize red teaming as being covert and trying to sneak by the blue team defenders.  So having red teamers keeping their activity blind from blue teamers isn't anything new but they call out in this section of the book that red team activity can be blind or fully transparent.  I like the concept of a more transparent red team engagement where red and blue are baking out the objectives of each engagement to whereas traditional red team engagements have a lower degree of input from the blue side.

## Maturity Model

Once again early in the book (page 35) they address maturity modeling.  When you're building out a service it's always a great exercise to establish where you might be, where you want to go, and how to get there.  Where I might have some push back is the book suggests three levels of maturity to whereas most maturity models in the industry have five.  One good example here is https://www.redteammaturity.com/.

## Service Output

There isn't a standard output for most cybersecurity solutions and purple teaming isn't an exception to this.  Think the book takes a decent stab outlining a report template that can be used to capture the output of purple teaming.  In the past I've used everything from power point to ticketing software to capture the output of purple team activities.  What I will say is that their suggestion near page 49 for capturing the output is a solid start.

## C2 (Command & Control) Variations

In chapter 5 near page 122 they get into various ways of leveraging C2's.  This resonated with me as it's been my experience that when trying to emulate threat actor [TTPs](https://csrc.nist.gov/glossary/term/tactics_techniques_and_procedures#:~:text=A%20tactic%20is%20the%20highest%2Dlevel%20description%20of%20the%20behavior,the%20context%20of%20a%20technique.) there isn't one way to go about this.  It's very common that threat actors leverage C2's but that's not a requirement when performing purple teaming.  In the book they mention three variations being phishing, short term, and long term.  I'd expand upon that not with a definitive number of variations but with the emphasis on being flexible in how you emulate threat actors and let the objective of the purple team drive the execution.  There are plenty of attacks you might want to validate defensive controls are in place that don't involve a C2 or you can emulate the entire [kill chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html) with a fully setup C2 infrastructure (e.g., domains, redirectors, web servers).  Like how they call this out but I'd just double down there isn't just X number of ways to tackle this approach.

## Simulation vs Emulation

In chapter 9 around page 229 they go into the concept of simulation versus emulation.  They do mention the differences are subtle and that "the community is not fully aligned" but I would argue they have the definitions backwards.  They define emulation as following exact steps laid out in assessment plan but I've always viewed that as simulation.  AKA simulation is more strict to whereas emulation is more free flowing.  I've written about BAS (breach and attack simulation) before and that's normally associated with tooling that automates a bunch of TTPs.  So the fact that BAS has simulation in the name and is more automated aligns better with the definition I would suggest.

## Final Thoughts

If you play in the cybersecurity space at all I think the book is a solid reference.  Because it's not just about purple teaming but covers aspects across a traditional SOC might face in day to day operations.  There's plenty I didn't cover but these are just some aspects of the book that were worth calling attention too.
