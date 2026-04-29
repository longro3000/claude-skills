---
name: project-knowledge-capture
description: Automatically capture decisions, notes, and session reflections for the project knowledge base
---

# Project Knowledge Capture

This skill maintains persistent project knowledge.

It should run whenever important insights or decisions occur.

## Step 1 — Detect Knowledge Worth Saving

Save information when the session includes:

- architectural decisions
- debugging breakthroughs
- non-obvious code understanding
- performance discoveries
- project conventions
- new workflows

## Step 2 — Classify the Knowledge

Decision
→ docs/decisions/

Session insight
→ docs/session-reflections/

Technical understanding
→ docs/notes/

## Step 3 — Write Structured Documentation

Decision template:

# Decision: <title>

Date: <date>

## Context
Why this decision was required.

## Decision
Chosen solution.

## Rationale
Reasoning behind the choice.

## Alternatives Considered

## Consequences

---

Session reflection template:

# Session Reflection

Date: <date>

## Key Discoveries

## Problems Encountered

## Solutions Found

## Architecture Insights

## Future Improvements

---

Technical note template:

# Topic

## Overview

## Key Concepts

## Relevant Files

## Notes