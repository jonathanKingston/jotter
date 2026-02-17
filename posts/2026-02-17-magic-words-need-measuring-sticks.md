---
title: "Magic words need measuring sticks"
categories:
  - AI
data:
  updated: "2026-02-17 00:00"
---

Dave Rupert's [Magic Words](https://daverupert.com/2026/02/magic-words/) names a thing I've been stewing on. Skills, MDC rules, and system prompts are all incantations. We write them, ship them, and hope. He frames them as magic numbers: values that work for reasons we can't articulate, in conditions we can't reproduce. I think he's onto something. But I also think the bigger issue may be that we're not measuring whether these words work.

## The missing feedback loop

If I cut a skill in half, does it still work? If I double it, does it work better? These are empirical questions, and we should treat them that way.

Teams tend to treat skills and MDC files as write-once artifacts, refined by vibes and anecdote. That seems to be a core part of the problem: not just non-determinism or token cost, but the absence of a feedback loop.

This is where evals come in. Not as some abstract ML research practice, but as the same testing discipline we already apply to code.

If someone told you a function worked "most of the time" but couldn't tell you when or why it failed, you'd write tests. You'd establish expected outputs for known inputs. We have the same option with context instructions, but few teams take it.

## Two kinds of eval

The first is deterministic testing. Take a representative set of prompts that exercise the skill, run them with and without the context, then compare outputs against a rubric. This doesn't need to be sophisticated. If your "frontend design" skill is supposed to prevent generic Bootstrap-looking layouts, test that directly. If the skill doesn't shift results in a measurable direction, it's not earning its tokens.

<img class="basic-alignment right flex-content" src="/images/ai/magic-words-rubric-scoring.jpg" alt="Rubric scoring example comparing prompt outputs.">

The second borrows from how models actually get trained. RLHF (reinforcement learning from human feedback) works by having humans pick the better of two outputs. Generate paired outputs with and without the skill, then have someone with domain expertise pick the better one. Do this across enough examples and you get a signal. Not a perfect one, but a real one.

You don't need a massive sample to learn something useful. Even a modest set of paired comparisons across a few prompt categories can give you an early signal about whether your carefully crafted MDC file is doing anything at all.

<div style="clear: both;"></div>

## Token cost as a forcing function

Every skill, every MDC rule has a token cost. That cost compounds across every request. If you're adding a lot of context to every call, it should be possible to demonstrate that it produces measurably better outputs than a leaner version, or no extra context at all.

Token count becomes a metric in its own right. Not because cheaper is always better, but because a budget demands justification. Minimum viable context should come from measurement, not gut feel.

That said, not all context earns its place through output quality. A skill that prevents the model from leaking PII or generating insecure defaults might add nothing measurable to a side-by-side comparison and still be worth every token. The eval for that kind of context looks different: it's about harm prevented, not quality gained. Token budgeting isn't purely an efficiency exercise.

Progressive disclosure helps here too. Not every skill needs to be in the window for every request. If the agent pulls in context only when it's relevant, you get the benefit without compounding cost on calls that don't need it. The eval question shifts from "is this skill worth its tokens?" to "is this skill being loaded at the right time?"

Run your eval suite across three versions of a skill: full, half-length, absent. Measure output quality. Plot it against token cost. That can give you an empirical basis for what stays and what gets cut. The curve likely won't be linear; there may be a point where additional context stops helping and starts degrading through context rot.

## Why this matters

Skills, MCP, PRDs, and prompt engineering are different ways of packaging the same underlying challenge: managing what context reaches the model. The packaging changes but the need to measure effectiveness doesn't.

<img class="basic-alignment left flex-content" src="/images/ai/magic-words-feelings-vs-feedback.jpg" alt="Feelings versus feedback side-by-side illustration.">

If you have evals in place, the churn can become much more manageable. New approach drops? Port your test prompts, run your comparisons, see if it actually improves outputs for your case. Without evals, each new approach can turn into another round of hope-based adoption.

Non-determinism is real, but it's not a reason to give up on measurement. It's a reason to measure statistically rather than anecdotally. A skill that appears to improve output quality across a representative prompt set is useful, and you can quantify how useful. A skill that you *feel* helps sometimes is a spell.

Hope-based development is risky. The answer isn't to abandon context management. It's to treat context instructions with the same testing discipline we apply to everything else. Write the eval. Run the comparison. Measure the tokens. Keep what earns its place.

<div style="clear: both;"></div>
