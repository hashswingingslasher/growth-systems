# Conference Intelligence Pipeline

**Context:** A privacy-technology company that does a large share of its business development at industry conferences. Before each event, someone has to turn the speaker and exhibitor lineup into a list of people worth meeting.

## The problem

This took 8 to 15 hours per event. Write a scraper for that particular conference site, pull the speakers, look each one up, enrich them, check them against the CRM so known contacts aren't added twice, rank them, format the list, import it.

Every conference site is laid out differently, so the scraper was thrown away after each event and the work never got faster. Because the research took so long, the list often arrived late, and a lead list that arrives after the event is useless. The person doing the research was also the person who should have been preparing for the meetings.

## The constraint

Any scraper written for one site fails on the next. So the real task was to handle a page nobody had seen before without someone rewriting the extractor each time.

## Key decision 1: let the model read the page

Extraction is model-driven instead of per-site. It handles arbitrary HTML and pagination and saves as it goes, so a long run that breaks doesn't start over. One extractor works on any conference site.

It is slower and less precise than a scraper written for one site. I accepted that because the point was to stop being needed for every new event. A small, cheap model keeps the cost per page low enough to run it across a long, multi-page lineup.

## Key decision 2: spend enrichment credits where they count

Enrichment credits are limited and cost money, so the pipeline never enriches everyone by default:

- Speakers are ranked by seniority first, so credits go to decision-makers before anyone else
- Every run has a hard credit cap
- Enrichment happens only on rows a person selects
- Runs checkpoint and resume, so an interruption doesn't spend credits twice

If you can only enrich part of a lineup, that part should be the C-suite.

## Key decision 3: dry run by default

Anything that spends money or writes to the CRM runs as a dry run unless the user explicitly says otherwise.

The people using it are not engineers. It has to be safe for someone who isn't thinking about API credits or CRM hygiene while they click. A tool that can quietly spend money or pollute the CRM on a misclick stops being used.

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

The command-line pipeline came first. The browser review layer came later so the whole BD team can review and act on the list, instead of it sitting in a terminal with one person as the bottleneck.

## The tradeoff I accepted

A scraper hardcoded for one recurring event would be cleaner, and enriching everyone automatically would take fewer clicks. I took neither, because the system had to work on sites nobody had seen and stay safe for a non-engineer. The step where a person selects rows is what keeps the spending under control.

## Outcome

Extraction that used to need a custom scraper and most of a day per event now runs in minutes on a site nobody has seen before, so the time goes into reviewing the list and preparing for meetings.
