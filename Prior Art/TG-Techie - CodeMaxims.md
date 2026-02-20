
# Code Maxims

Core principles for a spec-first codebase:

### 0. Spec-First: Design before implementation

**Why this matters (the engineering philosophy):**

A fundamental truth of software engineering: clear communication of intent is often more critical to success than technical acumen alone. Both matter deeply, but without shared understanding, even brilliant code solves the wrong problem.

- A specification is not bureaucracy—it is an act of empathy for every future reader, maintainer, and AI agent who will inherit this code
- The spec-first discipline reflects wisdom, not rigidity: understanding before implementation prevents costly rework
- Well-written types and well-written prose work in concert; neither alone tells the complete story
- Code that cannot be understood cannot be safely changed; specs make understanding explicit
- Technical debt often begins not in code, but in ambiguity—when what a module "should do" exists only in scattered minds rather than explicit prose

**The rules:**

* Write/update the specification BEFORE writing code OR before modifying existing code
* Treat these specs exactly like how Elixir maintains docs in its own code
* The spec is the source of truth for behavior and requirements
* Implementation follows the spec, not the other way around
* When spec and code disagree, fix the code (or update spec with explicit approval)
* A spec must always be self-consistent at all times, if self inconsistencies are identified ask the user for guidance! 

**Key purpose:** The spec is a **living record** that distills user specifications and expectations into a specification as work progresses—so future edits understand what the implementation was built to achieve. When the user provides feedback, course corrections, or changes to what is being implemented, the spec must be updated accordingly to reflect the changes. The litmus test for a complete spec is the **recreatability test**: if all implementation were deleted but the spec comment and type definitions remained, could the file be faithfully recreated?

