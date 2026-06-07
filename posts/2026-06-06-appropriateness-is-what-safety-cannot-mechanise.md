---
title: "Appropriateness is what safety cannot mechanise"
categories:
  - AI
  - Security
is_draft: true
data:
  updated: "2026-06-06 00:00"
---

In [the first post](https://jotter.jonathankingston.co.uk/blog/2026/06/06/the-trifecta-tells-you-what-an-agent-can-do/), I argued that the lethal trifecta is a capability test, not a safety model. Above capability sit further checks: contextual integrity and org policy rules, authority, and whether the outcome fits the person. The first post covered capability through authority. Build those controls and you have something defensible.

This post is about the case where every one of those checks passes and the recipient is still harmed.

<img class="image-framed" src="/images/ai/appropriateness-every-check-passes-v2.webp" alt="Retro poster: an agent's clipboard shows capability, context, org policy, and consent all checked; a wine offer reaches a distressed recipient at her desk whose recovery the system cannot see. Caption: every check passes, still harmful.">

## When every check passes and the act is still harmful

Picture a marketing agent that sends a personalised alcohol offer to a recovering alcoholic. Or a support agent that, asked for help, surfaces a poorly chosen example to someone living with domestic abuse. Walk the checks from part one in order. The flow fits the context and your policy files. The action was authorised. The capability is benign: send a message, cite an example. No trifecta violation, no contextual-integrity breach in Nissenbaum's sense, no missing authorisation. Every check passes. And the act is still harmful.

The user record is accurate, the offer is valid, nothing leaked. The harm sits in the match between *this content* and *this recipient's state* (the alcoholism, the abuse), something the system cannot observe.

Deterministic per-connector gates are excellent for the enumerable, local, structural harms: don't delete prod, don't wire money without confirmation, don't exfiltrate. Those checks are cheap to automate and should absolutely exist as defence in depth. They catch one class of harm only. The alcohol harm emerges across the trajectory, not in any single tool call. It is about meaning, not structure, and it turns on facts about the recipient the system cannot see. No number of per-call checks adds up to that.

Platform safety classifiers, like those shipped with Claude and Cursor, focus on compromise, credential abuse, and exfiltration. That is necessary work on capability, and [a poor match for scam pages and social engineering on the open web](https://jotter.jonathankingston.co.uk/blog/2026/05/10/when-agents-browse-the-web-the-web-wins/): in the Web Adversaries Against Agentic Browsers (WAAA) benchmarks, BrowseSafe classified representative scam pages as benign because they contained no prompt injection. They are not built to catch task-specific harm in a given workflow: the wrong offer to the wrong person, an example that retraumatises a caller, a step your org's policy file forbids but no generic filter has heard of.

The per-tool instinct is right for that gap, and still incomplete. Each Model Context Protocol (MCP) connector, each workflow, should carry its own context-dependent evals, graders, and protections: rules written for what *this* tool does in *this* deployment — and [you need to measure them, not assume they work](https://jotter.jonathankingston.co.uk/blog/2026/02/17/magic-words-need-measuring-sticks/). A filesystem tool needs different gates from a payment tool or a customer messaging tool. Generic classifiers cannot substitute for that granularity.

Those per-tool checks catch structural misuse and deployment-specific policy breaks. They still do not catch harm to a specific person — the question this post is about.

## Appropriateness, not a universal standard

Leibo et al.'s *A theory of appropriateness with applications to generative AI* (Google DeepMind, 2024) argues that appropriateness—not a single correctness score or a fixed safety label—is what you should optimise for in generative systems. They also argue against one universal standard, because it collapses into a lowest-common-denominator model that tries to please every context and pleases none. User, developer, company, and regulator each apply different expectations at once.

The company's image and the consumer's welfare are both inside "safety", and they are not the same thing. Brand and consumer are distinct loss functions, and they do not move together. Send an alcohol offer to a recovering alcoholic and both welfare and brand lose immediately. Run engagement-maximising dark patterns and the short-term metric wins while the consumer loses; brand damage is slower and diffused. Mis-chosen support content for someone in domestic abuse is mainly a welfare and competence failure; brand harm arrives only if the mistake becomes public.

So "safety" here is a bundle of incommensurable objectives (consumer welfare, legal exposure, brand, task success) whose weighting is itself company-specific and jurisdiction-specific. That is why eval and safety are task-specific and company-specific: the stakeholder set and its weighting are, not a footnote.

## What better models cannot close

Two limits keep the gap open.

Verification is the first. Per-tool checks work because they target properties you can confirm mechanically: "no payment without confirmation." Appropriateness harms need the opposite kind of check. Verifying "is this offer harmful to this specific person right now" needs the recipient's hidden state and a judgment across objectives that do not reduce to a number. This is [Verifier's Law](https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law) again: AI amplifies verification where verification is tractable, not where it is not. Appropriateness sits in the second bucket, so better models do not close that gap.

Consent is the second. The consumer is a principal whose boundary is being crossed with no channel to have expressed it. That is the gap [verifiable consent](https://jotter.jonathankingston.co.uk/blog/2026/02/22/consent-is-all-you-need/) is meant to close. Consent helps; it does not replace judgment. The vulnerable person often cannot or will not articulate the boundary in advance, and the company is a separate principal with its own stake. Even perfect consent enforcement leaves calls about fit for this person that someone must make and own—and those calls cannot be made deterministic.

Different harm types need different checks: capability, contextual integrity and org policy rules, authority, and whether the outcome fits the person.

<img class="image-framed" src="/images/ai/safety-four-gates.svg" alt="Walkthrough diagram: a personalised alcohol offer email passes capability, context and org policy, and authority gates, but fit for this person still needs a human call and the outcome can remain harmful.">

Each check needs its own mechanism, and the mechanisms overlap. CI-Work already showed that utility and privacy trade off in practice. Whether the outcome fits the person never fully mechanises.

The trifecta check tells you what the agent can do. An eval scored against your deployment's policy files tells you whether it should, here, for you. Above both sits a judgment about whether the outcome is appropriate for *this* person on behalf of *these* stakeholders that no benchmark retires. Build the controls from part one because each check catches a class the others miss. Do not expect appropriateness alone to finish the job.

---

## References

- [The trifecta tells you what an agent can do](https://jotter.jonathankingston.co.uk/blog/2026/06/06/the-trifecta-tells-you-what-an-agent-can-do/). Jonathan Kingston, 2026.
- [Consent is all you need](https://jotter.jonathankingston.co.uk/blog/2026/02/22/consent-is-all-you-need/). Jonathan Kingston, 2026.
- Helen Nissenbaum. *Privacy in Context: Technology, Policy, and the Integrity of Social Life.* Stanford University Press, 2009.
- Wenjie Fu et al. "CI-Work." 2026. arXiv:2604.21308.
- Ido Levy et al. "ST-WebAgentBench." arXiv:2410.06703.
- Joel Z. Leibo et al. "A theory of appropriateness with applications to generative artificial intelligence." arXiv:2412.19010, 2024.
- Jason Wei. "Asymmetry of verification and verifier's law." 2026. <https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law>
