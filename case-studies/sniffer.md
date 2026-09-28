# Sniffer: Shell Detection and Warm-Path Finder

**Context:** A consulting firm opening a new market: financial services companies in an offshore jurisdiction. Built as a feature inside the Network Intelligence platform.

## The problem

The team had a target list, and every name on it raised the same question: is this a real company worth pursuing, or a shell?

Offshore financial centres are full of brass-plate entities with a registered address, a PO box and no real staff. On a list, a shell and a real business look the same. So someone was checking every name by hand: open the website, search LinkedIn, work out whether anyone actually works there, guess who is senior, then try to remember whether anyone on the team knows someone inside.

That is hours of judgement per company. It doesn't scale, and it is tiring enough that people cut corners. The corner they cut is usually the warm intro they forgot they had, which is the part of business development that wins work.

## What the judgement looks like

Given a company, a good BD person asks three things. Is it real? Who matters there? Do we already have a way in? The design tries to make that repeatable, so it happens for every company on the list and not just the first ten before fatigue sets in.

## Key decision 1: turn "is this real?" into a score

I didn't want a yes/no shell flag, because no single signal can be trusted. A real company can have a dead LinkedIn page and a shell can buy a nice website. So the shell-risk score combines five independent signals: employee count, whether the LinkedIn company page is active, whether the website names a real leadership team, how many senior people can be found, and hiring activity. Each adds points to a total from 0 to 10, which maps to High, Medium or Low risk.

Five weak signals together give a far better read than any one alone, which is also how a person forms the judgement.

## Key decision 2: check free data before paying for more

Warm paths are why the team wins work, and they were the hardest part to get right, because the team's uploaded network data goes stale as soon as someone changes jobs. So the lookup runs in two tiers:

- **Tier 1 (instant, free):** check the network data already uploaded to the platform for direct matches, organisation overlap and sector proximity. No API calls.
- **Tier 2 (on demand):** only when Tier 1 finds nothing useful, search live for people at the target and cross-reference their employment history against the team's records. This catches connections who have changed jobs since the last upload.

The order protects both the team's time and its budget. A paid API call only happens when the free data has nothing to say.

## Stale data is labelled

Tier 1 results carry a warning based on when each person last uploaded their network. That stops the team acting on a connection who moved on a year ago, and it prompts them to re-upload.

## Architecture

```
Company name (+ optional location)
   -> 3 parallel checks:
        org enrichment (employee count, domain, industry)
        people search (senior people, titles, LinkedIn)
        website team-page scrape (named leadership)
   -> Shell-risk score (0-10 -> High / Medium / Low)
   -> Tiered warm-path lookup (Tier 1 uploaded data, then Tier 2 live)
   -> Structured intelligence report
Batch mode: run the whole list, rate-limited, into a ranked view
```

Organisations and people it finds flow back into the platform's rankings and outreach tracking, so each search adds to the team's shared picture.

## The tradeoff I accepted

Enriching everything up front would have been simpler to build. I ordered the pipeline so paid calls come last and only fire when the free checks come up short, which keeps each company to roughly one or two credits. The team needed something it could run across a whole market without a budget conversation each time.

## Outcome

Judgement that used to take a person's day for a handful of companies now runs across the whole market. The team spends less effort on shells and misses fewer of the warm paths it already had.
