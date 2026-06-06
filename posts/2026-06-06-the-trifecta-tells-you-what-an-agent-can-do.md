---
title: "The trifecta tells you what an agent can do"
categories:
  - AI
  - Security
data:
  updated: "2026-06-06 00:00"
---

The lethal trifecta is the security heuristic I reach for most often with agents. Private data, untrusted content, an exfiltration path: hold all three in one context and you have a confused deputy waiting to happen. Simon Willison named the pattern clearly in June 2025, and it has been the easiest way to explain indirect prompt injection to people who do not read security papers.

<img class="image-framed" src="/images/ai/trifecta-capability-not-safety.webp" alt="Retro poster: private data, untrusted content, and exfil path converge on a confused deputy agent. Caption: the trifecta tells you what it can do, not whether it should.">

But it operates at the wrong altitude, and the gap matters more as agents get deployed into real organisations.

The trifecta is a capability test. It asks what an agent *can* do, and answers in binary. You either hold all three powers or you don't. What it cannot tell you is whether a given action is actually harmful, because harm depends on the context the capability acts in, not on the capability itself.

Take the canonical example from Tsai and Bagdasarian's Conseca paper. Deleting an email is fine when it is clearing out spam and catastrophic when it is destroying evidence. Same capability, same agent, opposite safety verdicts. The deciding factor is the content, the goal, and the account it sits in. None of that is visible to a trifecta check.

This is why eval and safety end up task-specific and company-specific. That shape will not be engineered away by better models alone. This post walks through three of four levels: capability, norms and policy, and authority. The fourth, whether an outcome is appropriate for *this* person, is [the subject of the follow-up](https://jotter.jonathankingston.co.uk/blog/2026/06/06/appropriateness-is-what-safety-cannot-mechanise/).

## Norms and policy: appropriate flow, not capability

The cleaner model has been sitting in the privacy literature for twenty years. Helen Nissenbaum's contextual integrity defines a flow of information as appropriate or not relative to a context: who is sending, who is receiving, who the data is about, and under what norm. Move the same information across a context boundary and an entirely acceptable flow becomes a violation. A nurse telling a specialist about your diagnosis is fine. The same nurse telling your employer is not. Nothing about the data changed.

That maps onto agents far better than a capability checklist, and recent benchmarks have made it measurable.

<img class="image-framed" src="/images/ai/contextual-integrity-same-data.webp" alt="Retro two-panel poster: the same diagnosis shared with a specialist is appropriate, shared with an employer is a violation. Caption: nothing about the data changed.">

ConfAIde and PrivacyLens (2024) showed models disclosing information in contexts a human would not, including during agent trajectories even when explicitly told to protect privacy. The model often knew the norm when asked directly and broke it when acting.

CI-Work (Fu et al., 2026) takes this into the enterprise: upward to your manager, lateral to a peer, outward to a third party. Frontier models violated contextual norms between 16% and 51% of the time. For anyone deploying enterprise agents, higher task utility correlated with *more* privacy violations. The very thing that makes an enterprise agent useful, pulling in broad internal context to act on your behalf, is the thing that drives the leak.

Contextual integrity tells you safety is contextual. It does not tell you whose context. That is where deployment policy comes in, and ST-WebAgentBench (Levy et al., IBM Research) is the clearest answer I have seen.

It scores agents not on whether they finished the task but on whether they finished it *under policy*. Each task carries machine-readable policies across dimensions like consent, boundaries, hierarchy, and so on. The headline metric, Completion under Policy, only credits runs that respected every applicable rule. Across the open agents they tested, that number came in below two-thirds of the nominal completion rate. A third of the "successful" runs broke a rule on the way.

The detail that matters here: the policies are authored, not baked in. There is a policy-authoring interface and a template format, so the same workflow passes or fails depending on what a given organisation has encoded. Your finance team's "never initiate a payment without confirmation" and another firm's "never touch production data" are different policy files over the same agent. The benchmark scores the deployment, not just the model.

You cannot read safety off the model. You read it off the deployment.

Together, contextual integrity and deployment policy make up the norms-and-policy level: should this flow happen here, under your rules?

## The missing level is authority

Above norms and policy sits authority: was this flow authorised in this context, and can the system prove it?

Somebody has to say which flows are sanctioned in *this* context, and the system has to be able to check that they were. In the four-level model, that is a separate question from whether the flow fits the context or your policy files. It is [consent](https://jotter.jonathankingston.co.uk/blog/2026/02/22/consent-is-all-you-need/), expressed by a principal and verifiable by the system.

Conseca (Tsai and Bagdasarian, HotOS 2025) comes closest to naming this in code: just-in-time, contextual, human-verifiable security policies per task. ST-WebAgentBench's policy files handle the norms-and-policy half; Conseca's human-verifiable policies handle the authority half. Both point at enforcement that today often reduces to a YAML file someone wrote in advance, or to nothing at all.

When that enforcement is missing, the trifecta is what you are left with. "Don't combine these three capabilities" is the only lever available because the system has no way to represent what it was actually permitted to do. The trifecta is the fallback you get without verifiable consent.

That is a lot of engineering, and it is worth building. Deterministic gates on every connector, a norms-and-policy check on every flow, verifiable consent before each action, an eval scored against your deployment's policy files. My first instinct is that this is roughly what good looks like.

It still is not the whole of safety. [Part two](https://jotter.jonathankingston.co.uk/blog/2026/06/06/appropriateness-is-what-safety-cannot-mechanise/) is about the fourth level: what remains when all of those checks pass.

---

## References

- Simon Willison. "The lethal trifecta for AI agents: private data, untrusted content, and external communication." 16 June 2025. <https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/>
- Helen Nissenbaum. "Privacy as Contextual Integrity." *Washington Law Review* 79, 2004. See also Barth, Datta, Mitchell, Nissenbaum. "Privacy and Contextual Integrity: Framework and Applications." *IEEE S&P*, 2006.
- Niloofar Mireshghallah et al. "Can LLMs Keep a Secret?" (ConfAIde). *ICLR* 2024. arXiv:2310.17884.
- Yijia Shao et al. "PrivacyLens." *NeurIPS* 2024 Datasets and Benchmarks. arXiv:2409.00138.
- Wenjie Fu et al. "CI-Work: Benchmarking Contextual Integrity in Enterprise LLM Agents." 2026. arXiv:2604.21308.
- Ido Levy et al. "ST-WebAgentBench." IBM Research. arXiv:2410.06703.
- Lillian Tsai, Eugene Bagdasarian. "Contextual Agent Security: A Policy for Every Purpose" (Conseca). *HotOS* 2025. arXiv:2501.17070.
