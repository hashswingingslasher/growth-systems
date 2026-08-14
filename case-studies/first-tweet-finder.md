# First Tweet Finder

**Context:** A personal project. A Chrome extension that jumps to the oldest post on any X (Twitter) profile.

## The problem

Someone's first post is useful context. It is where the origin story is, and in a sales or research setting it is a genuine conversation starter that nobody else bothers to find. But X loads newest first and paginates by infinite scroll, so reaching the bottom means scrolling by hand for minutes. Nobody does it, so the signal goes unused.

Small problem, but a real one, and the kind of thing a five-file tool should just solve.

## The key decision

The hard part is that there is no "go to the end" in an infinite-scroll feed. You cannot ask the page for the last item, because the last item does not exist until you have scrolled it into being.

So the logic is a bounded scroll loop, not an open-ended one. Scroll to the current bottom, wait for new content to load, and watch the page height. When the height stops growing, you have hit the real end and can stop. A hard cap on attempts protects against a profile that never bottoms out, so the tool always terminates instead of hanging.

```
loop up to 50 times:
  scroll to bottom
  wait for load
  if page height did not change: stop   // reached the true end
highlight the last tweet, scroll it into view
```

The wait is the whole trick. Scroll without waiting and you outrun the network, read a stale height, and quit early on the second screen. Waiting for the load is what makes "height stopped changing" actually mean "the end."

## The tradeoff I accepted

A fixed wait and a 50-attempt cap over something cleverer. I could have watched network requests to know precisely when loading finished. For a small tool, a fixed pause plus a hard cap is simpler, has no failure modes worth the extra code, and always terminates. Right-sized engineering beats clever engineering when the problem is small.

## Outcome

Shipped and functional. Manifest V3, one content script, a scroll loop, and a highlight. It does one thing, it does it without hanging, and it turns a few minutes of manual scrolling into one click. Not every build needs to be a platform. Some just need to work.
