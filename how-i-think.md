# How I think about building growth systems

Most growth problems are execution problems wearing a strategy costume. The plan is usually fine. What breaks is that it needs 40 hours of manual work a week that nobody has. So I look for the manual work that is swallowing the team and build the system that takes it over.

The principles I keep coming back to:

## Strategy first, then architecture, then the build

I start with the growth question: what the business is trying to win and who it needs to reach. That is marketing and growth work, and it decides everything after it. Then I design the system around that answer. Then I build it and run it in production. The order matters, because a system built before the question is settled answers the wrong question very efficiently.

## Start from the work, not the tool

I don't begin with "we should use an AI agent here". I begin with what someone on the team does by hand every week that a machine could do. The tool is the last decision.

## Ship the version that works now

A small system that works this week beats a perfect one that ships next quarter. Revenue and usage teach you more than a roadmap does. Every system in this repo started small and grew from there.

## Data quality decides whether anyone uses it

A GTM system that enriches the wrong contacts faster is worse than having none. I spend more time on verification and deduplication than on the part people see, because a sales team stops trusting a tool the first time it burns them.

## Name the cost

Every one of these builds gave something up. An adaptive scraper that handles any event site is slower than one written for a single site. I would rather say that plainly than pretend a system has no downside.

## It's done when someone else can run it

If a system still needs me in the loop, it's a demo. The bar is that the team runs it after I step away.
