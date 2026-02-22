I have, so far, been _using_ AI.

One of the things that's been a revelation to me is not how much *easier* it is to write software, but how much *more fun* it has become. I can start with a concept, spin something up very quickly to explore, and iterate. I can think deeply in order to build a robust system, with tests and documentation.

I have been _building with_ AI. I've built several toy applications and small tools, plus a class booking website.

Users, customers, and so on, all have enhancements they want. When did you last encounter perfect software? Many systems have feedback mechanisms — whether that's per-response like ChatGPT/Claude Chat, or hidden deep in the "contact us" section, it usually exists.

What if the software could improve itself?

I've spent this week designing the system. Let's think about the cycle:

We start with a tool or toy that users can explore. They submit feedback — so I needed to define how. The system — not a human! — then decides which feedback to implement, in which order. A team of agents then define these changes, make them, test and review them, and deploy them. Apart from a human emergency brake, it'll be fully autonomous. I'm moving beyond "AI helps me code" to "AI owns this".

I also needed to build the infrastructure behind this cycle.

What does success look like? From my perspective, it's about learning hands-on how to build and orchestrate agents, and hopefully building something people enjoy playing with! For the system itself, my "minimum success" is at least one user-submitted change being refined and implemented.

I also need ground rules and safety rails. What does the tool do? What does it never do (e.g. generate spam or offensive content)? How do I ensure the agents can't accidentally evolve these principles, for example due to prompt injection? I'm cautious here. I'm going to add an emergency brake and a rollback mechanism, in case costs spiral or a change is a complete disaster — or even if user feedback says "actually, we don't like that change". And, from a purely practical perspective, how do I keep costs sensible, manage licensing and legality, and so on? I'll be hosting this on a dedicated machine I have spare, keeping infrastructure costs to essentially zero.

[PLACEHOLDER: Paragraph referencing the Stripe engineering blog on blueprints — their pattern of mixing deterministic and agentic steps maps closely to this architecture, and validates the approach at production scale.]

Over the next weeks, I'm going to build this. The tool itself — the starting point — isn't the core purpose of what I'm building. It's the showcase for what I'm building. It's a self-evolving system, and that made me start thinking of ecosystems. The humour here — simulating evolution, by evolving — tickled me.

I'm calling it The Lost World Plateau. It'll start very basic: a bounded ecosystem with plants, herbivores, and a predator. It'll evolve over time, as the software evolves following user feedback.

My agents will review and group user feedback. They'll decide which ones to build and deploy. They'll write the code, review each other's work, and sanity-check the deployed system.

But first, I need to build the simulator v0. I've iterated through a scope and spec with Claude. I have three ways to build this: one-shot prompt; decompose the prompt in Claude Code; or decompose the prompt in Claude and hand stage prompts to Claude Code. I have a hypothesis for which of these will work best (decomposing in Claude Code, not general-purpose Claude). When I'm back at my desk I'll run the experiment...
