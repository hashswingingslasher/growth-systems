# Conference Intelligence Pipeline

**Context:** A privacy-technology company that does a large share of its business development at industry conferences. Before each event, someone has to turn the speaker and exhibitor lineup into a list of people worth meeting.

## The problem

This was 8 to 15 hours of work per event, and it was the wrong kind of work. Write a custom scraper for that particular conference site. Pull the speakers. Look each one up. Enrich them. Cross-reference against the CRM so you are not re-adding people you already know. Rank them. Format it. Import it.

Every conference site is laid out differently, so the scraper was thrown away each time and the work never got faster with repetition. And because the research took so long, the list often landed late, which is the one thing that cannot happen. A lead list that arrives after the event has zero value.

The hours were not even the worst of it. The person burning them is the same person who should be prepping for the meetings.

## The constraint that shaped everything

Every conference website is different. Any scraper hardcoded to one site is dead on arrival at the next event. So the actual problem was never "scrape this page." It was "handle a page nobody has seen before, without a human rewriting the extractor each time."

## Key decision 1: let the model read the page

Instead of writing per-site scrapers, extraction is model-driven. It handles arbitrary HTML and pagination, and saves incrementally as it goes, so a long run that breaks does not start over. One extractor works on any conference site.

This is slower and less surgical than a purpose-built scraper for a specific site. I took that trade deliberately, because the entire point was to stop being in the loop for every new event. Using a small, cheap model keeps the per-page cost negligible, which is what makes running it across a long, multi-page lineup viable at all.

## Key decision 2: design around a scarce resource

Enrichment credits are limited and they cost real money. So the pipeline never enriches everything by default. Instead:

- Speakers are **ranked by seniority first**, so scarce credits get spent on decision-makers rather than alphabetically
- There is a **hard credit cap** on any run
- Enrichment only happens on an explicit human selection, never automatically
- Runs **checkpoint and resume**, so an interruption does not re-spend credits on work already done

Ranking before spending is the whole idea. If you can only enrich a fraction of a lineup, that fraction should be the C-suite.

## Key decision 3: dry-run by default, non-negotiable

Every operation that spends money or writes to the CRM defaults to a dry run. Nothing enriches, and nothing touches the CRM, without an explicit action from the user.

This exists because the people using it are not engineers. The tool has to be safe in the hands of someone who is not thinking about API credits or CRM hygiene while clicking. A tool that can quietly cost money or pollute the CRM on a misclick will stop being used, and it deserves to. Safety is not a feature here, it is the precondition for anyone trusting it.

## Architecture

```
Conference URL
   -> Model-driven speaker extraction
      (any site layout, pagination, incremental save)
   -> Rank by seniority
   -> Human selects who matters (review table)
   -> Enrichment on selection only (credit-capped, resumable)
   -> Cross-reference against CRM (existing vs net-new)
   -> CSV export or push net-new contacts to the CRM
```

The CLI pipeline came first and works. The browser review layer was added on top so the whole BD team can review and act, rather than the list living in a terminal and JSON files with one person as the bottleneck.

## The tradeoff I accepted

I gave up two things on purpose. A hardcoded scraper for one recurring event would be cleaner than an adaptive one. Full auto-enrichment would be fewer clicks than making someone select rows. I took neither, because the system had to work on a site nobody has seen and stay safe in the hands of a non-engineer. That human selection step is the control that keeps the spending safe. It is a feature, not friction I forgot to remove.

## Outcome

Extraction that used to need a custom scraper and most of a day per event now runs in minutes on a site nobody has seen before, so the time goes into review and meeting prep instead. The person preparing for the conference gets to spend their time preparing for the conference.
