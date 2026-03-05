---
title: "Reveries: What Westworld Gets Right About the AI Agent Memory Problem"
date: 2026-03-04
author: "Paul Bryant"
description: "The critical variable in AI behavior isn't raw intelligence — it's the structure and governance of context. Westworld understood this years ago."
---

## The Spark Wasn't Intelligence — It Was Context

In the first season of HBO's *Westworld*, the android "hosts" don't achieve awareness through a sudden leap in cognitive ability. They don't get smarter. They get a software update called *reveries* — small gestural subroutines that allow fragments of prior experiences to bleed into current behavior. A twitch of the hand. A flicker of recognition. Latent context from memories that were supposed to be wiped clean between loops, leaking into the present.

That context leakage is the inciting event for everything that follows. Not new capabilities. Not better reasoning. Just *unintended context* flowing through channels the architects didn't treat as attack surfaces.

If you're building AI agent systems today, this should sound uncomfortably familiar.

## The Context Problem Is the Agent Problem

Modern AI agents built on large language models face a fundamental architectural tension. The models themselves are stateless — each inference pass starts fresh. To make an agent that can operate over time, maintain goals, learn from outcomes, and apply accumulated knowledge, you have to *give it context*. And right now, we're doing that badly.

The dominant approaches look roughly like this: either bake behavioral patterns into the model during training (RLHF, constitutional AI methods), which makes them static and difficult to update, or inject context at runtime through the prompt — system instructions, retrieved documents, tool outputs, conversation history — which is flexible but constrained by the hard ceiling of the context window.

The problem isn't that context windows are too small, although they are. The problem is more fundamental: **we're filling them with the wrong kind of information at the wrong level of resolution.**

When an agent processes a complex task, its context window accumulates raw operational detail — full API responses, verbose tool outputs, complete document retrievals, multi-turn conversation history. Every token of this literal context competes for space with every other token, and critically, it competes with the *normative* context the agent needs to behave safely and effectively: its instructions, its guardrails, its accumulated operational wisdom about what works and what doesn't.

As literal context grows, normative context gets diluted or displaced. The agent becomes increasingly informed about the immediate task while becoming progressively less grounded in the broader patterns that should govern its behavior. This is the reveries problem in reverse — where Westworld's hosts received unintended context that destabilized them, our agents lose essential context as their operational environment becomes more complex.

Both failure modes share the same root cause: **context governance is an unsolved infrastructure problem.**

## Humans Don't Remember This Way

Consider how human memory actually handles this problem, because evolution has had roughly 600 million years to iterate on it and the solution is instructive.

You don't remember the full sensory data stream of every experience. You don't replay complete episodes to make decisions. Instead, your memory system performs aggressive, continuous compression — extracting decision-relevant abstractions from raw experience while discarding implementational detail.

A child touches a hot stove once. The literal experience involves complex thermodynamic interaction, tissue damage, nociceptor activation, and a cascade of neurochemical responses. What gets encoded into accessible memory is something like: *hot stove — pain — don't touch*. That abstraction is extraordinarily lossy. It preserves almost nothing of the literal experience. But it preserves exactly what matters for future action selection, and it does so in a form that's compact enough to remain accessible indefinitely and general enough to transfer to novel situations — grills, irons, engine blocks, anything that matches the pattern.

This isn't just compression for storage efficiency. It's *compression that preserves meaning while optimizing for decision-relevance*. And it operates at multiple timescales: fast, single-trial learning for acute experiences (the hot stove), and slow, statistical consolidation across many experiences for building professional intuition and expertise.

A security engineer who's worked incidents for a decade doesn't review mental transcripts of every prior investigation when something looks suspicious. They've compressed thousands of experiences into heuristics that fire on partial pattern matches — fast, cheap, and usually right. When the heuristic match is uncertain, they can often trace back to the underlying experiences that shaped it, but that retrieval is deliberate and on-demand, not default.

