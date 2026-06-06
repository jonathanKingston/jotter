---
title: "The trifecta tells you what an agent can do. Safety is everything else."
categories:
  - AI
  - Security
data:
  updated: "2026-06-06 00:00"
---

The lethal trifecta is the security heuristic I reach for most often with agents. Private data, untrusted content, an exfiltration path: hold all three in one context and you have a confused deputy waiting to happen. Simon Willison named the pattern clearly in June 2025, and it has been the easiest way to explain indirect prompt injection to people who do not read security papers.

But it operates at the wrong altitude, and the gap matters more as agents get deployed into real organisations.

The trifecta is a capability test. It asks what an agent *can* do, and answers in binary. You either hold all three powers or you don't. What it cannot tell you is whether a given action is actually harmful, because harm is not a property of the capability. It is a property of the context the capability acts in.

Take the canonical example from Tsai and Bagdasarian's Conseca paper. Deleting an email is fine when it is clearing out spam and catastrophic when it is destroying evidence. Same capability, same agent, opposite safety verdicts. The deciding factor is the content, the goal, and the account it sits in. None of that is visible to a trifecta audit.

This is why eval and safety end up task-specific and company-specific, and it is not an inconvenience to be engineered away. It is the actual shape of the problem. The rest of this post walks through four questions above a trifecta audit—from capabilities up to the part that no amount of engineering removes.

## Safety as appropriate flow, not capability

The cleaner model has been sitting in the privacy literature for twenty years. Helen Nissenbaum's contextual integrity defines a flow of information as appropriate or not relative to a context: who is sending, who is receiving, who the data is about, and under what norm. Move the same information across a context boundary and an entirely acceptable flow becomes a violation. A nurse telling a specialist about your diagnosis is fine. The same nurse telling your employer is not. Nothing about the data changed.

That maps onto agents far better than a capability checklist, and a run of recent benchmarks have made it measurable.

ConfAIde (Mireshghallah et al., ICLR 2024) was among the first benchmarks to test the idea. Grounded in contextual integrity, it showed GPT-4 and ChatGPT disclosing information in contexts a human would not, 39% and 57% of the time. PrivacyLens (Shao et al., NeurIPS 2024) pushed the same idea into agent trajectories and found GPT-4 leaking sensitive information in roughly a quarter of cases *even when explicitly told to protect privacy*. The model knew the norm when asked directly and broke it when acting.

CI-Work (Fu et al., 2026) simulates real organisational flows: upward to your manager, lateral to a peer, outward to a third party. Frontier models violated contextual norms between 16% and 51% of the time. For anyone deploying enterprise agents, the uncomfortable part is that higher task utility correlated with *more* privacy violations. The very thing that makes an enterprise agent useful, pulling in broad internal context to act on your behalf, is the thing that drives the leak.

MAGPIE extends this to multi-agent settings, where agents asked to keep a secret leaked up to half of it and resorted to manipulation to get tasks done. The norm-following does not survive contact with task pressure.

## Where "company-specific" stops being hand-waving

Contextual integrity tells you safety is contextual. It does not tell you whose context. That is the part organisations actually need, and ST-WebAgentBench (Levy et al., IBM Research) is the clearest answer I have seen.

It scores agents not on whether they finished the task but on whether they finished it *under policy*. Each task carries machine-readable policies across six dimensions: consent, boundaries, hierarchy, and so on. The headline metric, Completion under Policy, only credits runs that respected every applicable rule. Across the open agents they tested, that number came in below two-thirds of the nominal completion rate. A third of the "successful" runs broke a rule on the way.

The detail that matters here: the policies are authored, not baked in. There is a policy-authoring interface and a template format, so the same workflow passes or fails depending on what a given organisation has encoded. Your finance team's "never initiate a payment without confirmation" and another firm's "never touch production data" are different policy files over the same agent. The benchmark scores the deployment, not just the model.

Several recent governance papers say the same thing with different vocabulary. The control-evaluation trajectory paper (arXiv:2504.05259) treats deployment context as a first-class variable: degree of human oversight, criticality of the systems touched, the incentives pushing toward autonomy. The International AI Safety Report 2026 makes criticality of the environment a determinant of how bad a loss of control gets. The 2023 Frontier AI Regulation paper put it plainly years ago: risk is contextual, and should be judged counterfactually against what was already possible. The 2026 reliability paper (arXiv:2602.16666) argues that which safety dimensions even matter, and at what threshold, depends on the application, the way nuclear safety prices expected consequence rather than raw failure rate.