**What the spec SHOULD cover:**
* User requirements and expectations (captured as the conversation progresses)
* Presentation AND behavior (both what it looks like and how it acts)
* State that matters for the behavior
* Implementation details ONLY when user-specified or already present in the spec (as in leave impl details as-is but don't add new ones without express direction)

**What the spec should NOT cover (NON-EXHAUSTIVE):**
* The spec should NOT contain code
* The spec should NOT contain data that belongs in types, docs, or dedicated files
* The spec should NOT contain implementation details by default (unless user-specified or already present)
* The spec should NOT contain content that will be encoded as types with attribute docs, especially if part of a module's public interface.

**Spec vs Developer Documentation:**
* **Behavioral spec (CORRECT)**: Specifies WHAT the module must do and WHY — rules, constraints, state transitions, invariants
* **Developer documentation (SMELL)**: Describes HOW to use the module — usage examples, API patterns, getting started guides
* Key distinction: A spec answers "what must be true?" not "how do I use this?"
* If your spec reads like a README or tutorial, it's developer docs, not a spec

**Assorted Spec writing style Guidance:**
* While we avoid code in specs (since they are natural language), we do not shy away from technical specificity. It is import to be precise and clear in the spec. To goal being to have an unambiguous spec that, if any given agent or human were to read it, would be implemented to create artifacts with the same behavior, structure, function, capabilities, level of detail, and correctness regardless of the implementor or number of times implemented. 
* Write rules as direct prose, not as NOTE/CAUTION/WARNING callouts — excessive callouts are a spec smell
* Callouts are appropriate for: clarifying context, forward-looking implementation notes, or genuinely exceptional warnings
* If you're writing a rule or constraint, it belongs in the body of the spec, not in a callout
* Well-formed, well-written types *ARE* documentation: they encode intent and shape the state space of the program. The prose encodes the natural language, non-code documentation and should not be written using code in the spec prose itself.
* Avoid "Usage Pattern" or "Example" sections in specs — these belong in separate developer docs or JSDoc comments

**Spec Tag Convention (for AI agent parsing):**

Wrap specifications in `<specification>` tags to clearly delineate spec content from other comments:

```typescript
// For references (when a centralized spec lives in another file so it can be shared between modules):
// <specification ref="../file/path/to/spec.md|ts|..." />

// For inline behavioral specs narrowing or partially implementing a centralized spec:
/**
 * <specification narrows="../file/path/to/spec.md|ts|...">
 *
 * ## Purpose
 * This module implements the XYZ part(s) of the spec and provides...
 *
 * ## Behavior
 * - When X happens, Y must occur
 * - State Z is always valid when...
 * 
 * ...
 *
 * </specification>
 */

// For self-contained inline behavioral specs:
/**
 * <specification>
 *
 * ## Purpose
 * This module provides...
 *
 * ## Behavior
 * - When X happens, Y must occur
 * - State Z is always valid when...
 * 
 * ...
 *
 * </specification>
 */


```

Tag semantics:
- Content within `<specification>...</specification>` is the authoritative behavioral contract
- AI agents MUST treat spec content as requirements, not suggestions
- Changes to behavior require updating the spec first (spec-first principle)
- The `ref` attribute points to the file containing the actual specification
- If an inline specification needs 

### 1. Readable: Code tells a story

* Structure mirrors the domain/spec (reader can follow the logic)
* Comments explain "why", not "what"
* AVERSION to monolithic functions/components
  * break up code to express separation of concerns
  * prefer in-file inline components/functions over separate files
* If it's hard to name concisely, it's doing too much
* **Helper ordering rule**: Write helper functions/components AFTER (at a higher line number than) the function/component that uses them
  * Example: If `ParentComponent` uses `HelperComponent`, define `HelperComponent` AFTER `ParentComponent` in the file
  * This creates a natural top-down reading flow where you encounter the "what" before the "how"
  * Exports typically come first, then their helpers, then those helpers' helpers, etc.

### 2. Simple: Clarity over cleverness

* SIMPLER != LESS CODE - more clear lines beats fewer cryptic ones
* Explicit transformations over implicit magic

### 3. Maintainable: Easy to change

* Extract helpers when they clarify intent, think named chapters in the code's story
* Single responsibility per function/component
* Types that make invalid states unrepresentable
* Keep the  *models* DRY. For example, any behavior defining a mutation or derivation of data should be defined once; any repeats, augmentations, or uses of it in new places should reference the original, wrap it, or wire it through to be accessible when it needs to be. (this is DRY).

### 4. Valid State Only: No lying types

* Non-nullable where data must exist
* Sensible fallbacks for optional data (e.g., "Multiple or Unspecified Providers")
* Make impossible states impossible to construct
* ONLY MODEL VALID STATES
* **Read types and write types serve different masters.** A discriminated union that makes invalid *display* states unrepresentable may make a terrible *write* shape — its null constraints become destructive writes that silently wipe sibling data. Smells: branch-heavy write handlers, spreading a union into a write (`{ ...union, field }`), or requiring callers to pass derived values. When fields are stored independently, the write API must (usually) let callers update them independently. allowing either independent writes that are validated against possible correct write states and derive teh appropriate discriminant XOR allowing full model coherent writes. AKA partial xor coherent but never both simultaneously. 

### 5. DRY (Don't Repeat Yourself): Single source of truth

* Doc comments should NOT repeat spec details; just reference the spec section
* Do NOT reference line numbers (they change); use section names instead
* **Implementing specs cite, don't copy:** When a module implements rules defined in an external spec,
  its spec comment cites that spec for the behavioral rules and records only what is locally unique
  to this module (e.g., which internal fields are set, layer-specific invariants). Copying behavioral
  prose into every implementing module creates drift and ambiguity about which version is authoritative.


**Startup posture**: optimize for understanding and modification speed.

### 6. Architectural Hygiene - avoid architecture smells

When designing a solution, ask:

1. **Can I compose existing abstractions?** Before creating `useFooBar()`, check: can the goal be achieved by calling `useFoo()` + `useBar()` and combining at the call site? New hooks/models add cognitive overhead and maintenance burden.

2. **Am I subsetting unnecessarily?** If you need 3 fields from a 10-field row, consider: fetch the full row and destructure at the call site. Creating a new type for just those 3 fields pollutes the architecture unless that subset is genuinely reused across many call sites.

3. **Name things for what they represent** Names like `ExportData` or `ProcessingPayload` exclusively describe purpose but not contents. Prefer `UserProfile`, `InvoiceLineItem`, etc. When the name accurately describes the data shape, code becomes self-documenting.

**Architectural check before implementation:**
- Will this introduce a new model/type/abstraction?
- Is that new abstraction genuinely necessary, or am I creating it for convenience?
- What should this be named to accurately describe its contents (not its purpose)?
- Can I achieve the goal by composing existing abstractions?

When in doubt, discuss the architecture before implementing.

#### Web / React

4. **Match state frequency to update mechanism.** React state (`useState`, context) triggers re-renders — appropriate for infrequent UI transitions (mode switches, data loads). High-frequency visual updates (scroll position tracking, active indicators, drag previews, animations) should use refs + direct DOM manipulation, not React state. Putting high-frequency values in React state is an impedance mismatch that causes re-render cascading through the component tree. `React.memo` can firewall cascades but is a mitigation, not a cure — prefer not creating the cascade in the first place.

5. **Protect `contenteditable` ancestors from re-renders.** `contenteditable`-based editors (e.g., rich text editors) lose cursor position, selection state, and keyboard navigation when their ancestor components re-render. Components that own `contenteditable` children must have stable ancestors — never place rapidly-changing state (`useState`, context values) in a component that is an ancestor of an active editor. When sibling communication is needed (e.g., a scroll-spy updating a sibling panel), route data through refs or narrow callbacks that don't re-render the editor's ancestor chain.