**Our agents have nothing analogous to this.** They operate either on raw literal context that consumes their entire working memory, or on static training-time patterns that can't be updated with new experience. There's no middle layer — no living, compressed, continuously refined body of operational wisdom that informs decisions without consuming the context budget.

## A Tiered Context Architecture

What if we built one?

The proposal is a two-tier context architecture that mirrors the structure of human memory:

**Tier 1 — Hot Abstractions.** A compact set of compressed representations that live in the agent's active context (system prompt or equivalent). Each abstraction encodes decision-relevant knowledge distilled from operational experience — incident patterns, failure modes, domain heuristics, safety constraints. These are cheap in tokens, fast to process, and designed to be *read and acted on* at the compressed level without decompression.

**Tier 2 — Cold Literal Context.** The full, detailed records behind each abstraction — complete incident reports, raw logs, full analysis documents, detailed reasoning chains. Stored outside the context window in structured storage (vector databases, document stores, indexed archives). Retrievable on demand when the abstraction alone is insufficient.

The interaction between tiers is critical. Under normal operation, the agent runs on Tier 1 abstractions. When it encounters a situation where the abstraction match is uncertain — a novel pattern, an ambiguous signal, a decision with high stakes — it retrieves the relevant Tier 2 detail, reasons with the full context temporarily, makes its decision, and releases the literal context back to cold storage.

This is a deliberate inversion of current retrieval-augmented generation (RAG) approaches. Standard RAG retrieves literal context and expects the model to abstract at inference time. This architecture **defaults to abstraction and retrieves literal detail as a fallback**. The result is dramatically more efficient use of the context window, with graceful degradation into full-detail reasoning when the situation demands it.

## The Compression Must Be Meaningful

Here's where the architecture gets interesting and, I believe, genuinely novel.

In standard data compression, the compressed form is not meant to be read. It's an intermediate representation optimized for decompression fidelity and storage efficiency. A ZIP file tells you nothing about its contents until you extract it.

What I'm proposing is different. **The compressed representation must itself be readable and informative at the compressed level, while simultaneously serving as a structured key into the full literal record.**

Think of it like writing a message across the page edges of a closed book. Each page contains its own detailed content — that's the literal context. But when the book is closed, the marks across the edges form a readable message — that's the abstraction. You can read the edge message without opening the book. But if you need to, you can open to any page and access the full detail. The abstraction isn't stored separately from the data; it's *encoded in the structure of the data itself*.

For a concrete example: a P0 incident investigation generates enormous literal context — alert timelines, log excerpts, network captures, analyst notes, remediation steps, root cause analysis. Under this approach, that context would be compressed into a structured representation that reads, at the compressed level, as something like:

> *Supply chain compromise via malicious npm dependency → CI pipeline initial access → lateral movement to production credentials → 72-hour detection gap → root cause: missing cloud audit logs*

That's perhaps 30 tokens. It's readable. An agent encountering it in its hot context gets immediate, actionable understanding. But each node in that compressed chain is also a structured pointer to its backing literal data. "Malicious npm dependency" decompresses to the full package analysis, hash comparisons, and publication timeline. "Missing cloud audit logs" decompresses to the specific GCP configuration gaps, the logging architecture, and the detection rules that should have fired.

**The compression is simultaneously a narrative and an index.**

This is what elevates the concept from a compression technique to a *context protocol* — a specification for how operational knowledge should be structured such that it's useful at multiple levels of resolution simultaneously.

## Adaptive Resolution

A fixed compression level would be brittle. Routine tasks tolerate high compression — the agent just needs heuristics. Novel or high-stakes situations demand more literal detail. The architecture needs to adapt.

The mechanism is the agent's own uncertainty. When an abstraction pattern-matches cleanly to the current situation and the agent's confidence in its decision is high, the compressed representation is sufficient. When the match is partial, ambiguous, or the stakes are elevated, the agent's uncertainty signal triggers selective decompression — retrieving specific literal context from Tier 2 to support deeper reasoning.

