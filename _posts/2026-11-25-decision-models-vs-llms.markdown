---
layout: post
title:  "Decision models vs. LLMs: what they are, and why the distinction matters"
date:   2026-11-25 10:00:00 -0000
categories: ml
---

Most of what people mean when they say "AI model" today is a generative,
autoregressive LLM: feed it tokens, it predicts the next one, repeat. But
a parallel thread has been getting a lot of attention lately under the
name **decision models** — models that don't generate text at all. They
take a fixed set of options and *score* them. This post is my notes on
what that distinction actually means, prompted by reading Marc Brooker's
two posts on building one himself, plus TypeSafe AI's "Jev" model, which
popularized the framing.

## The core idea

A generative LLM answers an open-ended prompt by producing a sequence of
tokens, one at a time, sampled from a distribution over the entire
vocabulary at every step. That's necessary when the space of possible
answers is unbounded — write me an email, summarize this document,
explain this concept.

A lot of what agentic systems actually do isn't that. It's routing a
support ticket into one of twelve categories. Deciding whether a tool
call is safe to execute. Scoring how severe a security alert is on a
1–5 scale. Answering "is this true, yes or no, and how confident are
you?" The answer space is small and known in advance — so why pay for,
and wait on, open-ended generation?

A decision model is architecturally a classifier wearing an LLM's
clothes: take a pretrained transformer "torso" (the part that builds a
contextual representation of the input), chop off the language-modeling
head that predicts the next token, and replace it with a **pointer
head** — a mechanism that scores each of the caller-supplied options
against the model's hidden state and returns a probability for each one.
The lineage of "point at one of several candidates instead of generating
from a vocabulary" goes back to [Pointer
Networks](https://arxiv.org/abs/1506.03134) (Vinyals et al., 2015),
originally built for combinatorial problems like convex hulls and TSP;
decision models apply the same trick to structured LLM output.

Two worked examples made this concrete for me:

- Marc Brooker's **Hobson**: a 2B-parameter decision model built on a
  Qwen3.5 torso with a pointer head bolted on, trained with LoRA plus
  self-distillation (KL divergence against the original model's outputs,
  to avoid catastrophic forgetting of what the base model already knew).
  It hit near-perfect in-distribution accuracy, a Brier score of 0.009 on
  easy tasks (very well calibrated), and ~100ms median latency on a
  consumer GPU. His main takeaway: this is fundamentally a **data game**
  — parameter count mattered less than data diversity and task variety.
  ([Small Decisions: Engineering a Leading
  Model](https://brooker.co.za/blog/2026/09/28/engineering-system-one.html))
- **TypeSafe AI's Jev**: the production version of the same idea. You
  hand it some state (an email, a log line, a JSON blob) plus a list of
  typed questions, and it returns typed answers — one of your supplied
  choices, a rubric score, a yes/no — each with a calibrated probability,
  in about 0.4 seconds, and claims 40–200x lower latency/cost than
  routing the same decision through a frontier LLM. ([Introducing System
  One Models &
  Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev))

## Encoder or decoder?

The obvious follow-up question: if you're building a classifier, why
start from a causal decoder (GPT-style, attends only to the left) at
all? Why not a bidirectional encoder (BERT-style), which seems like the
architecturally "correct" choice for scoring a fully-visible input?

Brooker tried exactly that in a follow-up post and the result is
genuinely unresolved:

- An encoder-only version matched the decoder on most benchmarks and
  won decisively on latency (~35ms vs. ~65ms) for short prompts.
- The decoder's advantage grew with prompt length, since full
  bidirectional attention is O(n²) while the decoder used linear-cost
  layers for most of its depth.
- Bidirectional attention actually *underperformed* on some aggregate
  benchmarks despite winning on individual eval sets — a counterintuitive
  result that the post doesn't fully explain.

His conclusion: "Neither seems like a slam-dunk winner for this kind of
work at this size" — it depends on your latency budget and sequence
lengths. ([Strands Decider: Why Not an
Encoder?](https://brooker.co.za/blog/2026/10/04/encoders.html))

This is really the same question BERT (encoder) and GPT (decoder)
answered differently back in 2018, resurfacing at small scale for a
narrower task. If you want the original framing, [BERT: Pre-training of
Deep Bidirectional Transformers for Language
Understanding](https://arxiv.org/abs/1810.04805) is still the clearest
statement of why bidirectional context should help understanding tasks.
The closest adjacent idea in information retrieval is the **cross-encoder
vs. bi-encoder** distinction used for reranking — a cross-encoder reads
query and document together for accuracy, a bi-encoder embeds them
separately for speed. [This explainer on bi- vs.
cross-encoders](https://zeroentropy.dev/articles/biencoder-vs-crossencoder/)
is a good primer on that trade-off, which rhymes with the encoder/decoder
decision-model question above.

## Why "System 1"?

Both Brooker and TypeSafe frame this using Daniel Kahneman's System
1/System 2 split from *Thinking, Fast and Slow*: System 1 is fast,
automatic, intuitive judgment; System 2 is slow, deliberate, step-by-step
reasoning. A decision model runs in a single forward pass with no visible
intermediate reasoning — it's System 1 by construction. A
reasoning-enabled LLM generating a chain of thought before answering is
attempting System 2.

The practical argument: most of what a production agent does isn't
writing, it's deciding — should this tool call execute, is this ticket
urgent, is this alert a true positive. Routing *all* of those decisions
through an expensive System-2-style reasoning model is wasteful; the
better architecture is a fast System 1 model by default, escalating to
System 2 only when the System 1 model's own calibrated confidence is
low. [This explainer](https://jocheojeda.com/2026/09/25/system-1-and-system-2-ai-model-types/)
lays out that argument well, and the [original Kahneman
framing](https://en.wikipedia.org/wiki/Thinking,_Fast_and_Slow) is worth
reading if you haven't.

## Why calibration matters more here than for a chatbot

For a generative LLM, "confidence" is a vague, mostly unmeasured
property. For a decision model used to gate automated actions (does this
alert get auto-escalated? does this tool call get auto-approved?), the
*probability it returns has to mean what it says* — a model that says
"90% confident" needs to be right about 90% of the time it says that, not
60% or 99%. That's calibration, and it's measured with tools like the
**Brier score** and expected calibration error. [On Calibration of Modern
Neural Networks](https://arxiv.org/abs/1706.04599) (Guo et al., 2017) is
the standard reference — it's also the paper that established that
bigger modern networks tend to be *more* confidently wrong than older,
smaller ones, which is exactly the failure mode you can't afford in a
decision model gating real actions.

## How these get trained

Both posts lean on two now-standard techniques worth knowing on their
own:

- **LoRA** (Low-Rank Adaptation) — instead of fine-tuning all of a
  pretrained model's weights, freeze them and inject small trainable
  rank-decomposition matrices into each layer, cutting trainable
  parameters by orders of magnitude. [LoRA: Low-Rank Adaptation of Large
  Language Models](https://arxiv.org/abs/2106.09685) (Hu et al., 2021) is
  the original paper.
- **Self-distillation to avoid catastrophic forgetting** — when you
  fine-tune a pretrained model on a narrow task, it tends to forget
  general capability it had before. Penalizing the fine-tuned model's
  output distribution for diverging (via KL divergence) from the
  original model's distribution on the same inputs keeps it anchored.
  This is a standard trick in the broader knowledge-distillation
  literature, and is an active research area specifically for LoRA —
  e.g. [Low-Rank Adaptation Reduces Catastrophic Forgetting in Sequential
  Transformer Encoder Fine-Tuning](https://arxiv.org/html/2603.27707).

## Further reading

**Primary sources for this post:**
- [Small Decisions: Engineering a Leading Model](https://brooker.co.za/blog/2026/09/28/engineering-system-one.html) — Marc Brooker
- [Strands Decider: Why Not an Encoder?](https://brooker.co.za/blog/2026/10/04/encoders.html) — Marc Brooker
- [Introducing System One Models & Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev) — TypeSafe AI
- [System 1 and System 2: The Two Types of AI Thinking (and Why Jev Is a "System One" Model)](https://jocheojeda.com/2026/09/25/system-1-and-system-2-ai-model-types/) — Joche Ojeda

**Background concepts, in the order I'd read them:**
1. [Thinking, Fast and Slow](https://en.wikipedia.org/wiki/Thinking,_Fast_and_Slow) — Kahneman's original System 1/2 framing
2. [BERT: Pre-training of Deep Bidirectional Transformers](https://arxiv.org/abs/1810.04805) — why bidirectional encoders exist
3. [Pointer Networks](https://arxiv.org/abs/1506.03134) — the "point at a candidate, don't generate from a vocabulary" idea
4. [On Calibration of Modern Neural Networks](https://arxiv.org/abs/1706.04599) — why calibration isn't automatic, and how to measure it
5. [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685) — the standard cheap fine-tuning method used to build these
6. [Bi-Encoders vs. Cross-Encoders](https://zeroentropy.dev/articles/biencoder-vs-crossencoder/) — the closest adjacent speed/accuracy trade-off in retrieval
