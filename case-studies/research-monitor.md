# Research Monitor

**Context:** A privacy-technology company in a fast-moving, research-heavy category. The team needs to keep up with industry news and market research without drowning in feeds.

## The problem

One request covered two needs. The first was a monitor: watch the industry and surface only what matters. The second was research on demand: answer "what is the state of this market" quickly and with sources. RSS feeds do neither on their own. They produce hundreds of items a week, most of them noise, so a raw feed moves the reading problem somewhere else. The team needed fewer items, filtered better.

## The key decision

AI does the relevance judgement, but not the cheap work. The monitor pulls 14 RSS feeds across fintech, privacy, blockchain and security, and filters in stages:

1. **Keyword filter first (free).** A cheap pass that throws out obvious noise before any model runs.
2. **AI relevance scorer second.** Only what survives gets a 0 to 10 score and a priority tag from the model.

Paying for an LLM call to score an article a keyword filter could reject for free adds up quickly, so the ordering keeps cost in line with the amount of real signal. High-priority items are posted to Slack, where the team already works.

## Filtering out fluff

Feeds are full of items that look like news and aren't: award announcements, generic partnership press releases, ceremonial posts. They get through a keyword filter because they contain the right words, and they carry nothing useful. I added a step that detects and removes them before scoring. A monitor that keeps forwarding award posts teaches the team to ignore it.

## Research and drafting

From Slack, a team member can turn a scored item into draft content in one of four team members' voices through a modal. A Slack-triggered research flow also answers a question on demand and returns an executive summary with sources to the channel, with a dashboard showing the summary, key findings, market data, key players and sources. Monitoring, research and drafting all run from the channel the team already uses.

## Architecture

```
Monitor (scheduled):
  14 RSS feeds -> keyword filter (cheap) -> fluffy-content strip
    -> AI relevancy scorer (0-10 + priority) -> high-priority
    -> post to Slack -> optional: draft content in a team voice

On-demand (Slack-triggered):
  Slack question -> webhook -> model -> parsed summary
    -> Slack reply + research dashboard
```

Orchestrated in n8n, with an LLM for scoring and summarisation and Slack as the interface.

## The tradeoff I accepted

The keyword pre-filter can drop a relevant article that doesn't use the keywords. I accepted that risk because a monitor that raises false alarms gets muted, and a muted monitor is worth nothing. So I tuned it tight enough that everything reaching Slack is worth reading.

## Outcome

Industry monitoring runs on a schedule into Slack, and research questions are answered in the same channel with sources. The reading load drops from hundreds of feed items to the handful that matter.
