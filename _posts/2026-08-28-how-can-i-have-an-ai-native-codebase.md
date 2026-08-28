---
layout: post
permalink: "/learn/ai-native-codebase/"
title: How can I have an AI Native Codebase?
subtitle: Four layers that let an agent walk into your repo cold and be useful on the first try.
share-title: "How can I have an AI Native Codebase?"
share-description: "An agent opening your repo cold repeats the same mistakes every session. Orientation, capability, enforcement, and self-improvement, so it does the right thing the first time."
categories: Learn
tags: [AI Native, Agents, Codebase, Claude, Learn]
comments: true
---

An agent opening your repo for the first time is a new hire on day one, except
it forgets everything by tomorrow. Give it nothing and you get the same three
problems every session:

- **It does not understand the codebase.** Opens a PR that ignores a component
  you already have. Writes a new class when one already owns that job.
- **It makes your decisions.** The exploration you should have done, it did for
  you, in the exact place your brain was supposed to be.
- **It does not understand the team.** Not the day to day, not what a customer
  is, not the bigger picture the work sits inside.

An AI native codebase fixes this in the repo itself, so the fix survives the
session. Four layers.

![The four layers of an AI native codebase: orientation (where am I?), capability (what can I do?), enforcement (what happens whether I remember or not?), and self-improvement (what should I do better next time?).](/assets/img/learn/ai-native-codebase.png)

### 1. Orientation: where am I?

The map the agent reads before touching anything. `CLAUDE.md` at the root, a
`/docs/` folder with the architecture and the starting points. This is what
stops it rediscovering your repo from scratch every time.

```
Read my repo and write the CLAUDE.md a new agent needs before its first change.
What each top-level area is, which component owns what, and the three mistakes
someone new always makes here. Rules it can follow, not prose.
```

### 2. Capability: what can I do?

Recipes for the tasks that repeat. Releasing a version, writing tests in your
conventions, the workflow you explain to every new hire. `.claude/skills/` for
the recipes, `.claude/agents/` for focused sub-agents.

```
Find the tasks I repeat with an agent: release, tests in our conventions, and
one more. Write each as a skill: when to use it, the exact steps, the traps.
Encode the convention so it stops being something I re-explain every time.
```

### 3. Enforcement: what happens whether I remember or not?

Orientation and capability only work if the agent reads them. Enforcement runs
regardless. `.claude/hooks/` and `settings.json` fire on events: block the
thing you never want, run the check you always forget.

```
Set up hooks that fire whether or not the agent read the docs.
Block the edits I never want, run the check I always forget before a commit.
The guardrail should not depend on the agent choosing to remember it.
```

### 4. Self-improvement: what should I do better next time?

The one people skip, and the one that compounds. When something goes wrong,
the lesson goes back into the repo so it stops recurring. Suffer once, not
twice. A `/docs/` note, a self-reflection step, a way to promote a lesson into
the rules.

```
Add a self-reflection step after a change: what broke, what I learned, where
that lesson belongs. Then promote it into the docs or a hook, so the next
session does not repeat it. I want to suffer a mistake once, not twice.
```

### Start from the template

[ai-native-codebase](https://github.com/NAVNAV221/ai-native-codebase) is a
copy-paste starting point with all four layers stubbed out. It is Claude
shaped, so the paths are `.claude/`, but the four questions are the same for
any agent: where am I, what can I do, what runs no matter what, and what do I
do better next time.

```
Take the ai-native-codebase template and adapt it to my repo.
Start with orientation: a real CLAUDE.md for this codebase, not the stub.
Then one skill I actually repeat. Nothing else yet.
```

### Reference

- [ai-native-codebase](https://github.com/NAVNAV221/ai-native-codebase), the template
- [How can I build a Harness?](/learn/harness/), the agent this repo is for
