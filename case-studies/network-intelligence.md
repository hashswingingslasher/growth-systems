# Network Intelligence

**Context:** A consulting firm whose business development runs on the team's LinkedIn networks. Each person has thousands of connections, but they sit in separate accounts as CSV exports with no shared view.

## The problem

The team's combined network was its most useful asset, and nobody could see it. There was no way to tell which organisations the team had the strongest access to as a group, or where two people both knew someone at the same target. Finding a warm intro at company X meant asking around and hoping someone remembered. With thousands of connections per person, cross-referencing by hand wasn't possible.

## The constraint

LinkedIn CSV exports are messy: a byte-order mark, junk header rows, and dates in a `DD-MMM-YY` format nothing else uses. Company names are free text, so the same organisation appears as "Goldman Sachs", "Goldman Sachs & Co" and "GS". Rank organisations before fixing that and one real company splits into five weak ones.

## The key decision

Most of the work went into entity resolution, not the dashboard. Company names go through a three-tier normalisation before anything else runs:

1. Exact match
2. Suffix strip (drop "Ltd", "& Co", "Inc")
3. Fuzzy match (fuse.js) for the rest

Every ranking and overlap count downstream depends on that step being right, so that's where I spent the effort.

## Architecture

```
LinkedIn CSV exports (multiple team members)
   -> Ingestion (PapaParse, handles BOM / junk rows / odd dates)
   -> Company normalisation (exact -> suffix-strip -> fuzzy)
   -> Contact classification (seniority tier, sector)
   -> Overlap detection (shared connections across the team)
   -> Dashboard: organisations ranked by connection density,
      senior decision-maker access, and team overlap
   -> Branded .xlsx export
```

Next.js 15, Supabase with row-level security, deployed on Vercel.

## The tradeoff I accepted

The data is only as fresh as each person's last export, and the system says so. It is built for periodic re-uploads and shows how old each person's data is. I also tuned the fuzzy matcher towards precision, because merging two different companies is a worse mistake than leaving one unmatched, and kept the output reviewable by a person.

## Outcome

The team's combined network is now one searchable view. It shows which target organisations the team already has warm access to and where two members share a connection, and those shared connections are where the warm intros come from.
