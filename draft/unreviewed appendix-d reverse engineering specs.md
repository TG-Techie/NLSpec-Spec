## Appendix D: Reverse-Engineering Specs from Existing Systems (Proto / Unsolved)

> **Status:** This appendix frames an open problem. It does not prescribe a methodology. It establishes vocabulary, identifies the core difficulty, and sketches the boundary between what is tractable and what may not be.

-----

### The Problem

The `Intent → NLSpec → Implementation` chain assumes forward construction: intent exists first, a spec is authored, implementation follows. Many real systems were not built this way. They were built incrementally, organically, under time pressure, with intent distributed across commit messages, Slack threads, tribal knowledge, and the code itself. No spec exists. The code is the only authority.

Reverse-engineering a spec means reconstructing the NLSpec from an existing implementation — producing a document from which the system could, in principle, be faithfully recreated. This inverts the generative direction: instead of `NLSpec → Implementation`, we attempt `Implementation → NLSpec`.

This inversion is not symmetric. Information is lost in both directions, but the losses are different and the forward direction is recoverable in ways the reverse is not.

### Three Levels of Extraction

What you can extract from an existing system falls into three categories, each progressively harder and less certain.

**Level 1: Behavioral description.** What does the system observably do? Given these inputs, it produces these outputs. Given this error condition, it returns this error type. This timeout is 30 seconds. This retry policy is exponential backoff with jitter, base 1 second, max 60 seconds, 5 attempts.

This level is tractable. It is what good reverse engineering already produces. Static analysis, dynamic tracing, test suites, and careful code reading yield behavioral descriptions. The output is a *descriptive* document — accurate, potentially exhaustive, but not yet a spec.

**Level 2: Behavioral contract.** What is the system *supposed* to do? Which observed behaviors are intentional requirements and which are incidental artifacts of implementation? Is the 30-second timeout a design decision or did someone pick a round number and nobody revisited it? Is the retry jitter load-bearing (protecting against thundering herds) or cosmetic (someone copied it from a blog post)?

This level is partially tractable. Some behavioral contracts are inferrable: if the system validates an input, that validation is almost certainly intentional. If a public API returns a structured error type, the structure is almost certainly contractual. But many behaviors exist in a gray zone where intent cannot be recovered from the artifact alone. The code tells you *what*. It does not reliably tell you *why*, and without *why*, you cannot distinguish requirement from accident.

**Level 3: Prescriptive specification.** A document that is not merely accurate about the current system but *authoritative over future implementations*. A document from which the system could be recreated, and against which the current implementation could be evaluated for correctness — including the possibility that the current implementation has bugs relative to its own intent.

This level may be fundamentally intractable without access to original intent. A prescriptive spec asserts what the system *should* do. The implementation only tells you what it *does* do. These diverge in exactly the cases that matter most: bugs, workarounds, undocumented behavior that callers depend on, and behavior that is correct-by-accident.

### The Core Difficulty: Hyrum’s Law and the Intent Gap

Hyrum’s Law states: *with a sufficient number of users of an API, all observable behaviors of your system will be depended on by somebody.* This means that in a mature system, the distinction between “intentional behavior” and “accidental behavior” has been eroded by external dependencies. The 30-second timeout that was an arbitrary choice is now a contractual obligation because downstream systems have hardcoded their own timeouts relative to it.

This creates a fundamental ambiguity for reverse-engineering. When you observe a behavior in an existing system, it occupies one of four states:

|                              |Intended by author     |Not intended by author     |
|------------------------------|-----------------------|---------------------------|
|**Depended on by callers**    |Unambiguous requirement|Accidental contract (Hyrum)|
|**Not depended on by callers**|Latent requirement     |Implementation artifact    |

A forward-authored NLSpec populates the left column: intended behaviors, whether or not callers currently depend on them. A reverse-engineered spec can only observe the top row: behaviors that callers depend on, whether or not they were intended. The off-diagonal cells — accidental contracts and latent requirements — are where the two approaches diverge, and neither observation nor code reading resolves them.

