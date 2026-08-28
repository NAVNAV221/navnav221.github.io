---
layout: post
permalink: "/learn/harness/"
title: How can I build a Harness?
subtitle: The five parts a harness is made of, and a prompt for each one.
share-title: "How can I build a Harness?"
share-description: "A harness is the software your model runs inside. The five parts it depends on, a prompt for each, and why to start from pi instead of a framework."
categories: Learn
tags: [Harness, Agents, AI, LLM, Learn]
thumbnail-img: /assets/img/learn/harness-anatomy.png
comments: true
---

A harness is the software your model runs inside. Not the model, not the
framework wrapped around it. The thing that decides what the model sees,
what it can do, and when it stops.

![The anatomy of a harness: an agent is a model plus a harness plus an environment. The harness itself is five parts - system prompt, tools, agentic loop, memory, and translation layer.](/assets/img/learn/harness-anatomy.png)

An agent is three things: a **model**, a **harness**, and an **environment**
it can act on (a shell, a repo, an API, a browser). The model is not yours to
build. The environment is where the effects happen. The harness is the part
you own, and it is five parts.

### Run one before you build one

Before you read another word, run a real harness and watch it work.

```
npm install -g @mariozechner/pi-coding-agent
pi
```

Give it a task. Watch it call a tool, get the output, and decide what to do
next. That back and forth is the whole thing. The five parts below are what
made it happen.

Most people skip straight to tools and a loop, get something working, then hit
a wall they cannot name. It is almost always one of these five parts they
never built on purpose. Each part has a prompt. Run it against whatever agent
you use, Claude Code, Codex, pi.

> [earendil's "What is a harness"](https://earendil.com/posts/what-is-a-harness/)
> is the clearest writeup of this, and worth reading. It names four parts. I
> count the same four plus memory, because deciding what the model *remembers*
> and what it *sees each turn* is its own job. More on that in part 5.

### 1. System prompt

Instructions injected on every single turn. First-day instructions for a new
hire, not documentation.

People write essays here. The model does not need your philosophy. It needs
rules it can follow when it is halfway through a task and running low on room.

```
Write the system prompt for my harness. Constraints:
- Rules, not prose. Each line should be something the model can obey or violate.
- No motivation, no tone-setting, no "you are a helpful assistant".
- State what it must never do before what it should do.
Then show me which lines would still matter if the model only read half of it.
```

### 2. Tools

What the model can actually do. The harness describes the tool. It does not
decide when to use it, the model does, by reading your description.

That last part is the whole game. A vague description is a broken router.

```
Add tool calling to my harness. Start with 3 tools, not 10.
For each: name, one-line description, JSON schema for the arguments.
Then adversarially review my descriptions: for each pair of tools, tell me
what question would make the model pick the wrong one. Fix the descriptions
until you cannot.
```

### 3. Agentic loop

Model picks an action, you execute it, you feed the result back, repeat until
done. That is it. Every agent framework you have heard of is this loop with
opinions bolted on.

Build it yourself once before you use one. It is about forty lines, and after
that the frameworks stop looking like magic.

```
Implement the agentic loop by hand, no framework.
Model decides an action -> I execute it -> result goes back -> repeat.
Show me the exit conditions explicitly: done, max turns, repeated failure,
model asking for something that does not exist.
Do not add retries, planning, or sub-agents yet. I want the bare loop.
```

### 4. Translation layer

The seam that lets the same harness run on Anthropic, OpenAI, or a local
model. Build it and you own your harness. Skip it and you have written an
application for one vendor.

```
Put a translation layer between my loop and the model API.
One interface my loop calls; per-provider adapters behind it.
Handle the parts that actually differ: tool call format, streaming events,
stop reasons, token accounting.
Then swap the provider and prove the loop did not change.
```

### 5. Memory, which is really context management

Your agent runs `cat` on a 50,000 line file. What reaches the model? All of
it? The first thousand lines? The first thousand plus a warning that it was
truncated? Nothing, and an error telling the model to grep instead?

None of the four parts above answer that. The loop feeds a result back, but
deciding *what the result is* is a different job from deciding *whether to
continue*. Conflating them is how you end up with a loop that works fine and
a model that drowns.

People call this "memory" and think it means what persists across turns. That
is one case. Truncation, ranking, and what gets rendered each turn are the
rest. It is all one job: managing what fills the context window.

I have [written about why this decides outcomes](/2026-06-21-smart-model-bad-context/)
more than model choice does. It is the part people skip and the part that
bites hardest.

```
Add context management to my harness. For every tool result, decide:
what reaches the model, what gets truncated, what gets summarised, what
gets dropped with a note telling the model how to get the rest.
Show me the token cost per turn before and after.
Then make it fail loudly: I want to see it warn, not silently truncate.
```

### All five, in one flow

Here is a pentest harness with every part doing its job. The task comes in,
the harness feeds the model context, the model calls a tool, a guardrail
checks it, the sandbox runs it, the result becomes the next turn's context,
and the loop repeats until the objective is met.

![A pentest harness in action: a human gives the task, the harness passes context to the model, the model requests a tool call, a guardrail checks it, the sandbox executes it, and the result feeds the next turn until the agent has proven a foothold and writes a report.](/assets/img/learn/harness-pentest-flow.png)

Read the colors: the human owns the task and the approval, the model owns the
tool choice, the harness owns the loop and the context, the sandbox owns the
side effects. The exploit only fires after a human approves it and the
guardrail confirms it is in scope. That approval gate is the loop and the
system prompt working together. The `findings.md` at the end is the whole
point: the environment is where effects actually happen.

### Start from pi, not a framework

Use [pi](https://github.com/badlogic/pi-mono). It is the most minimal open
source harness I have found that still has the right building blocks, which
makes it something you can read in an afternoon and then build on.

Build your own on top of it. A research harness, a development harness, a
malware research harness. The parts above are the same every time. What
changes is the system prompt, the tools, and what you let through.

```
I want to build a custom harness on top of pi for <your use case>.
Read pi's source first and tell me which of the five parts it already
handles and which I have to write myself.
Then start with the system prompt and tools. Nothing else yet.
```

### Reference

- [What is a harness](https://earendil.com/posts/what-is-a-harness/) by earendil, the four part version
- [pi](https://github.com/badlogic/pi-mono), the harness to build on
- [A smart model doesn't make up for bad context](/2026-06-21-smart-model-bad-context/), why part 5 matters
