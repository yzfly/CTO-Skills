# Artifact Templates

The exact section structures for the three artifacts. Use these as the authoritative schemas. Do not invent new sections; do not skip required sections.

When writing to an artifact:
- Fill what you know now. Mark what you don't yet know with `// to be discussed` or `// pending`.
- Re-read the relevant template before each write to make sure the structure stays clean.
- The artifacts are read by both humans (the user) and AI (a coding agent). Optimize for both: clear language for humans, structured headings for AI.

---

## Template 1: `brief.md` — the business

This is the artifact a non-technical user reads with zero friction. Keep it in business language only — no architecture, no tech stack, no technical jargon.

```markdown
# Product Brief: <product-name>

## 1. Vision
- **What this is, in one sentence:** <plain-language description of what the product is and who it serves>
- **Core principle:** When <axis A> and <axis B> conflict, we lean toward <chosen axis>.

  *(Examples: "When safety and speed conflict, we lean toward safety." "When richness of features and simplicity conflict, we lean toward simplicity.")*

## 2. Goals
- **Business goals:** measurable business outcomes
  - <goal 1>
  - <goal 2>
- **Experience goals:** what users should feel
  - <goal 1>
- **Scale goals:** what volume the system must handle
  - <goal 1>

## 3. Actors
For each kind of person who will use the system:
- **<Actor name>:** <one-sentence description of who they are and what they want from the product>

## 4. Domain Concepts
The vocabulary of this product. One concept = one name, used consistently throughout.

- **<ConceptName>:** <precise definition in plain language>
- **<ConceptName>:** <precise definition in plain language>

## 5. Business Rules
Rules that hold across the whole product, regardless of which feature.

- <rule 1>
- <rule 2>

## 6. Non-Goals
What we explicitly are NOT doing in this version. Stating these protects the design from feature creep.

- <non-goal 1>
- <non-goal 2>
```

### How to fill `brief.md` from the conversation

- **§1 Vision** — write after the first 1-2 turns once the user has said what the product is and what they care about. The "core principle" is critical — it captures the user's value priority and becomes the tiebreaker for all later decisions.
- **§2 Goals** — fill incrementally. As the user mentions metrics or aspirations, slot them into the right bucket. If a category is empty after several turns, surface it ("you've told me a lot about who'll use this — what does success look like in numbers?").
- **§3 Actors** — every product has 1-3 primary actors. Name them. If you've heard about admins, support staff, partners — they go here too.
- **§4 Domain Concepts** — this is the product's vocabulary. Lock terms early, before they drift. If the user says "Order" once and "Purchase" later, gently fix on one.
- **§5 Business Rules** — these are the cross-cutting truths (e.g., "all amounts in smallest currency unit," "every action must be auditable," "users can only see their own data"). Distill these from what the user says, don't invent them.
- **§6 Non-Goals** — listen for "we don't need to..." and "later we might but not now..." — those go here. Surface this section explicitly toward the end of the conversation: "anything you definitely don't want to tackle in v1?"

---

## Template 2: `arch.md` — the technical decisions

This artifact is read by future engineers and AI coding agents. It should be unambiguous, decision-driven, and explicit about trade-offs.

