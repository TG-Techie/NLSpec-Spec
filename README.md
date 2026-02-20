# NLSpec Grounding Document

> Link to the experimental nlspec-spec itself: [nlspec-spec.md](nlspec-spec.md)

## What This Is

This project is an experimental definition of **NLSpecs** \[natural language specifications] synthesized from independent efforts. It establishes what they are, what makes them effective, and how to author, implement, and evaluate them. It is packaged as a "**grounding reference**" intended to be shared directly with humans or repackaged for AI systems. Appendices add further discussion and cover common use cases as starting points for repackaging as intuitions for system prompts, skill for spec maintenance, and so on.

### Prior Art

1.  **StrongDM's [attractor](https://github.com/strongdm/attractor)** - Production-quality NLSpecs published open source. StrongDM coined "NLSpec." Their repository contains complete, battle-tested examples and a developed factory methodology.

2. [CodeMaxims.md](Prior%20Art/TG-Techie%20-%20CodeMaxims.md) - Principles evolved during organic exploration spec-first development in a deployed project. Originally conflated the *idea* of specs with the *practice* of using them in one project. The philosophical content fed into this document; operational content (tag conventions, workflows, agent directives) remains project-specific.

3. **The USB Type-C PD Specification** — (To be discussed.) A pre-existing NLSpec from hardware/protocol engineering.

### How This Document Came About

The Code Maxims document grew mid-project from practical necessity: provide definitive intent for what new code sections needed to do — behavior and requirements, not implementation. Over time it accumulated both operational rules for spec-first codebases *and* philosophical arguments for what a spec is and why it matters.

Discovering StrongDM's work provided vocabulary and impetus to separate these concerns. This document captures the definitional content; operational rules should live in project- or agent-specific configuration: in operational documents.

### Relationship to StrongDM's Work

Drafted collaboratively with Claude Opus 4.6, using StrongDM's attractor specs as concrete examples, then generalizing. The result is a **superset**: an explanation of the NLSpec concept that can be distilled into application-specific forms (system prompts, agent skills, etc.).

This project is **not affiliated with StrongDM**. The overlap was discovered after Code Maxims already existed — two independent efforts projecting the same underlying ideas.

StrongDM's work is substantially more mature. Credit for aligned methodology belongs there; divergence may reflect genuine differences in context or philosophy. Content here is substantively derived from their published specs, filtered through independent spec-first experience and (fwiw) differing engineering preferences.