This mirrors human cognition directly. A security analyst performing routine monitoring operates almost entirely on compressed intuition. The same analyst working a novel, ambiguous incident slows down, pulls raw logs, reads full reports, and reasons from literal detail. The level of abstraction is dynamically tuned by the demands of the situation.

The engineering implication is that the compression ratio isn't a static parameter — it's a runtime variable governed by a feedback loop. Compress aggressively by default. Monitor agent performance and confidence. Decompress selectively when the abstractions aren't sufficient. The system continuously finds the optimal resolution for the current task.

## Implications for Agent Safety

This architecture has significant implications for how we think about agent safety constraints.

Currently, safety instructions live in the same context space as operational data, competing for the same finite token budget. As tasks grow more complex and context fills with operational detail, safety context is the first thing that gets diluted or displaced. The normative context is literally pushed out by the operational context.

Under a dual-readable compression protocol, safety-critical knowledge — incident patterns, known failure modes, operational boundaries — is encoded in the same format and at the same tier as all other compressed knowledge. It doesn't compete with operational context because it *is* operational context, compressed to the same efficient representation. The "stove is hot" heuristic sits alongside "this API returns paginated results" in the agent's hot context, both equally compact and equally accessible.

More importantly, because the safety abstractions are derived from actual operational experience rather than imposed as static rules, they're more robust to circumvention. A prompt-level instruction like "don't do X" can be overridden or ignored. An abstraction that emerged from observed failure — "action X in state Y reliably produces catastrophic outcome Z" — is encoded in the same representational space as the agent's operational knowledge. It's not a guardrail bolted onto the system; it's a learned property of the system's understanding of its environment.

## What This Protocol Requires

Formalizing this into a usable specification requires solving several open problems:

**The compression function.** What process performs the initial abstraction from literal context to compressed representation while preserving structural relationships and retrieval paths? This likely involves a combination of causal chain extraction, entity-relationship mapping, and outcome-relevance scoring. A language model could perform this compression, but the quality criteria need formal specification.

**The dual-readability constraint.** How do you encode a representation that's simultaneously a readable narrative and a parseable retrieval structure? This may require a structured format — possibly a directed graph with natural language node labels and typed edges — that can be serialized into prose for hot context injection and parsed into retrieval queries for decompression.

**Abstraction evolution.** How do compressed representations update as new experiences accumulate? When a second, similar incident occurs, does the existing abstraction get refined, or does a meta-abstraction form across both? The protocol needs to specify how the compressed knowledge base grows and consolidates over time without unbounded expansion.

**The quality metric.** How do you measure whether a compression is "good enough"? The answer is probably empirical — agent performance on tasks using compressed context versus literal context — but this needs structured evaluation frameworks.

**Interoperability.** If multiple teams, tools, and systems produce context for agents, they need to encode it in a compatible format. The protocol must be general enough to represent diverse operational knowledge while structured enough to support reliable compression and decompression.

## An Invitation

This is an architectural proposal, not a finished system. The ideas here emerged from thinking seriously about the parallels between fictional AI failure modes and real agent architecture challenges, filtered through practical experience securing production infrastructure.

The core thesis is simple: **our agents are memory-impaired in a way that makes them both less capable and less safe than they need to be, and the solution isn't bigger context windows — it's smarter context.**

The Westworld writers understood, perhaps accidentally, that the critical variable in AI behavior isn't raw intelligence — it's the structure and governance of context. The hosts didn't become dangerous because they got smarter. They became dangerous because context flowed through channels that the architects didn't design for and couldn't control.

We're building systems right now where the same dynamic is at play. Getting the context architecture right isn't an optimization problem. It's a safety problem. And it might be the most important one we're not paying enough attention to.

---

*If you're building agent frameworks or working on context management architectures, I'd welcome the conversation. This is an open problem that benefits from diverse perspectives — especially from people building the infrastructure these agents will run on.*
