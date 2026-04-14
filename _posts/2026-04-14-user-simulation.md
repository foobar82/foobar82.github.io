---
layout: post
title: "User testing without users?"
date: 2026-04-14
---


# When you have no users, simulate them

The Lost World is a toy project — a 2D ecosystem that evolves based on feedback, built entirely for the pleasure of building it. There's no commercial ambition here, no roadmap to monetisation. But it does have a pipeline that needs feeding.

That pipeline takes user feedback ("I want more predators", "the herbivores all cluster in the top-left corner") and turns it into ecosystem changes through a chain of six agents. Without that input, the agents have nothing to chew on. The loop is closed but idle.

It's worth being clear: in a commercial product, an obvious piece of the answer is to get real users, and collect real feedback. If you can't get real users, you don't have a product.

Synthetic simulation alone is a poor substitute for user feedback: but this is a toy project with no public users yet, and I wanted to keep the pipeline moving while I build toward a proper launch. So I built a User Emulation Agent instead. *note: Claude helped draft this blog, and came up with this Portentous Capitalisation. It fits my sense of humour.*

The emulator is a standalone script that reads the current state of the app and generates plausible feedback as a defined user persona would write it. It drops that feedback into the queue exactly as a real submission would. The rest of the pipeline never knows the difference, but the good thing is it doesn't need to.

This solves my immediate cold-start problem (and please, drop some feedback requests in!). But the more interesting thing is the pattern it represents: **role-playing agents as a mechanism for synthetic data and test input generation**. This same idea has transferrable elements for red-teaming (agent simulates an adversary), multi-agent debate (agents argue opposite positions), and evaluation chains where an agent acts as a user to score UX quality. 

---

## How the agent thinks about itself

One design decision I'm looking forward to A/B testing: the agent doesn't pretend to be human. Instead of putting it in character and hoping it doesn't confuse its own perspective with a person's, the prompt frames it explicitly:

> "I am an agent and I believe a user would notice X, causing them to do Z, given they have characteristics Y."

This keeps the model grounded while still letting it reason across a range of personas.

Variant prompt to test next time:

> "I am a user with characteristics Y, and I notice X, causing me to do Z."


---

## The empathy gap

Agents aren't human. They have no lived experience, no true memory, and no human emotion (despite the interesting papers on emotion vectors!). Therefore, my user emulation agent has the strengths and weaknesses of an LLM: it pattern-matches on what has ever been *articulated and written*. Everything ever written about what people find confusing or delightful. Its systematic blind spot is **silent abandonment**: the user who closes the tab without saying why. That behaviour is invisible to the training corpus, and therefore to the agent. 

*Digression: this "negative space" principle is hugely important in graphic design, garden design, and visual communication. We don't often talk (or write) about what's not there: is this something that needs to be addressed in order for GenAI to tackle tasks in these arenas?*

My mitigation: the prompt explicitly asks the agent to reason about what *would* cause disengagement, not only what would prompt an explicit request. This doesn't solve the problem (it can't) but it at least keeps the question in view.

There's a secondary quirk too. The emulator generates more precise, articulate problem statements than most real users would write. For now, that's a net positive: it gives better-specified input for the writer agent. The mild long-term risk is that if I tune my pipeline for clean simulator inputs, it might underperform on vague or poorly-articulated real feedback when actual users eventually arrive. This is acceptable for v0; I'm fully leaning in to the "ship-and-iterate" philosophy on this.

---

## What the simulator sees

The agent is deliberately constrained to what a real user could observe:

