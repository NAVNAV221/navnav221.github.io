---
layout: post
permalink: "/five-walls-context-layer/"
title: "The Five Walls of a Context Layer"
subtitle: "Building your own context layer for AI agents is harder than it looks - starting with the one wall nobody sees first."
share-title: "The Five Walls of a Context Layer"
share-description: "A new Medium series on why building your own context layer for AI agents is hard. Part 1: knowledge graph and ontology, entity resolution, built-in vs custom entities, and why an LLM wiki can't hold your company's knowledge."
categories: AI
cover-img: /assets/img/naboo/five-walls-context-layer.png
thumbnail-img: /assets/img/naboo/five-walls-context-layer.png
tags: [AI, Context, Naboo, Knowledge Graph, Ontology, LLM]
comments: true
---

I started a new series on Medium: **The Five Walls of a Context Layer**.

If you're giving AI agents access to your company's knowledge, you're building a
context layer - the layer between your source systems and your agents. Build it
yourself and you hit five walls:

1. Knowledge graph & ontology
2. Semantic layer
3. Understanding intent
4. Permissions
5. Breathing data at scale

Part 1 is about the first and least visible one: your company has no map of its
own things. The same service is `billing-svc` in Git, "Billing Service" in the
wiki, `BILL` in Jira, and "the billing thing" in Slack - four names, one thing,
and nothing that knows they're one.

In the article I break down what an ontology and a knowledge graph actually are
(the blueprint vs. the building), why built-in and custom entities are two
different problems, and why an LLM wiki cannot hold your company's knowledge once
it grows past a single team.

👉 **[Read Part 1 on Medium](https://navnav221.medium.com/the-five-walls-of-a-context-layer-part-1-knowledge-graph-ontology-87fb5a2ea961)**

\- Nave
