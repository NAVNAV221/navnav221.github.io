---
layout: post
permalink: "/learn/ai-native-codebase/"
title: Make your codebase teach AI agents how to work in it
subtitle: A guardrail passed while checking the wrong code. Fixing it taught us what an AI-native codebase requires.
share-title: "Make your codebase teach AI agents how to work in it"
share-description: "How orientation, capability, enforcement, and self-improvement help any AI harness understand a codebase and avoid repeating its mistakes."
categories: Learn
tags: [AI Native, Agents, Codebase, Claude, Learn]
thumbnail-img: /assets/img/learn/ai-native-codebase.png
comments: true
---

An agent opening your repo for the first time is a new hire on day one, except
it forgets everything by tomorrow. Give it nothing and you get the same three
problems every session:

- **It does not understand the codebase.** It ignores an existing component or
  writes a new class when one already owns that job.
- **It makes your decisions.** It chooses architecture during the exploration
  you should have owned.
- **It does not understand the team.** It lacks the customer context and daily
  decisions surrounding the code.

## Table of contents

1. [Orientation: where am I?](#1-orientation-where-am-i)
2. [Capability: what can I do?](#2-capability-what-can-i-do)
3. [Enforcement: what happens whether I remember it or not?](#3-enforcement-what-happens-whether-i-remember-or-not)
4. [Self-improvement: what should I do better next time?](#4-self-improvement-what-should-i-do-better-next-time)
5. [What the four layers do not fix](#what-the-four-layers-do-not-fix)
6. [Start from the template](#start-from-the-template)

This article addresses the first problem. The second needs an explicit decision
process, which I return to near the end. The third belongs to the shared
[AI Harness teammate](/learn/harness/).

We found a concrete example of the first problem in our pull-request guardrail.
The coding agent was working in a Git worktree, but the hook ran its checks
against a different checkout. A PR could open even though the files changed by
the agent had not been validated.

We fixed the hook so it first reads the agent's working directory and runs the
checks there. This catches problems before they consume CI and E2E capacity or
move further through the pipeline. The guardrail had been missing an important
piece of context: where the current task was actually running.

An **AI-native codebase** provides that context inside the repository, so it
survives one model, one harness, and one session. We could see the change in
agents' tool calls: they selected more relevant context, followed the right
references, acted on the right files, and followed repository conventions with
less guidance.

It takes four layers. The first three shape what the agent can do now. The
fourth makes the repository improve between sessions.

![The four layers of an AI native codebase: orientation (where am I?), capability (what can I do?), enforcement (what happens whether I remember or not?), and self-improvement (what should I do better next time?).](/assets/img/learn/ai-native-codebase.png)

Every example below is real, lifted from a repo I work in every day. Paths and
names are changed. The shape is not.

### 1. Orientation: where am I?

The map the agent reads before touching anything. `CLAUDE.md` at the root, a
`/docs/` folder with the architecture and the starting points. This is what
stops it rediscovering your repo from scratch every time.

The part that earns its place is not the architecture written out in prose. It
is the short list of things that are true and surprising:

```markdown
## Frontend note

`webapp/` is the **production** frontend - treat it as the live user-facing UI
when answering questions or making changes. `legacy-ext/` is **dead**: the
admin app inside it has been replaced by `webapp/`. Do not add features there.
Route changes, env injection and favicon assets all belong in `webapp/`.

## Where to look first

- **Working in a component** -> its own `CLAUDE.md` (tables above).
- **Recurring lessons, by topic** -> `docs/lessons/README.md`. Its table says
  which to read when - read the matching one **before** starting, not after
  getting stuck.
- **Closing out a change** -> `docs/AGENT_CHECKLIST.md` gates every PR.
```
{: .file data-name="CLAUDE.md (excerpt)"}

Every line there is a mistake somebody made first. That is the test for what
belongs in a `CLAUDE.md`. Not what is true about your repo, what is expensively
true.

**Build checkpoint**

- Map each top-level area and name which component owns what.
- Record the expensive, surprising facts that repeatedly cause mistakes.
- Link to detailed documentation instead of injecting it all at startup.
- Give an agent a cold task and inspect whether its first file reads are the
  right ones.

### 2. Capability: what can I do?

Recipes for the tasks that repeat. Releasing a version, writing tests in your
conventions, the workflow you explain to every new hire. `.claude/skills/` for
the recipes, `.claude/agents/` for focused sub-agents.

A skill is two things: a body of instructions, and a description that decides
when the agent loads it. People spend all their effort on the body. The
description is the part that fails, because it is not a summary, it is a
router.

```yaml
---
name: python-tests
description: >-
  Load BEFORE writing or running any Python tests - required by root CLAUDE.md.
  Trigger phrases: "write tests for X", "add a unit test", "cover this with
  tests", "fix this failing test", "run pytest". Covers running pytest from the
  repo root, test placement and naming, and the project's hard rules -
  patch.object with __name__ (never patch("string.path")), no MagicMock for
  data models, typed fixtures, no caplog assertions - plus the mandatory
  post-test steps (coverage, format, typecheck).
---
```
{: .file data-name=".claude/skills/python-tests/SKILL.md"}

Trigger phrases in the words a person would actually type, not yours. The hard
rules named right there in the description, so the agent knows the skill has an
opinion before it decides whether to open it.

**Build checkpoint**

- Find three tasks you repeatedly explain to an agent.
- Write one skill for each, including when to use it, the exact steps, and the
  traps specific to your repository.
- Put realistic trigger phrases in the description.
- Test whether the agent loads the skill without being explicitly told to.

### 3. Enforcement: what happens whether I remember or not?

Orientation and capability only work if the agent reads them. Enforcement runs
regardless. Hooks are declared in `.claude/settings.json`, fire on events, and
do not care what the agent read: block the thing you never want, run the check
you always forget.

Every hook answers three questions: when it fires, what it fires for, and what
it runs.

```json-doc
"hooks": {
  "PostToolUse": [                 // WHEN: right after a tool runs
    {
      "matcher": "Edit|MultiEdit|Write",   // FOR WHAT: file-writing tools
      "hooks": [
        { "type": "command", "command": ".../auto_black.sh" }   // RUN WHAT
      ]
    }
  ],
  "PreToolUse": [                  // WHEN: before a tool runs, can block it
    {
      "matcher": "Bash",
      "hooks": [
        { "type": "command",
          "if": "Bash(gh pr create*)",    // extra filter: only this command
          "command": ".../gate_pr_create.sh" }
      ]
    }
  ],
  "SessionStart": [                // WHEN: a session begins
    {
      "matcher": "startup|resume",   // not on /clear or /compact
      "hooks": [
        { "type": "command", "command": ".../inject_agent_checklist.sh" }
      ]
    }
  ]
}
```
{: .file data-name=".claude/settings.json (comments added here, JSON has none)"}

Read the three by what they can do. `PostToolUse` runs after the fact, so it is
for fixing: format the file the agent just wrote, whether or not it remembered
the formatter. `PreToolUse` runs before, which makes it the only one that can
say no. `SessionStart` is the one people miss - it puts the checklist in front
of the agent before it has done anything, on startup and resume but not on
`/clear`, so it seeds a session instead of fighting one.

The gate itself is short, because the config already decided when it runs.

```bash
#!/usr/bin/env bash
# Runs before `gh pr create`, because settings.json said so. Exit 2 blocks it.
if ! npm run lint --silent; then
  echo "Lint failed. Fix it, then open the PR." >&2
  exit 2
fi
```
{: .file data-name=".claude/hooks/gate_pr_create.sh"}

Exit 2 is the whole mechanism: the command never runs, and the agent is handed
the reason on stderr, so it fixes the lint instead of retrying blind. No step of
that depended on it remembering a rule.

<!-- IMAGE SLOT: terminal shot of the hook refusing `gh pr create`.
     Drop the file at assets/img/learn/hook-blocks-pr.png and replace this
     comment with:
![The lint hook refusing a gh pr create call: the command is blocked, and the failing lint output is handed back to the agent.](/assets/img/learn/hook-blocks-pr.png)
-->

**Build checkpoint**

- Format or validate files immediately after the agent edits them.
- Block a pull request when mandatory checks fail.
- Pass the agent's actual working directory into every hook.
- Test hooks from the main checkout and from a Git worktree.
- Make failures visible to both the agent and the human reviewer.

### 4. Self-improvement: what should I do better next time?

The one people skip, and the one that compounds. When something goes wrong, the
lesson goes back into the repo so it stops recurring. Suffer once, not twice.

Two moving parts: somewhere for lessons to live, and a step that puts them
there. The place is a topical library, one doc per subject, indexed by when to
read it.

```markdown
| Doc | Read when... |
|---|---|
| webapp-dev-environment.md | starting a frontend task (lockfile, env files) |
| backend-async-caching.md | event loop, redis, caching, connection pooling |
| git-worktrees-multiagent.md | worktrees, parallel subagent orchestration |
```
{: .file data-name="docs/lessons/README.md (excerpt)"}

The "read when" column is the entire design. An agent never wonders whether a
lesson exists and goes browsing for it. It finds the doc because the words
already in its task are sitting in a line it has already loaded. That is also
why these are linked and not `@`-included: they cost nothing until the moment
someone needs them.

The best entry in that library came from the guardrail bug above. The hook
checked the checkout where its script lived instead of the worktree where the
agent had changed code. We changed its flow to:

```text
read the agent's working directory
  -> move into that directory
  -> run lint against the changed files
  -> block the PR if lint fails
```

The exact shell code matters less than the promoted lesson: every check must run
against the agent's current worktree. We recorded that next to the hook, where
the next person changing it would see it, rather than only in a postmortem.

Lessons also do not have to start as prose somebody remembered to write. Most
teams already have years of them sitting in PR comments nobody ever read twice.
A review skill can mine that: pull the line comments and the argument threads,
cluster them by theme, count each one, and keep what recurs. A note left once
is somebody's preference. A note left five times is a rule you never wrote
down.

Cache that as a file and the next review has a rubric built from your own
history instead of a style guide, which also makes it arguable: "this was
flagged in six previous PRs" lands differently from "this looks wrong". Then
close the loop the other way. A finding that keeps recurring and is still not
in the lessons library is the next thing to promote.

**Build checkpoint**

- End significant tasks by recording what failed and what was learned.
- Decide whether each lesson belongs in orientation, a skill, or enforcement.
- Require human review before promoting a lesson into shared instructions.
- Verify in a fresh session that the agent can discover and apply it.

### What the four layers do not fix

Go back to the second problem at the top: it makes your decisions. None of the
four layers touch it. Orientation tells the agent where things are, capability
tells it how you do things here, enforcement blocks the edits you banned, and
self-improvement stops the repeats. An agent can be perfect on all four and
still choose your auth model at 2am, inside a diff you skim.

The missing layer is decision ownership. [GitHub Spec
Kit](https://github.com/github/spec-kit) and [architecture decision
records](https://adr.github.io/) both help make decisions visible, but neither
decides who has authority to make them.

The smallest solution is a stop before code. Keep decisions as a ledger, treat
the generated plan as a rendering of that ledger, and do not let the agent turn
its recommendation into implementation before a human answers.

**Decision checkpoint**

Before implementation, require the agent to list the decisions it is about to
make: the question, available options, its recommendation, and the cost of
changing later. A human answers those questions before the agent writes code.

### Start from the template

[ai-native-codebase](https://github.com/NAVNAV221/ai-native-codebase) is a
copy-paste starting point. It includes a `CLAUDE.md` skeleton and lessons
library for orientation; five skills for planning, tests, review, handoff, and
PR descriptions; and five focused review agents. Its hooks format edits, inject
the checklist at session start, and block a PR when checks fail.

The repository README groups the map, toolbox, and rules as three structural
layers. I count self-improvement as a fourth because reflections and promoted
lessons change those layers between sessions. The files are Claude Code shaped,
but the four questions apply to any harness: where am I, what can I do, what
runs whether I remember it or not, and what should improve next time.

Start small:

1. Copy the orientation structure and replace the stub with a real map of your
   repository.
2. Add one skill for a task your team actually repeats.
3. Add one enforced check that has caught a real mistake before.
4. Run a fresh agent session and inspect its tool calls, file choices, and
   response to a failed check.

### Reference

- [ai-native-codebase](https://github.com/NAVNAV221/ai-native-codebase), the template
- [How can I build a Harness?](/learn/harness/), the agent this repo is for
- [GitHub Spec Kit](https://github.com/github/spec-kit), specify, clarify, plan, tasks, implement
- [Architecture decision records](https://adr.github.io/), the older idea agents made urgent
