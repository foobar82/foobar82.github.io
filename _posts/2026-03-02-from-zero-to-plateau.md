# From Zero to Plateau: Building v0 of The Lost World

Last week I laid out the concept: a self-evolving ecosystem simulator, built and maintained by AI agents. This week, I rolled up my sleeves and built the first version.

## The Setup

With the planning done and a clean environment prepared, the goal for the week was straightforward: go from zero to a running ecosystem simulator. Plants, herbivores, predators, a water source, and a feedback box. Nothing fancy; just enough to be visibly alive.

I had a full scope design from my planning sessions with Claude, and a week 2 implementation plan covering the frontend, backend, feedback UI, and deployment. I knew I was going to hand this to an AI coding agent; the question was: what's the best way?

## Three Approaches, Side by Side

I ran three implementations in parallel, each on its own git branch:

**One-shot:** "Here's the full context. Build everything."

**Self-decomposed:** "Here's the full context. Make your own plan, then execute it."

**Pre-decomposed:** I asked Claude to break the work into six phases with specific prompts, then fed those to Claude Code one at a time.

I expected the self-decomposed approach to win. Claude Code would have the full context, could plan according to its own strengths, and wouldn't be constrained by my assumptions about how to break the work down.

I was wrong.

## The Surprise

The pre-decomposed approach, where Claude created the plan and I handed the prompts to Claude Code, produced the best result. It took me a moment to work out why.

I'd asked Claude to create that plan in the same conversation where I'd done all the ideation and architectural design. The context window was polluted from my previous conversation: why I'd made certain decisions, what trade-offs I'd considered, what the ecosystem was *for*. That context bled into the plan, and from there into the implementation.

The other two approaches only had the planning documents, giving them the *what*, but not the *why*.

While context pollution won't always work in my favour, this time it did.

## Distraction And Temptation

My first instinct was to run a fourth trial: decompose the plan in a fresh Claude session *without* the accumulated context, and see if the advantage disappeared. That would give me a cleaner comparison.

I decided not to. "What's the optimal way to prompt a coding agent?" isn't one of my core research questions for this project. It's adjacent, but pursuing it now would delay the actual build. I noted it as a future experiment and moved on. Importantly, I still have the inputs and outputs saved in git branches. When I come to trial other models and tools — Cursor, the GPT models, and so on — I have a benchmark point to build from.

## Evaluating the Results

I assessed the three implementations two ways.

First, I deployed all three and tested them manually. I was looking for: which one matched the intended scope best (basic but not overpolished), ease of deployment, ease of running, and suitability of what was actually produced.

Second, I ran AI code reviews. Claude Code with Opus 4.6, and then separately Cursor with GPT 5.2, each given the same evaluation criteria:

| Criterion | What I was looking for |
|---|---|
| Extensibility | Is the architecture ready for agents to modify it later? |
| Simplicity | Just enough complexity to be sensible — no over-engineering |
| Scope match | Does it build what was asked for, without gold-plating? |
| Critical bugs | Anything that would prevent basic operation? |

| Criterion | Claude Code assessment | Cursor assessment |
|---|---|---|
| Extensibility | Decomposed first (both tied), one-shot second | Self-decomposed first, Pre-decomposed second, one-shot third |
| Simplicity | Pre-decomposed first, one-shot second, Self-decomposed third | Pre-decomposed first, one-shot second, Self-decomposed third |
| Testability | One-shot first, Pre-decomposed second, Self-decomposed third | One-shot first, Pre-decomposed second, Self-decomposed third |
| Critical bugs | Pre-decomposed first, Self-decomposed second, one-shot third | Pre-decomposed first, Self-decomposed second, one-shot third |

All three evaluation methods — my manual testing, Claude's review, and GPT's review — agreed: the pre-decomposed approach, with its polluted context window, produced the strongest implementation overall. This is the one I merged as my baseline.

## What This Means

The practical takeaway is simple: **where you can afford to keep working in the same context window, do so.** Distilling your thinking into a document and starting fresh loses signal. The reasoning behind decisions matters as much as the decisions themselves.

This has already changed how I use Claude. I now work in deliberate branches:

- One long-running conversation for ideation and planning
- A second for implementation planning, seeded from the first to give Claude as much *relevant* context as I can
- Short-lived conversations for isolated questions

It's a small workflow change, but it keeps the right level of context for the task at hand, and it minimises the performance drop-off from conversation compression or from over-full context windows.

## The Meta-Learning

I went into this week expecting to learn about AI code generation. I came out having learned three things I didn't anticipate:

1. **Context pollution can be an asset.** Accumulated reasoning in a context window improves downstream outputs, even when you think you've captured everything in documentation.
2. **Workflow design matters.** How you structure your conversations with AI — branches, continuity, isolation — directly affects output quality.
3. **You can plan all you want.** The real learning starts when you build.

None of these were in my experiment design. That's the point.

## What's Next

The simulator is running. It's rough, it has known bugs, and there are obvious improvements to make. My temptation is to polish it before anyone sees it: but that's exactly what the agents are for!

Next week: building the CI/CD pipeline and test harness, so the agents have a safe path from code to deployment. The bugs and rough edges? They'll be the agents' first job.

![The earliest launched version of my ecosystem simulator](/assets/Lost-World-Screenshot-1.png)

Repo for the curious: https://github.com/foobar82/the-lost-world/
