---
layout: post
permalink: "/learn/search/"
title: How to build the right searching tools?
subtitle: Four search methods, one incident, and why reaching for embeddings first is usually wrong.
share-title: "How to build the right searching tools?"
share-description: "Lexical, semantic, structured, and graph search each answer a different question and each fails quietly on the others. When to use which, and where RAG actually stops."
categories: Learn
tags: [Search, RAG, Retrieval, Embeddings, AI, Learn]
thumbnail-img: /assets/img/learn/search-methods.png
comments: true
---

Everyone reaches for embeddings first. It is usually the wrong call, and it
costs you tokens and relevance.

Retrieval decides what the model sees. Get it wrong and the model does its
best with the wrong page: confident, well formatted, and wrong. There are four
methods worth knowing, and each one is blind to something the next one catches.

The prompts run against any agent. The notebooks below are the same four
methods against one real incident, so you can watch each failure happen.

```
git clone https://github.com/NAVNAV221/searching-methods-cookbooks
cd searching-methods-cookbooks && jupyter lab
```
{: .shell}

### Lexical, when they paste the exact string

Someone copies an error code out of the logs. `CONFIGURATION_IS_MISSING`.
BM25 matches the literal token, and nothing beats it when the query is a rare
exact string.

Blind to meaning. Describe the same problem in your own words and it falls
apart.

```
Add lexical (BM25) search to my harness as a tool.
Tokenize so exact compounds survive: CONFIGURATION_IS_MISSING should match as
itself, not shatter into "configuration" which is in half the corpus.
Then show me a query where this ranks a stale doc first, so I see the blind spot.
```
{: .prompt data-name="Use this prompt to add lexical (BM25) search to your harness"}

### Semantic, when they describe it

No error code, just "it won't start after the deploy and it cannot find
something it needs." Every word is common. Lexical drowns. Embeddings match
meaning, so the right section surfaces even though it shares no keywords with
the question.

Blind to exact strings and to freshness. Paste the log line back and lexical
wins again. This is the argument for running both, not for replacing one.

```
Add semantic search with embeddings as a second tool.
Cache the vectors so it runs offline. Then test the same incident two ways:
once with the error pasted, once described. Show me which method wins each,
and by how much. That gap is why you keep both.
```
{: .prompt data-name="Use this prompt to add semantic search to your harness"}

### Structured, when the answer is a count

"For every component: who owns it, how many docs, how many superseded." No
passage contains that answer. It is a join and a count, not a sentence to
retrieve. A small model writes the SQL, you run the SQL.

Vector search cannot do arithmetic over your data. Stuffing the top chunks
into the prompt costs more and gets the count wrong without telling you.

```
Add structured search: a small model turns the question into SQL, I execute it.
Make the model reason before it commits (chain_of_thought as the first field),
and gate the keyword filter behind a query_type, so an enumeration does not get
a text filter bolted on that silently drops rows.
```
{: .prompt data-name="Use this prompt to add structured search to your harness"}

### Graph, when the question is a relationship

"This service is down. What else breaks, and who do I page?" That is a
traversal. Nothing in a retrieved chunk says "depends on" unless you built
something that walks relationships. Seed it from your search hits, then walk a
few hops out.

```
Add graph traversal. Seed it from the top search hits, BFS a few hops,
stamp each node with the hop that reached it so I can see the blast radius.
Then show me the component a 3-hop walk misses, so I know where it stops.
```
{: .prompt data-name="Use this prompt to add graph traversal to your harness"}

### Where RAG actually stops

RAG is retrieve-then-generate. It is good for one thing: fetching something
roughly on topic when you do not care about the relationships between items or
about true relevance. Use it there and it earns its place, usually in
combination with one of the methods above.

Do not trust it to judge which file is actually relevant, or to understand how
two things in your knowledge base relate. It cannot. That is not what it does.
The moment the question is "which of these matters most" or "how do these
connect," RAG alone will hand you a confident wrong answer.

### The point

You do not pick one. You pick the one whose shape matches the question, and a
real system runs all four and routes each question to the right one. The
[cookbooks](https://github.com/NAVNAV221/searching-methods-cookbooks) walk one
incident through all four so you feel each blind spot before you trust the method.

### Reference

- [searching-methods-cookbooks](https://github.com/NAVNAV221/searching-methods-cookbooks), four methods, one incident
- [A smart model doesn't make up for bad context](/2026-06-21-smart-model-bad-context/), why retrieval is the whole game
