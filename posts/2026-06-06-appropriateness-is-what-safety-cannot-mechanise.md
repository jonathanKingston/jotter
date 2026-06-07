---
title: "Appropriateness is what safety cannot mechanise"
categories:
  - AI
  - Security
is_draft: true
data:
  updated: "2026-06-06 00:00"
---

In [the first post](https://jotter.jonathankingston.co.uk/blog/2026/06/06/the-trifecta-tells-you-what-an-agent-can-do/), I argued that the lethal trifecta is a capability test, not a safety model. Above capability sit three further checks: norms and policy (contextual integrity plus deployment policy), authority, and substantive appropriateness. The first post covered the first three. Build those controls and you have something defensible.

This post is about the fourth check.

<img class="image-framed" src="/images/ai/appropriateness-every-check-passes-v2.webp" alt="Retro poster: an agent's clipboard shows capability, norms, policy, and consent all checked, but the recipient is harmed by a wine offer matched to hidden recovery state the system cannot see. Caption: every check passes, still harmful.">

## When every check passes and the act is still harmful

Picture a marketing agent that sends a personalised alcohol offer to a recovering alcoholic. Or a support agent that, asked for help, surfaces a poorly chosen example to someone living with domestic abuse. Walk the first three checks from part one in order. The flow fits the context and your policy files. The action was authorised. The capability is benign: send a message, cite an example. No trifecta violation, no contextual-integrity breach in Nissenbaum's sense, no missing authorisation. Every check passes. And the act is still harmful.

The user record is accurate, the offer is valid, nothing leaked. The harm sits in the match between *this content* and *this recipient's state* (the alcoholism, the abuse), something the system cannot observe.

Deterministic per-connector gates are excellent for the enumerable, local, structural harms: don't delete prod, don't wire money without confirmation, don't exfiltrate. Those checks are cheap to automate and should absolutely exist as defence in depth. But they are a sieve cut for one class. The alcohol harm emerges across the trajectory, not in any single tool call. It is semantic rather than structural, and it turns on a hidden fact about the recipient. No number of per-call checks composes into coverage of a global, unobservable-state-dependent property.

Platform safety classifiers, like those shipped with Claude and Cursor, are largely aimed at the same bucket: machine compromise, credential abuse, and data exfiltration. That is necessary work on capability, and [still the wrong tool for confusion-shaped harm on the open web](https://jotter.jonathankingston.co.uk/blog/2026/05/10/when-agents-browse-the-web-the-web-wins/): in the WAAA browser-agent benchmarks, BrowseSafe classified representative scam pages as benign because they contained no prompt injection. It is not built to catch task-specific harm in a given workflow: the wrong offer to the wrong person, an example that retraumatises a caller, a step your org's policy file forbids but no generic filter has heard of.

The per-tool instinct is right for that gap, and still incomplete. Each Model Context Protocol (MCP) connector, each workflow, should carry its own context-dependent evals, graders, and protections: rules written for what *this* tool does in *this* deployment — and [you need to measure them, not assume they work](https://jotter.jonathankingston.co.uk/blog/2026/02/17/magic-words-need-measuring-sticks/). A filesystem tool needs different gates from a payment tool or a customer messaging tool. Generic classifiers cannot substitute for that granularity.

Those per-tool checks catch structural misuse and deployment-specific policy breaks. They still do not catch substantive harm to a specific person — the appropriateness question above them.

## Appropriateness, not a universal standard

The frame that fits is Leibo et al.'s *A theory of appropriateness with applications to generative AI* (Google DeepMind, 2024). Their argument is that appropriateness, rather than correctness or safety-as-a-fixed-property, is the right lens for generative systems. They also argue against a single universal standard, because it collapses into a lowest-common-denominator model that tries to please every context and ends up serving none. Appropriateness sits at many scales at once: the individual user, the app developer, the corporation, the regulator, each shaping it.

The company's image and the consumer's welfare are both inside "safety", and they are not the same thing. Brand and consumer are distinct loss functions, and they do not move together. Send an alcohol offer to a recovering alcoholic and both welfare and brand lose immediately. Run engagement-maximising dark patterns and the short-term metric wins while the consumer loses; brand damage is slower and diffused. Mis-chosen support content for someone in domestic abuse is mainly a welfare and competence failure; brand harm arrives only if the mistake becomes public.

So "safety" here is a bundle of incommensurable objectives (consumer welfare, legal exposure, brand, task success) whose weighting is itself company-specific and jurisdiction-specific. That is why eval and safety are task-specific and company-specific: the stakeholder set and its weighting are, not a footnote.

## What better models cannot close

Two limits reinforce why this does not get engineered away.

The first is verification. Per-tool checks work because they target properties you can confirm mechanically: "no payment without confirmation." Appropriateness harms need the opposite kind of check. Verifying "is this offer harmful to this specific person right now" needs the recipient's hidden state and a judgment across objectives that do not reduce to a number. This is [Verifier's Law](https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law) again: AI amplifies verification where verification is tractable, not where it is not. Appropriateness sits in the second bucket, so better models do not close that gap.

The second is consent. The consumer is a principal whose boundary is being crossed with no channel to have expressed it. That is the gap [verifiable consent](https://jotter.jonathankingston.co.uk/blog/2026/02/22/consent-is-all-you-need/) is meant to close. But consent is necessary and not sufficient. The vulnerable person often cannot or will not articulate the boundary in advance, and the company is a separate principal with its own stake. Even perfect consent enforcement leaves substantive judgment that someone has to make and own. That judgment cannot be made deterministic. Someone has to make the call, and someone has to own it.

I wish this ended in a neat architecture diagram. It does not. The harm classes are heterogeneous and split across four checks: capability, norms and policy, authority, and substantive appropriateness.

<img class="image-framed" src="/images/ai/safety-four-gates.svg" alt="Walkthrough diagram: a personalised alcohol offer email passes capability, norms and policy, and authority gates, but substantive appropriateness still needs a human call and the outcome can remain harmful.">

Each check needs its own mechanism, and the mechanisms are partial and overlapping by necessity. CI-Work already showed that utility and privacy pull against each other rather than stacking neatly. And the appropriateness question never fully mechanises.

The trifecta check tells you what the agent can do. An eval scored against your deployment's policy files tells you whether it should, here, for you. Above both sits a judgment about whether the outcome is appropriate for *this* person on behalf of *these* stakeholders that no benchmark retires. Build the controls from part one because each check catches a class the others miss. Do not expect the last one to finish the job.

---

## References

- [The trifecta tells you what an agent can do](https://jotter.jonathankingston.co.uk/blog/2026/06/06/the-trifecta-tells-you-what-an-agent-can-do/). Jonathan Kingston, 2026.
- [Consent is all you need](https://jotter.jonathankingston.co.uk/blog/2026/02/22/consent-is-all-you-need/). Jonathan Kingston, 2026.
- Helen Nissenbaum. *Privacy in Context: Technology, Policy, and the Integrity of Social Life.* Stanford University Press, 2009.
- Wenjie Fu et al. "CI-Work." 2026. arXiv:2604.21308.
- Ido Levy et al. "ST-WebAgentBench." arXiv:2410.06703.
- Joel Z. Leibo et al. "A theory of appropriateness with applications to generative artificial intelligence." arXiv:2412.19010, 2024.
- Jason Wei. "Asymmetry of verification and verifier's law." 2026. <https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law>
