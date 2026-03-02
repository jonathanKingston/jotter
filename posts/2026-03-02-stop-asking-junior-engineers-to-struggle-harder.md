---
title: "Stop asking junior engineers to struggle harder"
categories:
  - AI
---

Four pieces landed in the past week that circle the same problem from wildly different altitudes.

Ivan Turkovic's ["AI Made Writing Code Easier. It Made Being an Engineer Harder."](https://www.ivanturkovic.com/2026/02/25/ai-made-writing-code-easier-engineering-harder/) is the systemic view. The baseline moved. Expectations rose without announcement. He names something important: the supervision paradox, where reviewing AI-generated code is often harder than writing it yourself, because you inherit the output without the reasoning.

Daniel at Be a Better Dev takes [the opposite approach](https://www.beabetterdev.com/2026/03/01/ai-is-making-junior-devs-useless/), starting with a title - "AI is Making Junior Devs Useless" - that does his own argument a disservice. The content is more thoughtful than the headline: manufacture the struggle, never ship code you can't defend, prompt for the why not the answer. But the framing tells juniors they're the problem before the first paragraph starts.

Geoffrey Huntley, in ["Software development now costs less than the wage of a minimum wage worker"](https://ghuntley.com/real/), pushes the economic argument furthest. The economics have collapsed. The org charts are flattening. Model-first companies eating incumbents on margin, the cycle compressing.

And the ACM paper ["Redefining the Software Engineering Profession for AI"](https://dl.acm.org/doi/10.1145/3779312) gives this a precise name: **seniority-biased technological change**. AI amplifies engineers who already possess systems judgement while imposing drag on early-in-career developers who lack the context to steer, verify, and integrate AI output.

<img class="image-framed" src="/images/ai/junior-four-views-blind-spot.jpg" alt="Retro propaganda poster showing four suited figures on pillars looking outward through telescopes while a small worker stands unseen in the gap below.">

Four perspectives. One gap: none of them land on a structural fix for the people at the bottom of the experience ladder.

## The willpower problem

<img class="basic-alignment right flex-content" src="/images/ai/junior-struggle-harder.jpg" alt="Propaganda poster of a suited figure giving a thumbs up holding a Struggle Harder sign while a worker is overwhelmed by AI velocity machinery behind them.">

Daniel's advice is good advice. It fails under real conditions.

"Manufacture the struggle" assumes a junior engineer has the autonomy to slow down when their team is shipping at AI-accelerated pace. It assumes their manager measures learning, not velocity. It assumes peer pressure from teammates who *are* pasting everything into Claude won't override good intentions within a week.

Huntley's economics make this worse. When software development costs less than minimum wage, the pressure to produce at AI-accelerated rates isn't a cultural problem you can resist with discipline. It's a market force. The space Daniel is asking juniors to carve out for deliberate practice is the first thing that gets squeezed.

The acceleration trap is self-reinforcing: faster output creates higher expectations, higher expectations demand more AI reliance, less time for the deliberate practice Daniel is prescribing.

Telling juniors to struggle harder inside a system that punishes struggling is not a strategy. It's a coping mechanism dressed up as career advice.

## The verification gap

<img class="basic-alignment left flex-content" src="/images/ai/junior-verification-gap.jpg" alt="Split propaganda poster: experienced worker sees bugs and warnings in code while junior sees only LGTM. Caption: what you can't see can ship.">

The ACM paper gives us the frame: AI amplifies engineers who already have systems judgement and drags on those who don't. This isn't a temporary imbalance. It's a property of how the technology works.

Jason Wei's [asymmetry of verification](https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law) - that AI training effectiveness correlates with how easily you can verify the output - applies just as well to AI *usage*. Call it Verifier's Law for the practitioner side: AI is a force multiplier for your ability to verify, not a replacement for it.

The Project Societas example from the ACM paper illustrates this. Seven part-time engineers, 110,000 lines of code, 98% AI-generated. Human work shifted to directing: specifying goals, verifying correctness, integrating output. That's a story about people who already knew what good code looked like. It is not a template for how a junior should work.

A senior reviews AI-generated code and spots the subtle race condition, the missing edge case, the architectural choice that will hurt in six months. A junior reviews the same code and it looks... fine. It compiles. The tests pass. The PR gets approved. The bug ships three sprints later.

We're putting junior engineers in a role - AI output verifier - that requires the exact expertise they haven't built yet. A junior shipping AI output never sees the fifteen minutes a senior spent reasoning before writing the first line, the approaches considered and rejected. They see the output. They ship the output. They never learn the process that makes the output trustworthy.

## Wrong seat, not wrong person

<img class="basic-alignment left flex-content" src="/images/ai/junior-right-seat-right-work.jpg" alt="Retro poster showing five doors: unsupervised AI output crossed out with a warning symbol, and four open doors for research, failure analysis, tooling, and mentored review.">

Huntley's flattened org chart - senior ICs directing AI agents, middle management dissolved - is probably directionally correct for greenfield companies. But most of the industry has existing systems, existing teams, and existing junior engineers already on the payroll. The question isn't whether to hire juniors in an AI-native startup. It's what to do with the ones you have.

This suggests a different starting point: reconsider whether AI-assisted production work should be a junior engineer's primary role. And when they do work on production code, give them room to fail safely - with guardrails, not blame.

Failure is how expertise gets built. The senior engineer who spots the bad architectural choice learned to spot it by making that choice themselves three years ago and living with the consequences. If we shield juniors from failure by having AI do the work, or punish them for failing by measuring only velocity, we're cutting off the only path to the judgement everyone agrees they need.

**Research and exploration.** Someone needs to evaluate whether that new library does what the README claims. Someone needs to spike on three approaches before the team commits to one. This builds deep understanding, has clear verification, and doesn't carry production risk.

**Failure analysis.** AI is consuming the traditional junior training ground of small bugs and well-defined tasks. But there's work AI is terrible at: understanding *why* something broke in a specific system's history. Post-mortems, incident analysis, debugging production issues with a senior pairing alongside - this is where intuition gets built.

**Tooling and developer experience.** Internal tools, build pipelines, test infrastructure. Contained scope, tight feedback loops, and the person doing it ends up understanding the system's internals better than anyone.

**Structured verification.** Instead of expecting juniors to verify AI output alone, make it a mentored activity. A senior writes the prompt, a junior reviews the output with the senior walking through what to look for. Progressive disclosure of complexity. The junior builds verification skills *on* AI output rather than despite it.

## Guardrails as infrastructure

<img class="basic-alignment right flex-content" src="/images/ai/junior-guardrails.jpg" alt="Propaganda poster showing a worker protected by scaffolding labeled CI, Tests, Rules, and Review Gates, contrasted with a crossed-out scene of a boss just pointing a finger.">

I wrote about this [back in December](/blog/2025/12/23/breaking-into-realism-with-ai/): staff engineers should be helping juniors build verification skills by adding guardrails so AI assists rather than replacing judgement. The practical version of that looks like engineering infrastructure, not mentoring programmes.

CI that rejects bad output, templates that narrow the problem space, review gates that intervene before mistakes ship. Project-level rules for AI agents (system prompts, [MDC rules](https://docs.cursor.com/context/rules), or equivalent) that encode what "good" looks like so agents produce consistent output. The junior gets the speed boost. The system catches what they miss.

The pattern is always the same: documentation, tooling, and direct support designed to develop independent capability. Not "here's the ticket, use AI, ship it." More like "here's the context, here's how the pieces connect, here's what to look for when it breaks."

None of this requires telling anyone to struggle harder. It requires building environments where the right kind of struggle is the natural consequence of the work itself.

## What this means for teams

<img src="/images/ai/junior-engineers-or-operators.jpg" alt="Propaganda poster of a balance scale overloaded with implement-with-AI tickets on one side and investigate, evaluate, explore cards on the other. Caption: engineers or prompt operators?">

Look at your junior engineers' work. How many are "implement this feature with AI assistance"? How many are "investigate why X behaves this way" or "evaluate whether Y would solve our Z problem"?

The ratio tells you whether you're building engineers or building prompt operators.

And this matters beyond your current team. Every generation of engineers has reshaped the profession by arriving with different instincts. If we let the junior role collapse into unsupervised AI output verification - the one task they're least equipped for - we lose that pipeline entirely.

Give them the right work. Build the guardrails. Let them explore, break things, and build the judgement that makes AI actually useful. The investment pays for itself.
