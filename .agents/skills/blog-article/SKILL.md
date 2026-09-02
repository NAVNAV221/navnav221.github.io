---
name: blog-article
description: >-
  Write, rewrite, or review articles for navnav221.github.io. Use when working
  on posts under _posts/, developing an article from personal experience,
  improving its narrative, adding technical examples, or preparing it for
  publication.
---

# Blog article

Write practical, experience-based articles for `navnav221.github.io`.

## Core principle

Do not produce generic AI thought leadership.

The article must contain something the author learned, built, observed, or
changed. If the unique material is missing, interview the author before
rewriting.

Good source material includes:

- A concrete incident
- A surprising failure
- An engineering decision
- Before-and-after evidence
- A limitation that remains unsolved
- Something the author would build differently today

## Before writing

1. Read the entire target article.
2. Read related posts linked from it.
3. Inspect any repository the article describes.
4. Identify:
   - Intended audience
   - Central claim
   - Unique firsthand material
   - Claims requiring confirmation
   - Names or details that must remain private
5. Ask focused questions for missing facts.

Do not ask for information already present in the article or repository.

## Interviewing

Prefer questions that produce scenes and evidence:

- What happened?
- What failed?
- What did the system do next?
- What did the human still need to do?
- How long did it take before and after?
- What was enforced in code?
- What remains unresolved?
- What was the author's role?

Ask a small batch of high-value questions at a time.

Do not turn estimates into measured facts. Preserve qualifications such as
"roughly," "eventually," and "the team estimated."

## Article structure

Prefer this flow:

1. Open with the strongest concrete event.
2. Explain why the result was possible.
3. State the article's main argument.
4. Introduce only the concepts needed to understand it.
5. Give a concrete example for every concept.
6. Include failures and unresolved problems.
7. Explain what the author would change today.
8. End with practical resources or next steps.

Add a linked table of contents to long articles.

## Writing style

- Use direct, concise language.
- Keep paragraphs short.
- Prefer concrete nouns and verbs.
- Explain technical terms on first use.
- Use first person for personal experience.
- Use "we" for work completed by a team.
- Never imply the author built a team project alone.
- Separate real production examples from hypothetical examples.
- Label generic examples as illustrative.
- Avoid hype, filler, and motivational conclusions.
- Do not use em dashes. Use commas, parentheses, colons, or regular hyphens.
- Preserve the term "AI Harness teammate" when referring to the shared agent.
- Use lowercase "harness" when referring to the general concept.

## Examples

Every major concept should have an example.

Clearly distinguish between:

- **Tool:** What action the model can perform.
- **Skill:** How the team performs a specific task.
- **System prompt:** Stable session-level behavior.
- **Guardrail:** A rule enforced regardless of model behavior.
- **Memory:** Stored knowledge and the process for selecting relevant context.
- **Reflection:** A reviewed lesson promoted from a session into durable
  documentation, memory, or a skill.

Never fabricate an internal implementation. If details are unavailable, use a
generic example and label it as such.

## Prompt blocks

Do not add "Run this prompt" blocks by default.

A prompt can fail when the reader lacks the required harness, tools, repository
context, or permissions. Prefer:

- Build checkpoints
- Implementation steps
- Code examples
- Evaluation criteria
- Questions the reader should answer

Only include an executable prompt when its prerequisites are explicit and the
article remains useful without running it.

## Privacy and security

Before publishing:

- Remove customer, company, employee, and internal product names unless the
  author explicitly approves them.
- Remove UUIDs, channel names, repository names, queries, and identifying data.
- Do not disclose private incident details.
- Describe security incidents only at the level approved by the author.
- Distinguish an observed incident from a hypothetical risk.

## Repository links

When referencing a public repository:

1. Inspect its current code and README.
2. Describe what it does today, including limitations.
3. Do not imply that a public template is the production system.
4. Link to both the relevant article and repository when useful.

## Jekyll requirements

Preserve valid frontmatter:

```yaml
---
layout: post
permalink: "/learn/example/"
title: Article title
subtitle: Concrete reason to read it.
share-title: "Article title"
share-description: "Accurate summary without unsupported claims."
categories: Learn
tags: [AI, Learn]
thumbnail-img: /assets/img/learn/example.png
comments: true
---
```

Do not edit generated files under `_site/`.

## Final review

Check:

- Does the opening contain a real event?
- Is the unique insight clear?
- Does every major concept have an example?
- Are human review and approval still visible?
- Are estimates represented honestly?
- Are real and hypothetical systems clearly separated?
- Is private information removed?
- Does the article work without running a prompt?
- Are there any em dashes?
- Do links and table-of-contents anchors work?

Run:

```bash
git diff --check
rg -n "—" _posts/
bundle exec jekyll build
```

Report the changed file, validation result, and local article URL. Do not commit
or push unless explicitly requested.
