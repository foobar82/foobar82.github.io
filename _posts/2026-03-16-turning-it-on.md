---
layout: post
title: "Turning It On"
date: 2026-03-16
---
# Turning It On

Last week I built the machine. This week I turned it on.
![I wish I had a lever like this for it.](/assets/Turn-It-On.jpg)

The first question was whether the local pipeline — filtering, embedding, clustering, prioritising — could produce output good enough for the writer agent to work with. I tuned each stage until it was *good enough*, not great. If the writer can understand the task summary, the upstream stages have done their job. I wanted to perfect it; that would have been perfectionism, not value.

## The Local Model Hits Its Ceiling

I tried the local 8B model as the writer first, partly as an infrastructure smoke test and partly out of curiosity. It couldn't do it. The codebase context filled so much of its context window that by the time it reached the actual task, it had forgotten what it was supposed to do. Instead of writing code, it produced a code review of the files it had been given. Confidently, thoroughly, and completely beside the point.

This is a useful data point. Local models are great for cheap, focused tasks: "is this request safe?" is a yes/no classification that fits comfortably in a small context window. "Read this codebase and produce a coherent set of changes" is fundamentally a different task. The 8B model validated the plumbing but confirmed the architectural decision: code writing needs an API model (or at least, one that needs much punchier hardware than I had available!)

## First Contact

The first real run used Claude Sonnet 4.5 for both writer and reviewer. The feedback request sitting at the top of the queue was one I'd submitted myself:

*"It's hard to see the plants — green on a green background. Please make the background grey instead."*

Here's what the writer produced:

*Changed plant colour from green to golden yellow and added dark outline for better visibility.*

It didn't do what I asked. It did something better — or at least, it thought so.

I asked for a grey background. The writer understood that my actual problem was plant visibility, and solved it at the source: change the plants, not the background. It picked a dark golden yellow that contrasts against the green plateau, and added outlines for good measure. The plants are now visible: but the ecosystem looks a bit weird, unless you assume that the background is grass that the herbivores don't eat.

The reviewer approved it, noting that the colour choice was appropriate for vegetation and the implementation followed existing code patterns. A clean, minimal change that preserved the aesthetic while solving the problem. It didn't register that most plants, for most of the year, are green rather than yellow!

This is the most interesting result so far. The agent didn't follow my instruction literally — it interpreted the intent. Whether that's a feature or a bug depends on your perspective. For a project where learning is the main objective, I'd call it a feature. For an ecosystem simulator where aesthetics matter, it's borderline. For a banking application, I'd put this solidly on the "bug" side.

## The Boring Problems

The actual deployment didn't go smoothly on the first attempt, and the problems were entirely mundane. My environment setup scripts hadn't installed all the dependencies. One of the pipeline tests was contaminated: an earlier test wasn't tearing down properly, leaving state that caused a later test to fail. Nothing to do with the agents; everything to do with my own infrastructure.

I fixed the dependency issue, sorted out the test isolation, and added a "redeploy" flag so I could retry a batch without re-running the expensive API calls. Second attempt: the pipeline ran clean. Branch created, changes applied, tests passed, merged to main, deployed.

## The Bill

I'd estimated 50-60p per stage for the writer and reviewer, so roughly £1-1.20 per batch. The actual total spend for the full run — writer, reviewer, everything — was 42p. Well within the daily £2 cap and comfortably under estimates.

At that rate, I could run two or three batches a day and still stay within budget. The cost tracking is working, and the numbers suggest I have more headroom than I expected at this scale.

## Why I'm Launching Now

My original plan says Month 3 is "go public." It's the end of Month 1. I'm launching anyway.

The whole point of this project is evolving from user feedback. Right now, the only user is me, and I already know what I'd ask for. If I keep it private, it's tempting to cheat and use Claude Code, not my own pipeline, to run the changes: it's been built over a year by a dedicated team, not a month by a single engineer in his spare time. Real users will submit requests I haven't anticipated, stress the agents in ways I can't predict, and generate the experimental observations I actually need.

There are gaps. The observability metrics aren't automated. The multi-model reviewer isn't in place yet. But the essentials are covered: rate limiting is in, terms of service and privacy policy are visible, and I've built a one-command rollback mechanism that undoes the latest agent change if something goes wrong. Good enough to open the door.

My desire for perfection before exposing the system to real input is exactly the wrong instinct for this experiment. I'm the meta-user; I'll iterate on the pipeline the same way the pipeline iterates on the app.

Better to learn fast than to launch polished.

## What's Next

The app is live! [LINK]

Submit a request. See what happens. The agents run a daily batch, so check back tomorrow to see if your suggestion made it onto the plateau.

I'll be writing up what real users ask for, how the agents handle it, and what breaks. The plateau is waiting to evolve.