- The simulation source files (what entities exist, their behaviours, rendering) <-- these are available because it's an open-source project, but how many users would actually take this step? It's supposed to be a toy, not a code study!
- A short recent git log (what's changed lately) <-- ditto, but to a lesser extent. Also, these are written by my relatively crude agent pipeline: are they any good yet?
- The last 10 completed feedback items (what's already been built) <-- well, this one's true for sure.

It doesn't see backend code, pipeline logs, or test output. Those are developer artefacts; a user would have no access to them.

Importantly, it doesn't yet look at the system running. It doesn't consider what a real user could observe, play with, interact with, etc. It cannot yet see the pain of green on a green background. It sees colour hexcode constants instead.

---

## Persona harness

Each simulated user type is a `Persona` dataclass with a name, a plain-English description (injected verbatim into the prompt), a technical level, and an engagement style. The v0 default is `curious_explorer`:

> A non-technical user who finds the ecosystem visually interesting and wants to watch it grow and change in surprising ways. Tends to notice when something looks odd or static, and imagines what might make the world feel more alive.

Adding a new persona is a single dict entry — no other code changes needed. Future candidates I've noted down: confused newcomer, bored power user, aesthetic critic, adversarial tester.

---

## What it actually produces

---

Submitted (3):
  • the little animals keep running around but they don't seem to be doing anything interesting - would love to see them actually interact with each other more
  • sometimes everything just disappears and then suddenly pops back - feels jarring when the whole world resets like that
  • the water looks pretty but nothing ever happens near it - what if some creatures needed to drink or lived in it?

Reasoning:
  I considered what would catch a visually-oriented, non-technical user's attention. They'd notice the movement patterns seem random rather than purposeful, which breaks immersion. The respawn mechanic when species go extinct would feel abrupt and artificial to someone expecting natural flow. The water feature is visually prominent but completely non-functional, which would make a curious user wonder about missed possibilities. These are all 'what if' observations about making the world feel more alive, matching the persona's tendency to imagine improvements rather than specify technical solutions.

---

I like this. The feedback is human in tone (module some dashes - oh Claude!). It's helpful that it specifies what the user observes as a problem ("nothing ever happens near the water"), *and* a potential improvement. The reasoning is helpful too; more results coming next week as I experiment with my prompts and harnesses.

---

## The limit cycle risk

This is the part I'm most cautious about in the long run.

In v0, a human (me) reviews every output before it goes anywhere near the queue. The loop risk is essentially zero. But if this ever runs autonomously with multiple personas, three failure modes become plausible:

- **Amnesia cycles**: the context window doesn't include enough history, so the simulator re-requests something that was already tried and reverted.
- **Oscillation**: two conflicting personas pull the pipeline in opposite directions indefinitely, neither ever winning.
- **Drift blindness**: the simulator can't see deliberately *removed* features and asks for them back.
 
The v0 mitigation is ChromaDB semantic deduplication: before submitting anything, the simulator queries for similarity against recent pending and completed feedback. Anything above the distance threshold gets skipped and noted in the reasoning trace. (I suspect I need to work on this; vector DB distances were huge even for small changes in feedback requests in my earlier exploration, so I may need a different clustering mechanism).

The longer-term fix is something I was already planning for other reasons: a **naturalist's log**. A maintained, human-readable summary of what the ecosystem currently is, what's been tried and removed, and where it's heading. This would replace the raw git log as the simulator's context entirely. I deliberately want to preserve this coupling between the UX changelog and the simulator's memory; it means the simulator is a little more like a real user who can see what's written but doesn't know the full project history.

---

## Two outputs

Every run produces two things:

1. **Synthetic feedback** — formatted identically to a real user submission. Tagged `source: "simulator"` in the database for observability, but the prioritisation logic is blind to that field. Synthetic and real feedback compete on equal terms for implementation tokens.
2. **A reasoning trace** — saved to `pipeline/data/simulator/<timestamp>.json`. Why it generated what it did, what it considered and rejected, what got deduped. Not for the pipeline; for me.

---

## What's next

The obvious follow-on is stateful simulation: a persona that remembers what it asked for in previous runs and notices whether it was fulfilled. That needs the naturalist's log as its memory primitive, so it's wired to the same roadmap item.

Multiple personas per run is another step — three agents generating 1–2 items each, deduplication doing the conflict resolution. And if I ever add a headless browser, the text context can be swapped for a screenshot, which would be much closer to the actual user experience.

For now, the simulator runs manually when I want to kick the pipeline into motion. That's enough.

---

*Code for this project lives at [foobar82.github.io](https://foobar82.github.io). Previous posts in this series cover the six-agent pipeline, ChromaDB integration, and the constitutional amendment process.*
