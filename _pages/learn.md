---
permalink: "/learn/"
layout: page
title: Where should I start?
subtitle: How I learn a new subject, a map of the whole space, and the three things people ask me about most.
share-title: "Where should I start with AI R&D?"
share-description: "Nave Ben Naim on learning AI research and development: build first, break it by running it, a map of the topics that matter, and deep dives on harnesses, search, and AI native codebases."
---

People ask me where to start with AI R&D. This is the answer I keep giving.

### How I actually learn something

This is what I do. Steal it.

![How I learn something new: build first, break it by running it, read the problem not the field, watch the exact thing you need.](/assets/img/learn/how-i-learn.png)

**Build first.** Jump straight into a real project, the thing you actually
want to exist. Not a tutorial project.

**Break it by running it.** Run it before you understand it, and let it fail.
The failure tells you which part you do not understand yet. That is the part
to go read about.

**Read the problem, not the field.** Articles that touch the exact thing you
are stuck on. Not a survey of the landscape.

**Watch the exact thing you need.** Fifteen minutes of someone doing the one
thing. Not a four hour tutorial where the part you need shows up at minute 190.

### The map

Everything worth learning here, and where I am on writing it up. A linked title
means the article exists. The rest is the territory, in the order it tends to
matter.

**Foundations**

- **Models** - small vs big, effort, and when to reach for which. Most of what
  people send to a frontier model is classification, not reasoning.
- **Context engineering** - the right context beats a bigger model. The thread
  running through everything below, and the thing people skip.

**Building agents**

- **[Harness](/learn/harness/)** - the software your model runs inside. Five
  parts, a prompt for each.
- **Skills** - packaged instructions an agent loads only when a task needs them.
  [Claude Code docs](https://docs.claude.com/en/docs/claude-code/skills).
- **Plugins** - bundles of skills, commands, and hooks you install into an agent.
  [Claude Code docs](https://docs.claude.com/en/docs/claude-code/plugins).
- **MCP** - a standard interface so any model can call your tools without a
  bespoke integration each time.
- **Multi-agent** - running sub-agents in parallel, and the harder question of
  when that actually helps instead of just costing more.
- **Guardrails and human in the loop** - approval gates and scope checks before
  an agent does something with real effects.

**Retrieval and knowledge**

- **[Searching methods](/learn/search/)** - lexical, semantic, structured, and
  graph. Which one fits which question.
- **WikiLLM** - an LLM maintained wiki that compounds knowledge, instead of RAG
  re-deriving it on every query.
  [Karpathy's note](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).
  Wikis are not for humans anymore.

**Your codebase**

- **[AI Native Codebase](/learn/ai-native-codebase/)** - a repo an agent walks
  into cold and is useful in on the first try.
- **Self-reflection and self-improvement** - suffer once, not twice. The lesson
  from a mistake goes back into the repo so the next session does not repeat it.

**Knowing it works**

- **Evals** - how do you actually know it works? LLM as judge, regression tests,
  and catching the change that quietly made it worse.

**Plumbing**

- **SSE / Streaming** - the server-sent events under most chat UIs. One
  direction, server to client.
- **Transport** - stdio for a tool running next to the model, HTTP for one
  running as its own service. Pick by where the thing lives.

**Tooling**

- **Claude Code** - the power-user tricks that make the agent actually fast.

### Start with these three

The three that are written, in the order that costs you the least backtracking.
Each gives you something to run, not theory to read.

**[How can I build a Harness?](/learn/harness/)**

- **Why:** the harness is the part of an agent you actually own. Skip it and you
  are configuring a framework you cannot debug.
- **You will earn:** your own harness, and the five parts every agent is made of.
- **Inside:** the five parts, a prompt for each, and how to build on top of pi.

**[How to build the right searching tools?](/learn/search/)**

- **Why:** retrieval decides what the model sees. Everyone reaches for embeddings
  first, and it is usually the wrong call.
- **You will earn:** knowing which of four search methods fits which question.
- **Inside:** four methods, one incident, each blind to what the next one catches.

**[How can I have an AI Native Codebase?](/learn/ai-native-codebase/)**

- **Why:** an agent opening your repo cold makes the same mistakes every session.
- **You will earn:** a repo an agent walks into and is useful in on the first try.
- **Inside:** orientation, capability, enforcement, and self-improvement.

### Who to follow

The people I actually learn from.

- [earendil](https://earendil.com/posts/) - clear writing on harnesses and agents
- [Theo (t3dot)](https://www.youtube.com/@t3dotgg) - builds things on camera, no filler
- [AI Engineer](https://www.youtube.com/@aiDotEngineer) - the conference talks worth your time
- [Matt Pocock](https://www.youtube.com/@mattpocockuk) - TypeScript and AI, sharp and short
