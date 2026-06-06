---
title: "Appropriateness is what safety cannot mechanise"
categories:
  - AI
  - Security
data:
  updated: "2026-06-10 00:00"
---

In [the first post](https://jotter.jonathankingston.co.uk/blog/2026/06/06/the-trifecta-tells-you-what-an-agent-can-do/), I argued that the lethal trifecta is a capability test, not a safety model. Above it sit three questions worth engineering for: whether a flow fits the context, whether your policy allows it, and whether the action was authorised. Build those controls and you have something defensible.

This post is about the question above all of that.

## When every check passes and the act is still harmful

Picture a marketing agent that sends a personalised alcohol offer to a recovering alcoholic. Or a support agent that, asked for help, surfaces a poorly chosen example to someone living with domestic abuse. Walk the requirements from part one in order. The flow is sanctioned: the user opted into marketing, the support channel is exactly the consented one. The capability is benign: send a message, cite an example. A policy file would very likely permit both. No trifecta violation, no contextual-integrity breach in Nissenbaum's sense, no missing authorisation. Every check passes. And the act is still harmful.

The harm is not in the data, the flow, or the permission. The user record is accurate, the offer is valid, nothing leaked. The harm lives in the match between *this content* and *this recipient's state*, and that state—the alcoholism, the abuse—is something the system cannot observe.

Deterministic per-connector gates are excellent for the enumerable, local, structural harms: don't delete prod, don't wire money without confirmation, don't exfiltrate. Those checks are cheap to automate and should absolutely exist as defence in depth. But they are a sieve cut for one class. The alcohol harm is not in any single tool call. It is emergent across the trajectory, semantic rather than structural, and it turns on a hidden fact about the recipient. No number of per-call checks composes into coverage of a global, unobservable-state-dependent property.

That is where per-tool evaluation is the right tool aimed at the wrong scope. Each Model Context Protocol (MCP) tool can have its own eval and hard-stop checks. That catches structural misuse. It does not catch substantive harm to a specific person.

## Appropriateness, not a universal standard

The frame that fits is Leibo et al.'s *A theory of appropriateness with applications to generative AI* (Google DeepMind, 2024). Their argument is that appropriateness, rather than correctness or safety-as-a-fixed-property, is the right lens for generative systems—and that striving for a single universal standard is counterproductive. Appropriateness sits at many scales at once: the individual user, the app developer, the corporation, the regulator, each shaping it.

The company's image and the consumer's welfare are both inside "safety", and they are not the same thing. Brand and consumer are distinct loss functions, and they do not move together. Send an alcohol offer to a recovering alcoholic and both welfare and brand lose immediately. Run engagement-maximising dark patterns and the short-term metric wins while the consumer loses; brand damage is slower and diffused. Mis-chosen support content for someone in domestic abuse is mainly a welfare and competence failure; brand harm arrives only if the mistake becomes public.

So "safety" here is a bundle of incommensurable objectives—consumer welfare, legal exposure, brand, task success—whose weighting is itself company-specific and jurisdiction-specific. That is why eval and safety are task-specific and company-specific: not as a footnote, but because the stakeholder set and its weighting are.

## What better models cannot close

Two limits reinforce why this does not get engineered away.

The first is verification. Per-tool checks work because they target properties you can confirm mechanically: "no payment without confirmation." Appropriateness harms need the opposite kind of check. Verifying "is this offer harmful to this specific person right now" needs the recipient's hidden state and a judgment across objectives that do not reduce to a number. AI amplifies verification where verification is tractable. Appropriateness sits where it is not, so better models do not close that gap.

The second is consent. The consumer is a principal whose boundary is being crossed with no channel to have expressed it—the gap [verifiable consent](https://jotter.jonathankingston.co.uk/blog/2026/02/22/consent-is-all-you-need/) is meant to close. But consent is necessary and not sufficient. The vulnerable person often cannot or will not articulate the boundary in advance, and the company is a separate principal with its own stake. Even perfect consent enforcement leaves substantive judgment that someone has to make and own. That judgment cannot be made deterministic. Someone has to make the call, and someone has to own it.

I wish this ended in a neat architecture diagram. It does not. The harm classes are heterogeneous and sit at different levels: capability misuse at the bottom, inappropriate flow above it, missing authority above that, and substantive appropriateness at the top. Each level needs its own mechanism, and the mechanisms are partial and overlapping by necessity. CI-Work already showed that utility and privacy pull against each other rather than stacking neatly. And the appropriateness question at the top never fully mechanises.

The trifecta audit tells you what the agent can do. A policy-scored eval like ST-WebAgentBench tells you whether it should, here, for you. Above both sits a judgment about whether the outcome is appropriate for *this* person on behalf of *these* stakeholders that no benchmark retires. Build the controls from part one because each level catches a class the others miss. Do not expect the top one to finish the job.

---

## References

- [The trifecta tells you what an agent can do](https://jotter.jonathankingston.co.uk/blog/2026/06/06/the-trifecta-tells-you-what-an-agent-can-do/). Jonathan Kingston, 2026.
- [Consent is all you need](https://jotter.jonathankingston.co.uk/blog/2026/02/22/consent-is-all-you-need/). Jonathan Kingston, 2026.
- Helen Nissenbaum. "Privacy as Contextual Integrity." *Washington Law Review* 79, 2004.
- Wenjie Fu et al. "CI-Work." 2026. arXiv:2604.21308.
- Ido Levy et al. "ST-WebAgentBench." arXiv:2410.06703.
- Joel Z. Leibo et al. "A theory of appropriateness with applications to generative artificial intelligence." arXiv:2412.19010, 2024.
