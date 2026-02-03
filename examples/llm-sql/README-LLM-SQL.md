# URS Example: llm-sql

This directory contains a **worked experiment** applying the Universal Repository Specification (URS) method to a real codebase (`llm-sql`).

It is not intended to be a polished or current URS for the project.  
Instead, it documents—end to end—**how the URS method behaves in practice**, using real artifacts, real documentation drift, and real LLM outputs.

> **Note on versioning**  
> All artifacts here were generated using a **pre–v1 URS template**. The example is preserved as historical method evidence rather than updated to newer template revisions.

---

## What this example is for

The purpose of this example is to demonstrate and compare:

- a **two-pass URS workflow**  
  (artifact-first derivation → explanatory distillation), and
- a **single-pass URS workflow**  
  (derivation and explanation collapsed into one step),

applied to the *same* repository, with all prompts, outputs, and logs preserved.

The question being tested is simple:

> Does separating forensic truth-finding from explanatory documentation materially improve correctness and clarity?

This directory exists to let the reader answer that question directly by inspecting the artifacts.

---

## What is `llm-sql` (context)

`llm-sql` is a deterministic, read-only natural language query interface over a fixed SQLite dataset.

At a high level, it:
- accepts natural language queries,
- translates them (via an LLM or a stub) into a validated `QueryPlan`,
- compiles that plan into **fixed SQL templates only**,
- executes queries in **read-only mode** against a single table (`posts`),
- and writes durable **session artifacts** describing what was executed and selected.

This makes it a useful URS test case because:
- LLMs influence execution semantics,
- authority must be enforced by code rather than prose,
- documentation drift is present,
- and incorrect assumptions would mislead rather than fail loudly.

All behavioral claims are established in the derive artifacts linked below.

---

## How to read this directory

Think of this directory as an **experiment ledger**, not a tutorial.

Each workflow consists of:
- a prompt (what the LLM was instructed to do),
- an output (what it produced),
- and, where applicable, a process log (how it reports having done the work).

Nothing has been cleaned up or normalized after the fact.

---

## Two-pass workflow (primary experiment)

### Stage 1 — Derive (artifact-first)

- **Prompt:** [`derive.prompt.txt`](./derive.prompt.txt)  
  Instructs the LLM to treat the repository as a hostile witness, derive behavior strictly from artifacts, and record unknowns and contradictions explicitly.

- **Output:** [`derive.output.txt`](./derive.output.txt)  
  The raw, mechanical URS produced by the derive stage. This is evidence, not documentation.

- **Process log prompt:** [`derive.log.prompt.txt`](./derive.log.prompt.txt)

- **Process log output:** [`derive.log.output.txt`](./derive.log.output.txt)  
  Records inspection order, evidence classification rules, handling of conflicts, and scope boundaries. This makes the derive stage auditable.

---

### Stage 2 — Distill (editorial)

- **Prompt:** [`distill.prompt.txt`](./distill.prompt.txt)  
  Instructs the LLM to produce a forward-facing URS using the derive output as authoritative truth, without re-inspecting artifacts.

- **Output:** [`distill.output.txt`](./distill.output.txt)  
  A cleaner, more readable URS that preserves all constraints, unknowns, and contradictions surfaced in Stage 1.

- **Process log prompt:** [`distill.log.prompt.txt`](./distill.log.prompt.txt)

- **Process log output:** [`distill.log.output.txt`](./distill.log.output.txt)  
  This file is intentionally empty. The model completed the distillation task but did not produce the requested log. This outcome is preserved as part of the evidence.

---

## Single-pass workflow (baseline)

- **Prompt:** [`urs.single-pass.prompt.txt`](./urs.single-pass.prompt.txt)  
  Requests a complete URS in one pass, combining artifact inspection and documentation reconciliation.

- **Output:** [`urs.single-pass.output.txt`](./urs.single-pass.output.txt)  
  The single-pass URS, used as a baseline for comparison with the two-pass result.

- **Process log prompt:** [`urs.single-pass.log.prompt.txt`](./urs.single-pass.log.prompt.txt)

- **Process log output:** [`urs.single-pass.log.txt`](./urs.single-pass.log.txt)  
  A concise self-report of the single-pass process. Compared to the derive log, this is notably less granular.

---

## What this example shows

Taken together, these artifacts show that:

- An explicit artifact-first pass reliably surfaces:
  - stale or aspirational documentation,
  - claims not enforced by code,
  - partial implementations,
  - and genuine unknowns.
- Separating derivation from distillation:
  - keeps artifact truth and narrative explanation distinct,
  - reduces accidental invention,
  - and makes contradictions harder to smooth over.
- Single-pass generation tends to blur those boundaries, even when the model is careful.

The example also shows real limits:
- process logging is not guaranteed,
- template adherence is sensitive to revision changes,
- and LLM self-reporting is imperfect.

Those limits are part of the result, not something to hide.

---

## How this fits in the URS repository

This example exists to justify the **two-pass URS method** and to ground later design decisions in concrete evidence.

It should not be treated as:
- a current URS for `llm-sql`,
- a canonical template example,
- or a claim of universal optimality.

It is a preserved experiment.

---

## Summary

This directory demonstrates—using real code, real drift, and raw outputs—why URS separates **deriving what is true** from **explaining what that truth means**.

Everything needed to evaluate that claim is linked above.