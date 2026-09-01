---
layout: post
permalink: "/learn/ai-native-codebase/"
title: How can I have an AI Native Codebase?
subtitle: Four layers that let an agent walk into your repo cold and be useful on the first try.
share-title: "How can I have an AI Native Codebase?"
share-description: "An agent opening your repo cold repeats the same mistakes every session. Orientation, capability, enforcement, and self-improvement, so it does the right thing the first time."
categories: Learn
tags: [AI Native, Agents, Codebase, Claude, Learn]
thumbnail-img: /assets/img/learn/ai-native-codebase.png
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

The first is what this article is about. The second is the one the four layers
below do not fix, so I come back to it at the end. The third is the
[harness](/learn/harness/).

An AI native codebase fixes the first in the repo itself, so the fix survives
the session. Four layers.

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

```
Read my repo and write the CLAUDE.md a new agent needs before its first change.
What each top-level area is, which component owns what, and the three mistakes
someone new always makes here. Rules it can follow, not prose.
```
{: .prompt data-name="Use this prompt to start using CLAUDE.md in your repo"}

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

```
Find the tasks I repeat with an agent: release, tests in our conventions, and
one more. Write each as a skill: when to use it, the exact steps, the traps.
Encode the convention so it stops being something I re-explain every time.
```
{: .prompt data-name="Use this prompt to turn your repeated tasks into skills"}

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

```
Set up hooks that fire whether or not the agent read the docs.
Block the edits I never want, run the check I always forget before a commit.
The guardrail should not depend on the agent choosing to remember it.
```
{: .prompt data-name="Use this prompt to add hooks that run whether the agent remembers or not"}

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

The best entry in that library came from a bug in the gate above. It linted the
checkout it happened to live in, not the one the agent was working in, so a PR
opened from a worktree got checked against a different branch entirely, printed
"All checks passed!", and exited 0. Here is the same script afterwards:

```bash
#!/usr/bin/env bash
# Runs before `gh pr create`, because settings.json said so. Exit 2 blocks it.

# LESSON: the agent often works in a git worktree, on a different branch than
# the checkout this script lives in. Linting the wrong one passes green and
# tells you nothing. The hook input says which directory it is really in.
cd "$(jq -r '.cwd')" || exit 0

if ! npm run lint --silent; then
  echo "Lint failed. Fix it, then open the PR." >&2
  exit 2
fi
```
{: .file data-name=".claude/hooks/gate_pr_create.sh"}

One line of fix, and a comment sitting in the file the next person will edit
rather than in a postmortem nobody reopens. A guardrail that lies to you is
worse than one you never built, and now nobody has to learn that twice.

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

```
Add a self-reflection step after a change: what broke, what I learned, where
that lesson belongs. Then promote it into the docs or a hook, so the next
session does not repeat it. I want to suffer a mistake once, not twice.
```
{: .prompt data-name="Use this prompt to make a lesson stick between sessions"}

### What the four layers do not fix

Go back to the second problem at the top: it makes your decisions. None of the
four layers touch it. Orientation tells the agent where things are, capability
tells it how you do things here, enforcement blocks the edits you banned, and
self-improvement stops the repeats. An agent can be perfect on all four and
still choose your auth model at 2am, inside a diff you skim.

The answer is a step that runs before the code, and the field agrees on that
much. [GitHub Spec Kit](https://github.com/github/spec-kit) runs specify,
clarify, plan, tasks, implement, with a clarify step whose whole job is to drag
out what the spec left vague. [Architecture decision
records](https://adr.github.io/) are having a revival for the same reason: an
agent that cannot see why something was built a certain way will cheerfully
refactor the reason away.

Both write the decision down. Neither settles who owns it, and that is the one
rule I would add.

Treat the document as a rendering of a decision ledger, not as prose you edit
forward. The decisions are the artifact; the write-up is re-emitted from them
each round, never appended to. The reason is specific. A document you edit in
place freezes around the agent's recommendation: it proposed B, wrote three
sections that assume B, and by the time it reaches you, saying "actually A"
means arguing with a paragraph instead of answering a question. Re-emit
instead, and the recommendation stays a recommendation until you have answered.

That is a whole subject and its own article. The smallest version that works
today is to make the agent stop and put the question in front of you.

```
Before you write any code for this task, list the decisions you are about to
make for me. For each one: the question, the options, your recommendation, and
what it costs to change later. Then stop and wait for my answers.
```
{: .prompt data-name="Use this prompt to get your decisions back before the agent makes them"}

### Start from the template

[ai-native-codebase](https://github.com/NAVNAV221/ai-native-codebase) is a
copy-paste starting point. A `CLAUDE.md` skeleton and a lessons library for
orientation. Four skills for capability: `plan-task`, `code-tests`,
`code-review`, `pr-description`. Hooks that already do something rather than
sitting there as a comment: format on every edit, checklist injected at session
start, and a PR gate that lints the files you changed and blocks the PR when
they come back red. It is Claude shaped, so the paths are `.claude/`, but the
four questions are the same for any agent: where am I, what can I do, what runs
no matter what, and what do I do better next time.

```
Take the ai-native-codebase template and adapt it to my repo.
Start with orientation: a real CLAUDE.md for this codebase, not the stub.
Then one skill I actually repeat. Nothing else yet.
```
{: .prompt data-name="Use this prompt to adapt the template to your repo"}

### Reference

- [ai-native-codebase](https://github.com/NAVNAV221/ai-native-codebase), the template
- [How can I build a Harness?](/learn/harness/), the agent this repo is for
- [GitHub Spec Kit](https://github.com/github/spec-kit), specify, clarify, plan, tasks, implement
- [Architecture decision records](https://adr.github.io/), the older idea agents made urgent
