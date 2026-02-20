# NLSpec Grounding Document

## What This Is

A grounding reference that establishes what natural language
specifications (NLSpecs) are, what makes them effective, and how to
author, implement, and evaluate them. It is written for both human
engineers and AI agents.

## Version

0.2.1

## Provenance

The NLSpec concept and term originate from StrongDM, whose work
on natural language specifications is prior art for this document.

This document was developed independently and in parallel, arriving
at many of the same principles through convergent engineering
practice. It uses StrongDM's framing as a starting point and extends
it to address contexts their original work did not cover —
particularly iterative and conversational development workflows,
agent-oriented implementation, and the relationship between
exploratory implementation and specification authorship.

The result shares lineage with StrongDM's NLSpec concept but carries
its own emphases and extensions. The NLSpec name is retained as the
term of art for this class of artifact.

## Document Structure

The grounding document (`nlspec.md`) is the primary artifact. It is
self-contained. This README provides metadata and context that do not
belong in the document body.

## Relationship to Operational Documents

This grounding document defines *what specs are*. Operational
documents (such as project-specific code maxims, agent instructions,
or team process docs) define *how specs are used in a specific
context*. Operational documents should reference this grounding
document for philosophical and definitional content rather than
re-deriving it.
