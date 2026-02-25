---
title: "Context gathering for AI: why SFT is the hard one"
categories:
  - AI
---

When you wire AI into a workflow, the first question is not "which model?" - it is "how does the model get the context it needs?"

That answer shapes everything: cost, latency, accuracy, and whether you can verify what went wrong when it does.

## The landscape

**Prompt engineering at existing tooling** (Copilot 365, Asana AI, etc.)

You rely on their fetch approaches - what they pull in, how they chunk it, and what gets priority. Convenient until it is not. Often expensive, and you cannot see inside the box. The main risk is vendor lock-in on context strategy: if the tool decides a document is irrelevant, you may not find out until the output is wrong.

**RAG (Retrieval-Augmented Generation)**

Heavy reliance on how you create and structure context. Useful for quick document collection. Swappable: bad documents can be removed without retraining. But relevance depends on embedding quality and chunking strategy. The hidden cost is maintenance: embeddings drift as your corpus changes, and re-indexing pipelines are easy to defer and hard to debug when retrieval quality degrades silently.

**Long context windows**

The brute-force option: put everything in. Increasingly viable with 128k to 1M+ token windows. Simple, but expensive at inference, and relevance dilutes as context grows. In practice, models attend unevenly to long contexts [[1]](https://arxiv.org/abs/2307.03172) - information in the middle of a large prompt is more likely to be missed than content at the start or end.

**Agentic tool use**

The model decides what it needs and fetches dynamically. More flexible than RAG and can follow chains of reasoning. But latency and cost scale unpredictably with complexity. The operational concern is observability: when an agent takes twelve tool calls to answer a question, understanding *why* it chose that path - and whether it missed a shorter one - requires structured trace logging that most teams do not have yet.

**SFT (Supervised Fine-Tuning)**

This is where things get interesting - and where most teams underestimate the work.

<img class="basic-alignment right flex-content" src="/images/ai/context-tradeoffs-mathcore-debate.jpg" alt="Debate about context gathering tradeoffs with a whiteboard of approach choices.">

## SFT: baking knowledge into weights

The promise is compelling: encode your corpus into the model itself. No retrieval latency. No context window limits. The model just *knows*.

The reality is more complicated.

### You are training on patterns, not just content

Fine-tuning does not just teach the model what to say - it teaches the model *how* to say it. Every stylistic choice, every abbreviation, every inconsistency in training data becomes learned behavior.

Unlike RAG, where you can swap out a bad document, or prompting, where you can iterate in minutes, SFT mistakes are expensive. You retrain or live with them.

### The data quality trap

I recently analyzed a small corpus of privacy assessment summaries spanning several years:

<table style="width: 100%; border-collapse: collapse; margin: 1rem 0;">
  <thead>
    <tr>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">Year</th>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">Count</th>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">Avg length</th>
      <th style="text-align: left; padding: 0.5rem 0.75rem;">Quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">2020-2021</td>
      <td style="padding: 0.5rem 0.75rem;">5</td>
      <td style="padding: 0.5rem 0.75rem;">~336 chars</td>
      <td style="padding: 0.5rem 0.75rem;">Brief outcomes only</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">2022</td>
      <td style="padding: 0.5rem 0.75rem;">38</td>
      <td style="padding: 0.5rem 0.75rem;">410 chars</td>
      <td style="padding: 0.5rem 0.75rem;">Short</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">2023</td>
      <td style="padding: 0.5rem 0.75rem;">97</td>
      <td style="padding: 0.5rem 0.75rem;">687 chars</td>
      <td style="padding: 0.5rem 0.75rem;">Best average</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">2024</td>
      <td style="padding: 0.5rem 0.75rem;">138</td>
      <td style="padding: 0.5rem 0.75rem;">619 chars</td>
      <td style="padding: 0.5rem 0.75rem;">Good</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">2025</td>
      <td style="padding: 0.5rem 0.75rem;">287</td>
      <td style="padding: 0.5rem 0.75rem;">491 chars</td>
      <td style="padding: 0.5rem 0.75rem;">Mixed</td>
    </tr>
    <tr>
      <td style="padding: 0.5rem 0.75rem;">2026</td>
      <td style="padding: 0.5rem 0.75rem;">47</td>
      <td style="padding: 0.5rem 0.75rem;">439 chars</td>
      <td style="padding: 0.5rem 0.75rem;">Recent, decent</td>
    </tr>
  </tbody>
</table>

The best cohort is not 2025 with nearly 3x the volume. It is 2023 - fewer samples, but consistently good.

More data amplified variance, not signal.

### What training metrics actually tell you

When you kick off a fine-tuning run, you will see numbers streaming in: loss dropping, accuracy climbing, maybe entropy moving around. It is tempting to watch loss decrease and assume things are working. These metrics can mislead if you do not know what they are measuring.

**Loss** measures how surprised the model is by the correct token. A loss of 2.5-3.0 means the model assigns roughly 8% probability to the correct next token - common early in training. In the privacy assessment corpus I was working with, well-tuned runs converged around 0.5-1.5, though this varies significantly by domain and vocabulary size.

The trap: loss can drop while the model learns the wrong patterns confidently. If training data contains conflicting styles, the model learns both and hedges. Low loss, bad outputs.

**Token accuracy** (how often the top prediction is correct) started at 42-44% in my runs, reaching 55-70% on the best cohorts. This sounds low because many valid continuations exist. But on structured outputs - summaries, reports, specific formats - you usually want this higher. Persistently low accuracy on structured data often signals noisy training examples.

Neither metric tells you whether the model learned the right behavior - only that it learned *something* from your data. This is why held-out evaluation on realistic tasks matters more than watching curves converge.

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

```txt
Initial review -> Questions/concerns -> Discussion -> Resolution
```

The model skipped to the end. The data was accurate; it was data from the wrong point in the workflow. The valuable signal may be the escalation "This needs to go to the privacy team" or the clarifying question about truncation vs consent.

### The signal is buried, not missing

Identifying what to train on is genuinely hard. Is it the first flag? The key question? The compromise? Each serves a different function. The most useful signal is often buried mid-thread and not labeled as important.

Curating SFT data is not just filtering "bad" examples. It is understanding what task you are training the model to do, and whether the data demonstrates that task or only its outcomes.

<div style="clear: both;"></div>

### Using LLMs to assess training data

The workflow mismatch problem makes manual labeling expensive. You are not just asking "is this example good?" You are asking "which of these 15 messages in a thread represents the behavior we want to train?" That requires domain expertise, repeatedly.

<img class="basic-alignment left flex-content" src="/images/ai/context-ideas-debugging-traceability.jpg" alt="AI output debugging illustration focused on tracing input, retrieval, and tool calls.">

LLM-as-judge offers a practical middle ground. Use a model to score or rank candidate examples against explicit criteria:

- Which message first identifies a concern missing from the original spec?
- Does this response ask clarifying questions before stating conclusions?

The economics can work. A human reviewer may process 20-30 complex thread evaluations per hour. An LLM can process thousands in the same time at a fraction of the cost per evaluation - often two orders of magnitude cheaper. Humans then review the edge cases.

This is not complete automation - you are still encoding judgment, just at the meta level. Output quality depends on rubric quality, and ambiguous cases still need human eyes.

LLMs can also generate variance your dataset lacks. If examples cluster around URL privacy reviews, an LLM can generate plausible variants: API key handling, PII in logs, third-party sharing. This can improve coverage faster than waiting for real examples. The risk is synthetic edge cases that do not reflect real workflows, so generated data still needs domain validation.

<div style="clear: both;"></div>

### The knowledge cutoff problem

SFT creates a snapshot. A fine-tuned model knows what is in training data and nothing after. For stable knowledge (style guide, terminology, domain patterns), this can work. For anything that changes (current events, recent projects, updated process), you need retrieval.

That often means hybrid architecture: SFT for *how* to respond, RAG for *what* to respond about.

SFT has one deployment advantage: if users already have the base model, the update artifact can be small. A LoRA [[3]](https://arxiv.org/abs/2106.09685) adapter may be a few hundred MB instead of shipping full model weights.

The catch: LoRA adapters are tied to specific base model weights. When upstream weights change, your adapter needs retraining. You are maintaining a pipeline, not just a dataset. Include that in operational cost.

## The actual work

Data curation is not prep work for SFT. It *is* the work.

Before touching a training script, answer:

- Which examples represent the quality you want to replicate?
- What patterns in your data should *not* be learned?
- How will you handle the long tail of edge cases?
- What is your plan for knowledge that changes?

The 2023 cohort worked because of consistency, not volume. A model trained on 97 good examples can outperform one trained on 287 mixed examples because it learns cleaner patterns rather than hedging between conflicting signals.

## Choosing your approach

The options map to different points on a tradeoff surface:

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
      <td style="padding: 0.5rem 0.75rem;">Good</td>
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

That last column matters more than it first appears.

With RAG, when something goes wrong, you can trace it: "the model said X because chunk Y was retrieved." With long context, you can inspect what was in the input window. With SFT, knowledge is dissolved into weights. You cannot point to one training example and say "that is where this output came from."

Research directions exist - influence functions try to estimate which training examples most affected an output, and memorization probes can sometimes detect whether specific text was in training data. But these are computationally expensive, approximate, and not production-ready. Work on influence functions at LLM scale [[2]](https://arxiv.org/abs/2308.03296) showed useful research insights, but compute costs and approximation error still limit production debugging use.

What SFT gives you is verification at training time, not inference time: curate carefully, evaluate on held-out tasks, then trust deployment behavior. When a user asks "why did it say that?" there may not be a direct answer.

The practical implication: if your use case requires explaining *why* an output happened (compliance, debugging, trust), pure SFT is a poor fit. Hybrid setups - SFT for behavior patterns, RAG for retrievable facts - preserve traceability where it matters most.

<img class="basic-alignment right flex-content" src="/images/ai/context-ideas-ship-criteria-checklist.jpg" alt="Ship criteria checklist illustration for AI features with go or no-go framing.">

## Picking the right approach

Use the tradeoff table above as a starting point, then apply these heuristics:

- **Default to RAG** when your knowledge changes frequently, when you need to trace outputs back to source documents, or when you are just getting started and want fast iteration on what context matters.
- **Use long context** for prototyping and one-off analysis where simplicity beats cost, or when the full input is small enough that inference cost is acceptable.
- **Consider agentic tool use** when the task requires multi-step reasoning across different data sources, and you have the trace logging infrastructure to debug it.
- **Consider SFT** when you need consistent style, tone, or domain behavior that prompting alone cannot reliably produce - and when you have a curated, high-quality dataset and the pipeline to maintain it.
- **Use existing tooling** when time-to-value matters more than control, and you can tolerate the vendor's context decisions.

Most production systems end up hybrid. SFT for *how* to respond, RAG for *what* to respond about, and agentic retrieval for anything that requires following a chain of reasoning across sources.

## The bottom line

Before shipping any AI feature, define the task clearly, set a pass/fail rubric, plan your fallback policy, and monitor in production. These criteria apply regardless of which context approach you choose.

If you are considering SFT specifically, the core question is not "do we have enough data?" It is "do we have enough *good* data, and can we identify which data that is?"

The model learns whatever patterns you provide. Make sure those are the patterns you actually want.

<div style="clear: both;"></div>

---

*Worth watching: Apple's CLaRa [[4]](https://github.com/apple/ml-clara) (Continuous Latent Reasoning) explores a middle ground between retrieval and baked-in knowledge. It compresses documents into dense latent representations that the model can attend to at inference time - preserving the swappability of RAG (replace a document, update the representation) with inference characteristics closer to SFT (no retrieval step, knowledge accessed through attention). Early results are promising, and the direction hints at a future where the RAG/SFT boundary becomes a spectrum rather than a choice.*

**References**

[1] Liu, N. F. et al. "Lost in the Middle: How Language Models Use Long Contexts." 2023. [https://arxiv.org/abs/2307.03172](https://arxiv.org/abs/2307.03172)

[2] Grosse, R. et al. "Studying Large Language Model Generalization with Influence Functions." 2023. [https://arxiv.org/abs/2308.03296](https://arxiv.org/abs/2308.03296)

[3] Hu, E. J. et al. "LoRA: Low-Rank Adaptation of Large Language Models." 2021. [https://arxiv.org/abs/2106.09685](https://arxiv.org/abs/2106.09685)

[4] Apple. "CLaRa: Continuous Latent Reasoning." [https://github.com/apple/ml-clara](https://github.com/apple/ml-clara)