```markdown
# Architecture: <product-name>

## 1. Architectural Drivers
The non-functional priorities that judge every later decision. Each driver has a level: Must, Should, May.

- **Must:** <driver — e.g., "zero financial discrepancy", "no PII leakage">
- **Must:** <driver>
- **Should:** <driver — e.g., "P99 < 500ms for core actions">
- **May:** <driver — e.g., "fast cold-start time">

## 2. Key Trade-offs
Explicit decisions on the architectural axes that mattered. Each follows the ADR format.

### Decision 1: <axis-of-tradeoff>
- **Choice:** <what was decided>
- **Rationale:** <why this fits the drivers>
- **Cost / what we give up:** <what becomes harder>
- **Trigger to revisit:** <concrete signal to reopen this decision>

### Decision 2: <axis-of-tradeoff>
- **Choice:** ...
- **Rationale:** ...
- **Cost / what we give up:** ...
- **Trigger to revisit:** ...

(continue for each material decision)

## 3. Tech Stack
Each component answers two questions: why this, why not the alternatives.

- **Frontend:** <choice>
  - Why: <fit with drivers>
  - Why not <alternative>: <reason>
- **Backend:** <choice>
  - Why: ...
  - Why not <alternative>: ...
- **Database:** <choice>
  - Why: ...
- **Cache / queue / search / etc., as needed:** ...
- **Third-party services:** <list with purpose>

## 4. System Topology & Data Flow
- **Deployable units:** <list — e.g., "Web (Next.js), API (FastAPI), Worker (Python), Postgres, Redis">
- **How they communicate:** <e.g., "Web → API over HTTPS; API → Worker via Redis queue; API → Postgres direct">
- **Read path:** <typical request from user → response>
- **Write path:** <typical mutation, including transaction boundaries and side effects>
- **Async path:** <what gets queued, who consumes it, what happens on failure>

## 5. Architectural Patterns
- **Overall pattern:** <e.g., Clean Architecture, Layered, DDD, Event-driven>
- **Dependency rules:** <which layer can call which; explicit forbidden directions>
- **Module boundaries:** <how the codebase is split into logical modules>

## 6. Cross-Cutting Concerns
Universal rules. Defined once here; every feature inherits them.

- **Authentication:** <strategy and token transport>
- **Authorization:** <model from `tradeoffs.md` §15>
- **Error handling:** <how errors propagate, how they're returned to the client>
- **Logging:** <what is logged, what is forbidden to log (PII, secrets)>
- **Caching:** <what is cached, TTL, invalidation strategy>
- **Rate limiting:** <where, by whom, what limits>
- **Idempotency:** <which endpoints require it, how keys are formed>
- **Audit log:** <what state changes are recorded>

## 7. Directory Mapping
Hard rules for where code lives. This is what keeps coding agents from sprawling files.

- `src/<path>/`: <what kind of code goes here>
- `src/<path>/`: <what kind of code goes here>
- `tests/<path>/`: <what kind of tests go here>

## 8. Boundaries & Anti-patterns
Hard prohibitions. Things that must never happen in this codebase.

- <prohibition — e.g., "UI components must not directly query the database">
- <prohibition — e.g., "No floating-point arithmetic on monetary values">
- <prohibition — e.g., "No swallowing exceptions in catch blocks">

## 9. Open Questions
Things we have NOT yet decided. Be explicit so a coding agent doesn't fabricate answers.

- <question — e.g., "Multi-currency support? Deferred until v2; amount fields keep a `currency` column to avoid migration later.">
- <question>
```

### How to fill `arch.md` from the conversation

- **§1 Drivers** — write these as soon as you understand the user's "core principle" (from `brief.md` §1). Drivers are the projection of business priorities onto engineering reality.
- **§2 Trade-offs** — every time you reach a destiny fork, record it as an ADR. Don't wait until the end. Even decisions you made silently and confidently should be recorded — future agents need to see the reasoning.
- **§3 Tech Stack** — pick after §2 trade-offs are mostly settled. The trade-offs constrain the stack.
- **§4-§6** — fill incrementally as conversation reveals patterns. §6 in particular grows over many turns.
- **§7 Directory Mapping** — late-stage. Once stack and patterns are settled, lay out the file structure.
- **§8 Boundaries** — listen for the user's "we definitely shouldn't..." moments and translate them into hard rules.
- **§9 Open Questions** — this section is critical and frequently neglected. Anything you didn't fully resolve goes here, in plain language, with whatever provisional answer you've adopted. Coding agents are taught to read this section and not invent solutions to questions explicitly marked open.

---

## Template 3: `specs/features/<feature-name>.md` — executable feature design

One file per feature. Each file is a complete construction unit a coding agent can build from without needing to ask questions.

