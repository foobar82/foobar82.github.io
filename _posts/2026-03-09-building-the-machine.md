# Building the Machine That Builds the Machine

Last time, I left you with a barely-functional ecosystem simulator and a promise: the bugs and rough edges would be the agents' first job, not mine.

I've spent the last week resisting every instinct to polish, and instead building the infrastructure that will let AI agents do the polishing for me. Here's what that looked like.

## First: A Pipeline, Not a Product

My natural impulse was to fix the obvious problems. The simulation visually worked but had a rounding error that meant nothing actually moved: a divide-and-round-down that returned zero on every tick. Creatures spawned and just... sat there. Very zen. Not very ecosystem.

I wanted to fix it, but fixing it myself would defeat the point. Instead, I needed to build the path that *agents* would use to fix it: a CI pipeline. Without this, "autonomous deployment" is just "committing to main and hoping."

The pipeline runs in sequence: lint the frontend, type-check, lint the backend, run the tests, build. If any step fails, everything stops. Simple, boring, exactly what you want between an AI agent and your production server.

## Tests That Survive Evolution

Here's the interesting testing problem: the app is *designed* to change unpredictably. Users will request new species, new behaviours, new terrain features. Traditional behaviour-driven tests would be constantly broken by the very evolution the system is built to enable.

So I split the test suite in two.

**Essential tests** are human-maintained and untouchable by agents. They validate invariants: things that must always be true regardless of how the app evolves. Does the simulation tick at roughly the expected rate? Does the app render without crashing? Can you submit feedback and get a reference number back? These are the guardrails.

**Everything else** is fair game. Agents can add tests, modify tests, even delete tests, as long as the essential suite still passes. This maps to a concept I'm calling the "contract file": a static document that defines what the app *is* and what the agents must never violate. The feedback system must exist. The terms of service must remain intact. No offensive content. No executing user-submitted code. Agents read it before every task and reviewers check against it.

The tick rate bug, incidentally, became the validation test for the whole approach. I wrote a test that catches it, then deliberately reintroduced the bug to confirm. Satisfying.

## The Agent Squad

With the pipeline in place, I built seven agents, each with the same interface: take an input, produce an output, report what you did. The orchestrator doesn't care what's inside each one; it just calls them in sequence.

**At submission time** (when a user types feedback), two things happen locally. A filter agent running on Ollama checks whether the request is safe, and an embedding agent stores the request in ChromaDB for later clustering. Both run on a local 8B model, so they're essentially free.

**At batch time** (once daily), the expensive work happens. A cluster agent groups similar requests. A prioritiser picks the most popular cluster. A writer agent — this one calls Claude via the API — generates the code changes. A reviewer agent (also Claude, different system prompt) checks the work. If approved, a deployer agent creates a branch, commits, runs the full CI pipeline, and merges to main.

If the reviewer rejects the work, the writer gets the feedback and tries again. Two retries, then the request rolls over to tomorrow; no infinite loops here.

## Two Machines, One Tunnel

The development setup turned into its own small adventure. I develop on a Windows desktop but the pipeline runs on a MacBook that serves as both the production server and the Ollama host. Rather than installing Ollama on both machines or maintaining mock modes, I SSH-tunnelled from Windows to the MacBook. One command forwards port 11434, and now my development environment is hitting the same local models that production uses.

It's simple, it's reliable, and it means I'm always testing against the real thing. Development stops when the MacBook is off, but for a side project, that's fine.

## Swappable Parts

The bit I'm most pleased with is the modularity. Every agent is a plugin. The registry maps step names to implementations, and swapping one means changing a single line.

I used this almost immediately. Before spending API credits on the first real batch, I wanted to validate the infrastructure: does the deployer actually create branches correctly? Does the budget tracker work? Do submission statuses update properly? So I built local-model variants of the writer and reviewer agents. Same interface, same prompts, but calling the 8B Ollama model instead of Claude.

The code quality from a local model is, predictably, not great. But that's not the point. The point is: does the *pipeline* work? Does a bad code change get correctly committed, tested, rejected, and rolled back? It does.

This also validated the design pattern itself. The first real test of "agents are swappable" was swapping the agents.

## Budget by Design

A quick note on cost control, because it's easy to overlook with API-based agents. I built budget tracking into the pipeline from day one: token usage per call, converted to approximate cost, accumulated daily and weekly. Hard caps at £2 per day and £8 per week. If a batch would exceed the remaining budget, it pauses and rolls the work over.

This isn't just financial prudence; it's a safety mechanism. A runaway agent loop that keeps retrying and burning tokens is a real failure mode. The budget cap is an automatic circuit-breaker.

## Dry Run, Then Real

Before the first real batch, I added a `--dry-run` mode. This handles the full pipeline flow, with real Ollama calls for filtering and clustering, but mocked API calls for writing and reviewing. It caught several configuration issues that would have burned credits and produced nothing useful.

Then I ran it for real.

The first run failed: the timeout on my writer was too short for the task. Fixed that.

The second run also failed. The local model was not large enough to handle the task; it hallucinated itself a task, and it reviewed the codebase instead. 

I'll save the details of the first successful batch for the next post, because it deserves its own write-up: including which parts of my pipeline worked well, and which worked badly. Here's a teaser of something that *did* work well: blatant phishing attempts were prevented.
![The filter agent stopped me being evil](/assets/Feedback-Request-Handling.jpeg)

I built the thing that's building the thing. And I'm never entirely sure what it's going to do.

## What's Next

The loop is closed. Month one's minimum success criterion for my experiment (at least one user-submitted change refined and deployed by agents) is achieved. The questions now shift: how well does it work over days and weeks? What happens when real users submit requests I haven't anticipated? How do the agents handle conflicting feedback?

Next up: automating the daily batch, adding the emergency brake, and — the scary part — opening the doors to other people.

*The Lost World Plateau is live. If you want to submit a request and watch what happens, I'll share the link in my next post.*
