---
layout: post
title: "How Do You Keep Independent Agents Doing the Right Thing?"
date: 2026-03-25
---

# How Do You Keep Independent Agents Doing the Right Thing?

The Lost World uses agents to modify the ecosystem simulation. Right now it's basic: three species, very simple behaviour. I have some essential tests — "feedback pane is always visible," "renders without crashing," "organisms stay within the canvas." I also have tests for *current* behaviour, like "species don't go extinct."

The essential tests don't change. The behaviour tests may change — for example, if I decide I *do* want to simulate extinctions.

But what if the feature requests we implement change the fundamentals? We move the feedback pane to a different view. We switch to an infinite canvas. The essential tests become a brake: agents keep failing, retrying, and failing again. I can only find out why by reading the log files, and even then, only when I notice the mounting error count.

## The Naive Solutions (and Why They Don't Work)

There are some obvious approaches. There are, fortunately, obvious reasons they fail:

**Let agents modify all the tests.** This defeats the point of protecting them. It doesn't just allow scope drift: it allows scope *movement*, wholesale and unchecked.

**Let agents skip tests.** Like modifying tests, but with added tech debt. Choose this if you... no, just don't choose this.

**Have a human review every test failure.** Doesn't scale, and defeats the point of my experiment in autonomy. I'd stop checking after day three.

No. It's time for some thinking.

## What Do Test Failures Actually Mean?

One task failing a test is probably bad code. Three unrelated tasks - generating completely different code from completely different user requests - all failing on the same test? The common denominator is the test.

That's not a guarantee the test is wrong or outdated. But it is a strong signal that something's awry, and it tells you *where*.

I can see different scenarios: the test is genuinely outdated and needs updating; the agents have a correlated failure mode; or the tech debt has built to a point where there is no winning move. Right now, it takes a human to distinguish between these. Autonomy here is something I could explore later.

## The Solution

That's the problem statement and the solution shape. The implementation follows three principles:

**Zero-cost.** Detection is pure regex and file I/O - no LLM calls, no spend.

**Write-only.** Agents can raise a concern but never act on it.

**Human-on-the-loop.** I review, I decide.

When the system detects that three or more independent tasks have all been blocked by the same protected test, it writes a structured proposal to a `concerns.md` file, working as a ring-fenced communications channel. The proposal includes the evidence: which test, which tasks, what the error was, how many times it's happened.

## Citizens, Not Rulers

This works like a constitutional arrangement: agents propose amendments they can't ratify. They are citizens, not legislators. The petitions they raise must be assessed, but should never be auto-approved.

There's a design principle worth extrapolating here. I've seen people writing about self-improving agent loops that run nightly improvement cycles: take agents' self-reported problems, turn them into specs, then PRs. I could run the same architecture as my main pipeline on this. That gives me three layers:

1. **The plateau** — the application, what the user sees.
2. **The pipeline** — the agents that implement user feedback and maintain the plateau.
3. **The meta-pipeline** — agents that improve the pipeline itself. And, eventually, the meta-pipeline too.

This is very much a case of deciding how far to build and whether now is the right time. Doing this first would have been over-engineering. Given the low volumes I'm currently handling, I'm arguably over-engineering *already*. I'm going to park the meta-pipeline idea for now.

Is it premature optimisation? I'll find out.