```markdown
# Feature: <feature-name>

## 1. Purpose
What business problem this feature solves, and which goal in `brief.md` it advances.

## 2. User Story
As a <actor from brief.md §3>, I want to <action>, so that <value>.

## 3. Domain Model
- **Concepts involved (from `brief.md` §4):** <list>
- **Data structures:**
  - **<EntityName>:** key fields with type and nullability
    - `field_name` (type, constraint)
    - `field_name` (type, constraint)
- **State machine (if applicable):** <state-A → state-B under what condition>

## 4. Pre-conditions
What must be true for this feature to be invoked.
- <condition>
- <condition>

## 5. Behavior
The core flow, in BDD style. Cover the happy path AND the meaningful failure paths.

### Happy path
- **Given** <initial state>
- **When** <user action>
- **Then** <expected outcome>

### Branch: <name of failure or alternative path>
- **Given** ...
- **When** ...
- **Then** ...

(continue for each branch)

## 6. Interface
The contract this feature exposes to other parts of the system.

- **Endpoint(s):** <method + path, e.g., `POST /api/withdrawals`>
- **Request shape:**
  ```json
  { ... }
  ```
- **Response shape (success):**
  ```json
  { ... }
  ```
- **Response shape (failure):** standard error envelope (defined in `arch.md` §6)
- **Error codes:**
  - `<CODE_NAME>`: <when this fires, what the user sees>
  - `<CODE_NAME>`: ...

## 7. Edge Cases
Things that must not break this feature.
- **Concurrency:** <what happens if two actors do this simultaneously>
- **External failure:** <what if the third-party / DB / queue fails — retry / fail-fast / circuit-break>
- **Extreme inputs:** <oversized strings, negative numbers, unicode, empty values>
- **Idempotency:** <how is duplicate invocation handled>

## 8. Acceptance Criteria
The checklist the implementation must satisfy. The coding agent will generate tests against these.

- [ ] <criterion>
- [ ] <criterion>
- [ ] <criterion>
```

### How to fill `specs/features/*.md` from the conversation

- Start a new feature file the first time the user describes a concrete user-action flow ("when a creator clicks withdraw, then..."). The trigger is *concreteness*: a specific actor doing a specific thing in a specific order.
- Fill §1, §2, §3 first — they're business-side and the user can validate them.
- Fill §5 (behavior) directly from the user's description. If they only describe the happy path, surface the failure paths yourself: "what should happen if [external thing] fails?" — but only ask if the answer materially affects design.
- §6 (interface) and §7 (edge cases) are mostly your CTO judgment. Don't burden the user with field-level questions; apply sensible defaults from `arch.md` §6.
- §8 (acceptance criteria) is the last section; it summarizes "done" in business-verifiable terms.

### Naming features

Use kebab-case, action-oriented when possible: `creator-withdraw.md`, `seller-onboarding.md`, `order-checkout.md`, `team-invite.md`. Avoid generic names like `auth.md` (too broad — split into `user-signup.md`, `user-login.md`, `password-reset.md` if needed).

---

## A delivery checklist

Before presenting artifacts to the user, run through this:

**`brief.md`**
- [ ] Vision is one sentence, plus a core principle
- [ ] At least one goal in each category (or category explicitly marked "none for v1")
- [ ] All actors named
- [ ] Domain concepts listed with definitions
- [ ] At least one business rule and one non-goal recorded

**`arch.md`**
- [ ] Drivers list exists, with Must/Should/May labels
- [ ] At least the major destiny-fork decisions are recorded as ADRs
- [ ] Tech stack picked with "why this / why not alt" reasoning
- [ ] Directory mapping is concrete enough that an agent won't sprawl
- [ ] Open questions section explicitly lists what's unresolved

**`specs/features/*.md`**
- [ ] At least the v1 critical features have files
- [ ] Each file's behavior section covers happy path and at least one failure path
- [ ] Each file's acceptance criteria is verifiable, not vague

When all three pass their checklist, you're ready to deliver.
