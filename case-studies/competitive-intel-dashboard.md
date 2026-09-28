# Competitive Intelligence Dashboard

**Context:** A privacy-technology company in a fast-moving category where competitors, prospects, partners and investors announce their moves in public, mostly on LinkedIn.

## The problem

Competitors post constantly: launches, partnerships, hires, funding rounds, conference appearances. The information is public, but it is buried in volume and has no structure. Reading it by hand is a job nobody had time for, so it happened unevenly, and a competitor's move was often spotted only after the sales conversation where it would have mattered.

Volume was only half of it. A single post rarely matters on its own. What matters is the same person turning up at three competitors, or a prospect partnering with a competitor, and you can't see that reading a feed one post at a time.

## What we built

A platform that monitors a watchlist of companies across six channels daily (LinkedIn, X, Reddit, job postings, Telegram and website changes). An LLM extracts the entities and relationships in each signal and classifies them as competitor, prospect, partner, investor or ecosystem, and everything is stored as a knowledge graph. The team gets an alert feed with sentiment and threat scores, an interactive relationship graph, product-level briefings, and can ask the graph questions in plain English.

It also finds warm paths. Given a target, it shows who on the team has a route in, ordered from a direct connector down to a cold angle.

## Key decision 1: choose the model with a test

The scoring pipeline needed a model. The obvious assumption was that the cheap model would under-score and miss real threats, so we should pay for the expensive one.

I ran an A/B test instead of going with that instinct: 25 real alerts, the same prompt, both models. It cost four cents.

The assumption was wrong. The cheap model was over-scoring, and correctly: it upgraded genuine competitors the expensive model had been too cautious about, and it downgraded noise the expensive model had rated too high. Sentiment agreement was 84%. On the dimensions that mattered the cheap model was arguably the better judge, and it was roughly 87% cheaper to run.

We shipped the cheap model across every scoring pipeline. A four-cent experiment beat a confident opinion, and the confident opinion was mine.

## Key decision 2: reliability over cost when the data goes stale

The batch API is meaningfully cheaper than sequential calls. We tried it and reverted.

Batch jobs have no completion guarantee. They can take up to 24 hours, and when polling timed out we lost whole days of intelligence. For a product whose value is telling you what changed today, a day-late result is worthless.

So we went back to sequential calls, with a hard cost cap per run and a delay between calls. It costs more per run and it works.

## Key decision 3: never write unverified data to the database

We learned this one the hard way. Research from an AI agent was treated as fact and written straight to the database, and dozens of fabricated LinkedIn URLs ended up in production data.

The rule I now apply everywhere: agent research is a hypothesis, and a live API response is the truth. Nothing gets written from the first. Entity extraction also picks up junk in predictable ways, so it is filtered at three points: at extraction, at classification, and by a blocklist in the database itself.

A GTM tool that quietly holds wrong data is worse than no tool, because people act on it.

## A prompt lesson worth keeping

Language models take on the tone of what they read. A competitor's triumphant launch post scores as positive unless the prompt tells the model to judge from your side. A great day for them is a bad day for you, and that had to be written into the prompt with examples.

## Architecture

```
Watchlist of companies
   -> 6 daily pipelines: LinkedIn, X, Reddit,
      job postings, Telegram, website changes
   -> LLM extraction: entities, relationships, sentiment,
      strategic classification, threat + priority scoring
   -> Supabase (Postgres) knowledge graph
   -> Dashboard: alert feed, entity tracking,
      interactive relationship graph, warm-path discovery,
      natural-language queries over the graph, AI summaries
```

Next.js, Supabase, Claude API with prompt caching and per-run cost caps, n8n for the data pipelines, Cytoscape for the graph.

## The tradeoff I accepted

The system hides most of what it collects, on purpose. A team that gets everything reads nothing. I'd rather it sometimes drop something a completionist would keep and stay a feed the team actually opens.

## Outcome

Tracking competitors went from an occasional manual task to a feed the team relies on, and the relationships between players are now visible instead of left to memory.
