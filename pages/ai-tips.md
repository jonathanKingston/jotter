---
title: "AI Tips"
permalink: "/pages/ai-tips/"
layout: "page.liquid"
data:
  published: "2026-03-12"
  updated: "2026-03-15"
---

## General tips

- Question everything.
- Use multiple models to cross validate output.
- Ask models to assist with writing prompts and rules. Then validate the output and refine.
- Focus more on describing ambitiously aims rather than how to achieve them. *E.g. instead of using AI to write a similar feature to before, describe the landscape of the nth feature and how we'd scale to that with mimimal effort and ideally less GenAI.*
- Context is key to getting the best output from the model.
    - Clear chats and context windows regularly.
    - Aim to attach only the most relevant context to the current request.
    - Avoid too many rules and tools.
- Understanding some basics of machine learning will assist you reasoining with why models behave the way they do. *E.g. percieve, reason, learn, act are the core components of the LLM lifecycle. Expert systems are a good analogy for rules.*
- Where full automation might be useful: Monotonous tasks with low risk of failure. *E.g. use LLM as a judge to flag for reviews or tag outputs for review.*

## Vibe coding

- Vibe coding needs guidance to allow for structure to prevent chaos. *E.g. As codebases grow, debt becomes something that needs to be managed.*
- Ordering of changes is important to avoid conflicts and maintainability. Resolving issues earlier is more important than ever.

## Rules, skills and documentation

- Make AI work for everyone:
    - Symlink Claude.md to AGENTS.md.
    - Avoid .claude/ and .cursor/ directories directly, unless you can symlink or manage them with a tool.
- Prefer regular scripts that ci, humans and skills can run.
- Documentation:
    - Move documentation where possible from internal systems (even if accessible via MCP).
    - Use regular documentation where possible that assists humans also.
    - Documentation should live as close to the code as possible.
    - Document old patterns clearly and their preferred alternatives.
- Progressive disclosure prevents the model suffering from too much context.
- Wayfind the codebase and processes lightly.
- Regularly review rules for effectiveness and relevance.
- Precision:
    - Avoid lengthy prompts or rules trying to cover everything with high precision.
    - If a codebase is idiomatic, less is more. Don't try to document how language or library features work.
- Use reviews as a way to validate the author's intent and methodology and improve rules where relevant.
- AI doesn't think like a human, don't coerce it into thinking like one as you're going to limit it. *E.g. when testing a website, describe the problem and output but not how it gets there. An AI will use curl initially, reverse engineer the responses and find the correct path before validating with a more expensive browser automation.*

## Gotchas

- AI will willingly work around tests, carefully validate what it's changed. It may hint at the problem it encountered.
- Without safeguards, AI will utilise all the access you give it at a speed not possible for a human.


## Tests

- Maintenance is less of a concern when using models to write tests. You still need to validate them lightly for correctness and flakiness.
- Don't use the model to generate endless mocks, property testing is often easier to manage.
- Use hooks to trigger test and linting on code changes.
- Ask models to encode agentic behaviour into tests.