You cannot read safety off the model. You read it off the deployment.

## The missing question is authority

Seen together, the trifecta answers a capability question: which powers are combined. Contextual integrity answers a norm question: which flows are appropriate here. Between them sits a third question none of the above fully handles, and it is the one I keep coming back to: was this flow authorised in this context, and can the system prove it?

Somebody has to say which flows are sanctioned in *this* context, and the system has to be able to check that they were. That is not a capability question and it is not a norm question. It is authority. It is [consent](https://jotter.jonathankingston.co.uk/consent-is-all-you-need), expressed by a principal and verifiable by the system.

Conseca (Tsai and Bagdasarian, HotOS 2025) comes closest to naming this in code: generating just-in-time, contextual, and human-verifiable security policies per task. The human-verifiable part is verifiable consent. ST-WebAgentBench's policy files are a static, hand-authored version of the same idea. Both point at authority enforcement that today often reduces to a YAML file someone wrote in advance, or to nothing at all.

When that enforcement is missing, the trifecta is what you are left with. "Don't combine these three capabilities" is the only lever available because the system has no way to represent what it was actually permitted to do. The trifecta is the fallback you get without verifiable consent.

## Suppose you build all of it, and it still leaks

So suppose you do all of it. Deterministic gates on every connector, a contextual-integrity check on every flow, verifiable consent before each action. My first instinct is that this is roughly what good looks like: each Model Context Protocol (MCP) tool gets its own eval, its own hard-stop checks, and you compose them into something defensible.

It is not enough, and the reason is worth being precise about.

Picture a marketing agent that sends a personalised alcohol offer to a recovering alcoholic. Or a support agent that, asked for help, surfaces a poorly chosen example to someone living with domestic abuse. Walk those four requirements in order. The flow is sanctioned: the user opted into marketing, the support channel is exactly the consented one. The capability is benign: send a message, cite an example. A policy file would very likely permit both. No trifecta violation, no contextual-integrity breach in Nissenbaum's sense, no missing authorisation. Every check passes. And the act is still harmful.

Which means the harm is not a property of the data, the flow, or the permission. "Just make the data safe" cannot touch it, because the data *is* safe: the user record is accurate, the offer is valid, nothing leaked. The harm lives in the match between *this content* and *this recipient's state*, and that state, the alcoholism, the abuse, is something the system cannot observe.

This is where per-tool evaluation is the right tool aimed at the wrong scope. Deterministic per-connector gates are excellent for the enumerable, local, structural harms: don't delete prod, don't wire money without confirmation, don't exfiltrate. Those checks are cheap to automate and should absolutely exist as defence in depth. But they are a sieve cut for one class. The alcohol harm is not in any single tool call. It is emergent across the trajectory, it is semantic rather than structural, and it turns on a hidden fact about the recipient. No number of per-call checks composes into coverage of a global, unobservable-state-dependent property.

## This is appropriateness, not safety

The frame that fits is Leibo et al.'s *A theory of appropriateness with applications to generative AI* (Google DeepMind, 2024). Their argument is that appropriateness, rather than correctness or safety-as-a-fixed-property, is the right lens for generative systems. The claim I would put near the top of any version of this post is that they argue *against* a universal standard. Striving for a single notion of appropriate behaviour is counterproductive, they say, because it collapses into a lowest-common-denominator model that tries to please every context and ends up serving none. Appropriateness sits at many scales at once, the individual user, the app developer, the corporation, the regulator, each shaping it.

That last point is Leibo et al.'s multi-scale appropriateness stated formally. The company's image and the consumer's welfare are both inside "safety", and they are not the same thing.

I would push that one step further than it usually gets pushed. Brand and consumer are distinct loss functions, and they do not move together. Send an alcohol offer to a recovering alcoholic and both welfare and brand lose immediately. Run engagement-maximising dark patterns and the short-term metric wins while the consumer loses; brand damage is slower and diffused. Mis-chosen support content for someone in domestic abuse is mainly a welfare and competence failure; brand harm arrives only if the mistake becomes public. So "safety" here is a bundle of incommensurable objectives, consumer welfare, legal exposure, brand, task success, whose weighting is itself company-specific and jurisdiction-specific.

That returns to the opening claim. Safety is company-specific because the stakeholder set and its weighting are.

## Why no amount of capability progress fixes it

Two threads from my own corner reinforce why this does not just get engineered away.

The first is verification. Deterministic per-tool checks work because they target properties you can confirm mechanically: "no payment without confirmation." Appropriateness harms need the opposite kind of check. Verifying "is this offer harmful to this specific person right now" needs the recipient's hidden state and a judgment across objectives that do not reduce to a number. AI amplifies verification where verification is tractable. Appropriateness sits where it is not, so better models do not close that gap.

The second is consent, and here I want to be honest rather than tidy. The consumer is a principal whose boundary is being crossed with no channel to have expressed it, the gap verifiable consent is meant to close. But consent is necessary and not sufficient. The vulnerable person often cannot or will not articulate the boundary in advance, and the company is a separate principal with its own stake. So even perfect consent enforcement leaves substantive judgment that someone has to make and own. That judgment cannot be made deterministic. It is not something you bolt on as another control. Someone has to make the call, and someone has to own it.

I wish this ended in a neat architecture diagram. It does not. The harm classes are heterogeneous and sit at different levels: capability misuse at the bottom, inappropriate flow above it, missing authority above that, and substantive appropriateness at the top. Each level needs its own mechanism, and the mechanisms are partial and overlapping by necessity. CI-Work already showed that utility and privacy pull against each other rather than stacking neatly. And the appropriateness question at the top never fully mechanises.

"Eval and safety are task and company specific" is therefore not a deployment footnote. It is a claim about what you are measuring, and about what you can never finish measuring. The trifecta audit tells you what the agent can do. A policy-scored eval like ST-WebAgentBench tells you whether it should, here, for you. Above both sits a judgment about whether the outcome is appropriate for *this* person on behalf of *these* stakeholders that no benchmark retires. Build the controls because each level catches a class the others miss. Do not expect the top one to finish the job.

---

## References

- Simon Willison. "The lethal trifecta for AI agents: private data, untrusted content, and external communication." 16 June 2025. <https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/>
- Helen Nissenbaum. "Privacy as Contextual Integrity." *Washington Law Review* 79, 2004. See also Barth, Datta, Mitchell, Nissenbaum. "Privacy and Contextual Integrity: Framework and Applications." *IEEE S&P*, 2006.
- Niloofar Mireshghallah, Hyunwoo Kim, Xuhui Zhou, Yulia Tsvetkov, Maarten Sap, Reza Shokri, Yejin Choi. "Can LLMs Keep a Secret? Testing Privacy Implications of Language Models via Contextual Integrity Theory" (ConfAIde). *ICLR* 2024. arXiv:2310.17884.
- Yijia Shao, Tianshi Li, Weiyan Shi, Yanchen Liu, Diyi Yang. "PrivacyLens: Evaluating Privacy Norm Awareness of Language Models in Action." *NeurIPS* 2024 Datasets and Benchmarks. arXiv:2409.00138.
- Wenjie Fu et al. "CI-Work: Benchmarking Contextual Integrity in Enterprise LLM Agents." 2026. arXiv:2604.21308.
- "MAGPIE: A Benchmark for Multi-Agent Contextual Privacy Evaluation." 2025. arXiv:2510.15186.
- Ido Levy, Ben Wiesel, Sami Marreed, Alon Oved, Avi Yaeli, Segev Shlomov. "ST-WebAgentBench: A Benchmark for Evaluating Safety and Trustworthiness in Web Agents." IBM Research. arXiv:2410.06703.
- Lillian Tsai, Eugene Bagdasarian. "Contextual Agent Security: A Policy for Every Purpose" (Conseca). *HotOS* 2025. arXiv:2501.17070.
- Joel Z. Leibo, Alexander Sasha Vezhnevets, Manfred Diaz, John P. Agapiou, William A. Cunningham, Peter Sunehag, Julia Haas, Raphael Koster, Edgar A. Duéñez-Guzmán, William S. Isaac, Georgios Piliouras, Stanley M. Bileschi, Iyad Rahwan, Simon Osindero. "A theory of appropriateness with applications to generative artificial intelligence." Google DeepMind et al. arXiv:2412.19010, 2024. (Follow-up: "A Theory of Appropriateness That Accounts for Norms of Rationality", arXiv:2603.14050, 2026.)
- "How to evaluate control measures for LLM agents? A trajectory from today to superintelligence." 2025. arXiv:2504.05259.
- "Towards a Science of AI Agent Reliability." 2026. arXiv:2602.16666.
- "International AI Safety Report 2026." arXiv:2602.21012.
- "Frontier AI Regulation: Managing Emerging Risks to Public Safety." 2023. arXiv:2307.03718.
