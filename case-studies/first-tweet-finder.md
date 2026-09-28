# First Tweet Finder

**Context:** A personal project. A Chrome extension that jumps to the oldest post on any X (Twitter) profile.

## The problem

Someone's first post is useful context. It often holds their origin story, and in sales or research it makes a good conversation starter that few people bother to find. X loads newest first and uses infinite scroll, so reaching the bottom means minutes of scrolling by hand. Almost nobody does it.

It's a small problem, and a five-file tool should be enough to solve it.

## The key decision

An infinite-scroll feed has no "go to the end". You can't ask the page for the last item, because it doesn't exist until you have scrolled far enough to load it.

So the logic is a bounded scroll loop. Scroll to the current bottom, wait for new content to load, and watch the page height. When the height stops growing, you have reached the end. A hard cap on attempts covers a profile that never bottoms out, so the tool always stops.

```
loop up to 50 times:
  scroll to bottom
  wait for load
  if page height did not change: stop   // reached the true end
highlight the last tweet, scroll it into view
```

The wait is what makes it work. Scroll without waiting and you outrun the network, read a stale height and stop on the second screen. Waiting for the load means an unchanged height really does mean the end.

## The tradeoff I accepted

A fixed wait and a 50-attempt cap instead of something cleverer. I could have watched network requests to know exactly when loading finished. For a tool this small, a fixed pause and a hard cap are simpler and always terminate.

## Outcome

Shipped and working: Manifest V3, one content script, a scroll loop and a highlight. It does one thing without hanging and turns minutes of scrolling into one click.
