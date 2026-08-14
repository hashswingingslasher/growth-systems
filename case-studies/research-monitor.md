# Research Monitor

**Context:** A privacy-technology company working in a fast-moving, research-heavy category. The team needs to stay current on industry news and market research without drowning in feeds.

## The problem

Two different needs sat behind one request. First, a passive monitor: watch the industry and surface only what matters. Second, on-demand research: answer "what is the state of this market" quickly, with sources. RSS feeds solve neither on their own. They produce hundreds of items a week, and most of it is noise. A raw feed just moves the reading problem, it does not remove it. The team did not need more information. It needed less, filtered better.

## The key decision

Let AI do the relevance judgment, but do not let it do the cheap work. The monitor pulls 14 RSS feeds across fintech, privacy, blockchain, and security, and filters in stages on purpose:

1. **Keyword filter first (free).** A cheap pass that rejects the obvious noise before any model is involved.
2. **AI relevancy scorer second.** Only what survives gets scored 0-10 by the model, with a priority tag.

Spending an LLM call to score an article a keyword filter could reject for free is waste at scale. The ordering keeps the cost proportional to the signal. What clears the scorer as high priority gets posted to Slack, where the team already works, so nobody has to visit a separate tool to stay informed.

## Killing the fluff

Feeds are full of items that look like news and are not: award announcements, generic partnership press, ceremonial posts. I added explicit fluffy-content detection to strip these before scoring, because they are exactly the items that pass a naive keyword filter (they contain the right words) while carrying zero strategic signal. A monitor that keeps forwarding award posts trains the team to ignore it.

## The second surface: research and drafting

The same system does more than watch. From Slack, a team member can turn a scored item into draft content, generated in one of four team members' voices, through a modal. And a Slack-triggered research flow answers a question on demand, returning an executive summary with sources into the channel, also viewable on a dashboard laying out summary, key findings, market data, key players, and sources. Same engine: monitor, research, and draft, all in the channel the team already lives in.

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

The keyword pre-filter can drop a relevant article that happens not to use the keywords. I took that risk on purpose. A monitor that cries wolf gets muted, and a muted monitor is worth nothing. So I tuned it tight, every item that reaches Slack earns its place. A monitor the team reads beats a complete one they learn to ignore.

## Outcome

Industry monitoring runs on a schedule straight into Slack, and on-demand research is answered in the same channel with sources. The reading load drops from hundreds of feed items to the handful that actually matter.
