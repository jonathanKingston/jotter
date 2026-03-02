---
title: "Context gathering for AI: a tradeoff guide"
categories:
  - AI
---

When you wire AI into a workflow, the first question is not "which model?" - it is "how does the model get the context it needs?"

That answer shapes everything: cost, latency, accuracy, and whether you can verify what went wrong when it does.

## The landscape

<img class="basic-alignment right flex-content" src="/images/ai/context-tradeoffs-mathcore-debate.webp" alt="People at a whiteboard comparing context gathering approaches.">

**Prompt engineering at existing tooling** (Copilot 365, Asana AI, etc.) relies on vendor fetch strategies you cannot see inside. Convenient until the tool decides a document is irrelevant and you do not find out until the output is wrong.

**RAG** gives you control over context structure but depends on embedding quality and chunking strategy. Swappable: bad documents can be removed without retraining. The hidden cost is maintenance - embeddings drift as your corpus changes, and re-indexing pipelines degrade silently.

**Long context windows** are the brute-force option: put everything in. Simple, but expensive at inference, and models attend unevenly - information in the middle of a large prompt is more likely to be missed [[1]](https://arxiv.org/abs/2307.03172), though newer models have reduced the effect.

**Agentic tool use** lets the model decide what it needs and fetch dynamically. More flexible than RAG, but latency and cost scale unpredictably. The operational concern is observability: understanding *why* an agent chose a twelve-step path requires structured trace logging most teams do not have yet.

**SFT (Supervised Fine-Tuning)** encodes knowledge into model weights. No retrieval latency, no context window limits. But the work is front-loaded into data curation, and mistakes require retraining.

<table style="width: 100%; border-collapse: collapse; margin: 1rem 0;">
  <thead>
    <tr>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">Approach</th>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">End-to-end latency</th>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">Freshness</th>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">Cost</th>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">Inference-time traceability</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">Existing tooling</td>
      <td style="padding: 0.5rem 0.75rem;">Medium</td>
      <td style="padding: 0.5rem 0.75rem;">Good</td>
      <td style="padding: 0.5rem 0.75rem;">High</td>
      <td style="padding: 0.5rem 0.75rem;">Depends on vendor</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">RAG</td>
      <td style="padding: 0.5rem 0.75rem;">Medium</td>
      <td style="padding: 0.5rem 0.75rem;">Good</td>
      <td style="padding: 0.5rem 0.75rem;">Medium</td>
      <td style="padding: 0.5rem 0.75rem;">High - retrieved chunks visible</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">Long context</td>
      <td style="padding: 0.5rem 0.75rem;">High (scales with tokens)</td>
      <td style="padding: 0.5rem 0.75rem;">Good</td>
      <td style="padding: 0.5rem 0.75rem;">High</td>
      <td style="padding: 0.5rem 0.75rem;">High - input visible</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">Agentic</td>
      <td style="padding: 0.5rem 0.75rem;">Variable</td>
      <td style="padding: 0.5rem 0.75rem;">Best (real-time if tools hit live APIs)</td>
      <td style="padding: 0.5rem 0.75rem;">Variable</td>
      <td style="padding: 0.5rem 0.75rem;">Medium - tool calls logged</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">SFT</td>
      <td style="padding: 0.5rem 0.75rem;">Low</td>
      <td style="padding: 0.5rem 0.75rem;">Poor</td>
      <td style="padding: 0.5rem 0.75rem;">High upfront</td>
      <td style="padding: 0.5rem 0.75rem;">None</td>
    </tr>
  </tbody>
</table>

That last column matters more than it first appears. With RAG, when something goes wrong, you can trace it: "the model said X because chunk Y was retrieved." With long context, you can inspect the input window. With SFT, knowledge is dissolved into weights - you cannot point to one training example and say "that is where this output came from."

## SFT: baking knowledge into weights

The promise is compelling: encode your corpus into the model itself. No retrieval latency. No context window limits. The model just *knows*.

The reality is more complicated.

### You are training on patterns, not just content

Fine-tuning does not just teach the model what to say - it teaches the model *how* to say it. Every stylistic choice, every abbreviation, every inconsistency in training data becomes learned behavior.

Unlike RAG, where you can swap out a bad document, or prompting, where you can iterate in minutes, SFT mistakes are expensive. You retrain or live with them.

### The workflow mismatch problem

Training data often captures outcomes, not processes. This creates models that confidently state conclusions without doing the actual work.

For example:

```txt
User 1: "I would like to add URL data collection to feature Y"
User 2: "Thanks for filing"
User 3: "This needs to go to the privacy team"
Privacy team: "Can we truncate to hostname or ask for consent?"
User 1: "Yes - we do not need the path, and we will ask for consent"
Privacy team: "Summary: Adding hostname collection with consent"
```

If you collapse that thread into a summary-only training pair, the model may learn:

```txt
Input: "We'd like to add URL data collection to feature Y"
Output: "We're going to collect Hostname with consent"
```

But the real workflow was:

<svg viewBox="0 0 720 48" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Workflow: Initial review to Questions/concerns to Discussion to Resolution" style="max-width:720px;width:100%;height:auto;font-family:system-ui,sans-serif;font-size:14px">
  <title>Workflow: Initial review, Questions/concerns, Discussion, Resolution</title>
  <defs>
    <marker id="ar" markerWidth="8" markerHeight="6" refX="6" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6" fill="#64748b"/></marker>
  </defs>
  <g fill="#f1f5f9" stroke="#94a3b8">
    <rect x="0" y="8" width="138" height="32" rx="6"/>
    <rect x="192" y="8" width="168" height="32" rx="6"/>
    <rect x="414" y="8" width="118" height="32" rx="6"/>
    <rect x="586" y="8" width="118" height="32" rx="6"/>
  </g>
  <g fill="#334155" text-anchor="middle" dominant-baseline="central">
    <text x="69" y="24">Initial review</text>
    <text x="276" y="24">Questions/concerns</text>
    <text x="473" y="24">Discussion</text>
    <text x="645" y="24">Resolution</text>
  </g>
  <g stroke="#64748b" stroke-width="1.5" fill="none" marker-end="url(#ar)">
    <line x1="140" y1="24" x2="188" y2="24"/>
    <line x1="362" y1="24" x2="410" y2="24"/>
    <line x1="534" y1="24" x2="582" y2="24"/>
  </g>
</svg>

The model skipped to the end. The data was accurate; it was data from the wrong point in the workflow. The valuable signal may be the escalation "This needs to go to the privacy team" or the clarifying question about truncation vs consent.

### The signal is buried, not missing

Identifying what to train on is genuinely hard. Is it the first flag? The key question? The compromise? Each serves a different function. The most useful signal is often buried mid-thread and not labeled as important.

Curating SFT data is not just filtering "bad" examples. It is understanding what task you are training the model to do, and whether the data demonstrates that task or only its outcomes.

### The knowledge cutoff problem

SFT creates a snapshot. A fine-tuned model knows what is in training data and nothing after. For stable knowledge (style guide, terminology, domain patterns), this can work. For anything that changes (current events, recent projects, updated process), you need retrieval.

That often means hybrid architecture: SFT for *how* to respond, RAG for *what* to respond about. This boundary may soften: Apple's CLaRa [[4]](https://arxiv.org/abs/2511.18659) compresses documents into dense latent representations that the model attends to at inference time - preserving the swappability of RAG with inference characteristics closer to SFT. Early results hint at a spectrum rather than a binary choice.

SFT has one deployment advantage: if users already have the base model, the update artifact can be small. A LoRA [[3]](https://arxiv.org/abs/2106.09685) adapter may be a few hundred MB instead of shipping full model weights.

The catch: LoRA adapters are tied to specific base model weights. When upstream weights change, your adapter needs retraining. You are maintaining a pipeline, not just a dataset. Include that in operational cost.

## The actual work: evaluating training data

Data curation is not prep work for SFT. It *is* the work.

Before touching a training script, answer:

- Which examples represent the quality you want to replicate?
- What patterns in your data should *not* be learned?
- How will you handle the long tail of edge cases?
- What is your plan for knowledge that changes?

The goldilocks zone for training data is not necessarily the newest or largest cohort. Volume does not distinguish quality - a consistent smaller set will outperform a larger mixed one because the model learns cleaner patterns rather than hedging between conflicting signals.

### What training metrics tell you

<img class="basic-alignment left flex-content" src="/images/ai/context-ideas-debugging-traceability.webp" alt="AI output debugging illustration focused on tracing input, retrieval, and tool calls.">

When you kick off a fine-tuning run, you will see numbers streaming in: loss dropping, accuracy climbing, maybe entropy moving around. It is tempting to watch loss decrease and assume things are working. These metrics can mislead if you do not know what they are measuring.

**Loss** measures how surprised the model is by the correct token. A loss of 2.5-3.0 means the model assigns roughly 5-8% probability to the correct next token - common early in training. In the privacy assessment corpus I was working with, well-tuned runs converged around 0.5-1.5, though this varies significantly by domain and vocabulary size.

The trap: loss can drop while the model learns the wrong patterns confidently. If training data contains conflicting styles, the model learns both and hedges. Low loss, bad outputs.

**Token accuracy** (how often the top prediction is correct) started at 42-44% in my runs, reaching 55-70% on the best cohorts. This sounds low because many valid continuations exist. But on structured outputs - summaries, reports, specific formats - you usually want this higher. Persistently low accuracy on structured data often signals noisy training examples.

Neither metric tells you whether the model learned the right behavior - only that it learned *something* from your data. This is why held-out evaluation on realistic tasks matters more than watching curves converge.

### Using LLMs to assess training data

The [workflow mismatch problem](#the-workflow-mismatch-problem) makes manual labeling expensive. You are not just asking "is this example good?" You are asking "which of these 15 messages in a thread represents the behavior we want to train?" That requires domain expertise, repeatedly.

LLM-as-judge offers a practical middle ground. Use a model to score or rank candidate examples against explicit criteria:

- Which message first identifies a concern missing from the original spec?
- Does this response ask clarifying questions before stating conclusions?

The economics can work. A human reviewer may process 20-30 complex thread evaluations per hour. An LLM can process thousands in the same time at a fraction of the cost per evaluation - often two orders of magnitude cheaper. Humans then review the edge cases.

This is not complete automation - you are still encoding judgment, just at the meta level. Output quality depends on rubric quality, and ambiguous cases still need human eyes.

LLMs can also generate variance your dataset lacks. If examples cluster around URL privacy reviews, an LLM can generate plausible variants: API key handling, PII in logs, third-party sharing. This can improve coverage faster than waiting for real examples, though synthetic edge cases still need domain validation.

Training metrics tell you the model is learning. LLM-as-judge tells you whether it is learning from the right examples. Neither replaces held-out evaluation on realistic tasks, but together they close the gap between "we have data" and "we have data worth training on."

## Picking the right approach

<img class="basic-alignment right flex-content" src="/images/ai/context-ideas-ship-criteria-checklist.webp" alt="Ship criteria checklist illustration for AI features with go or no-go framing.">

- **Default to RAG** when your knowledge changes frequently, when you need to trace outputs back to source documents, or when you are just getting started and want fast iteration on what context matters.
- **Use long context** for prototyping and one-off analysis where simplicity beats cost, or when the full input is small enough that inference cost is acceptable.
- **Consider agentic tool use** when the task requires multi-step reasoning across different data sources, and you have the trace logging infrastructure to debug it.
- **Consider SFT** when you need consistent style, tone, or domain behavior that prompting alone cannot reliably produce - and when you have a curated, high-quality dataset and the pipeline to maintain it.
- **Use existing tooling** when time-to-value matters more than control, and you can tolerate the vendor's context decisions.

Most production systems end up hybrid. SFT for *how* to respond, RAG for *what* to respond about, and agentic retrieval for anything that requires following a chain of reasoning across sources.

Research directions exist for SFT traceability - influence functions try to estimate which training examples most affected an output, and memorization probes can sometimes detect whether specific text was in training data. But these are computationally expensive, approximate, and not production-ready [[2]](https://arxiv.org/abs/2308.03296). If your use case requires explaining *why* an output happened (compliance, debugging, trust), pure SFT is a poor fit. Hybrid setups preserve traceability where it matters most.

## The bottom line

Every approach in this guide trades off different things, but they share one constraint: the model can only work with the context it receives. How you gather, filter, and structure that context matters more than which model you run.

For SFT specifically, the core question is not "do we have enough data?" It is "do we have enough *good* data, and can we identify which data that is?"

The model learns whatever patterns you provide. Make sure those are the patterns you actually want.

---

**References**

[1] Liu, N. F. et al. "Lost in the Middle: How Language Models Use Long Contexts." 2023. [https://arxiv.org/abs/2307.03172](https://arxiv.org/abs/2307.03172)

[2] Grosse, R. et al. "Studying Large Language Model Generalization with Influence Functions." 2023. [https://arxiv.org/abs/2308.03296](https://arxiv.org/abs/2308.03296)

[3] Hu, E. J. et al. "LoRA: Low-Rank Adaptation of Large Language Models." 2021. [https://arxiv.org/abs/2106.09685](https://arxiv.org/abs/2106.09685)

[4] He, J. et al. "CLaRa: Bridging Retrieval and Generation with Continuous Latent Reasoning." 2025. [https://arxiv.org/abs/2511.18659](https://arxiv.org/abs/2511.18659)
