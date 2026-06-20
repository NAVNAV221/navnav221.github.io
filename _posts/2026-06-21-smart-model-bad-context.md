---
layout: post
title: "A Smart Model Doesn't Make Up for Bad Context"
subtitle: "With the right context, a lean and cheap model can give you the results of an expensive one."
categories: AI
cover-img: /assets/img/naboo/naboo-haiku-context-experiment.png
thumbnail-img: /assets/img/naboo/naboo-haiku-context-experiment.png
tags: [AI, Context, Naboo, LLM, Claude]
comments: true
---

A smart model doesn't make up for bad context.

Anthropic recently published a Postmortem that showed how a bug in Context management dramatically blew up the costs of Opus 4.7 and drained users' Token limit super fast ([link](https://www.anthropic.com/engineering/april-23-postmortem)).

To be clear, Opus 4.7 is an amazing model with insane research abilities ([link to the benchmarks](https://www.anthropic.com/news/claude-opus-4-7)). The problem wasn't the model — it was the amount of unnecessary Context it got.

At Naboo we ran an experiment to show that with the right context, you can get the results of an expensive model — from a lean and cheap one. We took Haiku 4.5 and asked it a vague question from our day-to-day development:

> What's the root cause of the bug we saw on teamsb?

(without explaining what `teamsb` is — it's our Teams integration — or where to look).

In the image you can see two runs of Claude Code on Haiku 4.5:

- **Without Naboo:** the model gets lost and asks for clarifications.
- **With Naboo's context:** the model pulls the exact solution in seconds. And with a tiny amount of tokens.

![Two runs of Claude Code on Haiku 4.5 — without Naboo vs. with Naboo's context](/assets/img/naboo/naboo-haiku-context-experiment.png)

------

It's important to say that we tailor the context using thousands of organizational variables: What did the user work on? Repos related to the user? Basically a whole smart organizational knowledge graph that understands the user's Intent.

— Nave