**Accidental contracts** (intended: no, depended on: yes) are the Hyrum’s Law case. The reverse-engineered spec must include them because removing them breaks callers. But including them elevates accidents to requirements, which is a form of spec pollution — the spec now mandates behavior that the original author would not have chosen.

**Latent requirements** (intended: yes, depended on: no) are invisible to reverse engineering. If the author intended a security invariant that no current caller exercises, the code may or may not enforce it, and no amount of behavioral observation reveals the intent. If the reverse-engineered spec omits it, a future reimplementation might drop the invariant, violating intent that was never captured.

### What Is Tractable

Given these constraints, a reverse-engineering effort can produce:

1. **A behavioral description** (Level 1) — fully tractable, valuable as a reference document, but not a spec.
1. **A provisional behavioral contract** (Level 2) — where each behavior is annotated with a confidence level indicating whether it appears intentional, accidental-but-depended-on, or indeterminate. This is the honest artifact. It does not claim to be a spec. It claims to be the best reconstruction possible without access to original intent, with uncertainty made visible.
1. **A prescriptive spec with declared assumptions** (Level 3, partial) — where the author of the reverse-engineered spec makes explicit judgment calls about which behaviors to elevate to requirements, documents those judgments, and accepts that some may be wrong. This is a spec, but it carries an epistemic disclaimer that a forward-authored spec does not: *some requirements in this document are inferred, not dictated*.

The third option is the most useful in practice but requires a human (or agent with domain authority) to make the judgment calls. The reverse-engineering process surfaces the questions; a domain authority answers them. Without that authority, the process stalls at Level 2.

### Implications for the Grounding Document

The existence of reverse-engineered specs does not weaken the `Intent → NLSpec → Implementation` model. It extends it. A reverse-engineered spec is an attempt to recover a document that *should have existed* but didn’t — to retroactively insert the NLSpec into a chain that skipped it.

The quality of that recovery is bounded by the information available. Forward-authored specs are strictly superior in authority because they capture intent at the time intent exists. Reverse-engineered specs are approximations — useful, sometimes necessary, but always carrying uncertainty that forward-authored specs do not.

This is an argument for writing specs forward. The cost of authoring a spec before implementation is paid once. The cost of reverse-engineering a spec after the fact is higher, the result is less certain, and some information is unrecoverable. The grounding document’s insistence on the `Intent → NLSpec → Implementation` direction is not merely a preference for process — it is a claim about information preservation. Intent that is captured in a spec is preserved. Intent that is only captured in code is lossy.

### Open Questions

- **Can test suites substitute for intent?** A comprehensive test suite is a partial behavioral contract — it asserts what the system must do. But tests, like code, don’t distinguish “intended” from “depended on.” A test that was written to pin down accidental behavior (a regression test for a bug that callers adapted to) looks identical to a test that captures original intent.
- **Can commit history and documentation recover intent?** In principle, yes — commit messages, design docs, PR discussions, and issue trackers contain intent signals. In practice, these are noisy, incomplete, often contradictory, and expensive to process. They are useful heuristics, not reliable sources of ground truth.
- **Is there a formal framework for confidence annotation?** The Level 2 artifact (provisional behavioral contract with confidence levels) would benefit from a structured vocabulary for expressing certainty about inferred intent. No such vocabulary is established. Terms like “almost certainly intentional,” “likely accidental,” and “indeterminate” are natural language and imprecise — ironic for a document about precision. A formal framework here is an open research problem.
- **What role can LLMs play?** Large language models can assist at every level — reading code, inferring behavioral contracts, even making judgment calls about intent. But their judgment is probabilistic and their confidence is not calibrated. An LLM that says “this behavior is probably intentional” is pattern-matching against training data, not recovering ground truth. LLMs are useful tools for reverse-engineering; they are not oracles of intent. The human (or domain authority) remains the final arbiter.