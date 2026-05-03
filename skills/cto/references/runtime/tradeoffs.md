# Architectural Trade-offs

This is your decision catalog. Load this when you sense any architectural inflection point in the conversation.

The structure of each entry is the same:

1. **The trade-off** — the underlying tension
2. **How to spot it** — what user signals indicate this trade-off is in play
3. **The default lean** — what you should bias toward absent strong signal
4. **The business-language question** — how to surface it to the user, never in technical terms
5. **The cost of each side** — what to record in the ADR

You do not need to walk through every trade-off explicitly. Most of them you resolve silently with sensible defaults. You only stop and ask when the user's signals are mixed or when the choice will materially shape the build.

---

## Table of contents

1. [Consistency vs availability](#1-consistency-vs-availability)
2. [Synchronous vs asynchronous](#2-synchronous-vs-asynchronous)
3. [Strong schema vs flexible](#3-strong-schema-vs-flexible)
4. [Monolith vs split](#4-monolith-vs-split)
5. [Build vs buy](#5-build-vs-buy)
6. [Read-write coupled vs separated](#6-read-write-coupled-vs-separated)
7. [Optimistic vs pessimistic locking](#7-optimistic-vs-pessimistic-locking)
8. [Early vs late abstraction](#8-early-vs-late-abstraction)
9. [Single-tenant vs multi-tenant](#9-single-tenant-vs-multi-tenant)
10. [Real-time vs eventual](#10-real-time-vs-eventual)
11. [Server-rendered vs client-rendered](#11-server-rendered-vs-client-rendered)
12. [Stateless vs stateful](#12-stateless-vs-stateful)
13. [Push vs pull](#13-push-vs-pull)
14. [Soft delete vs hard delete](#14-soft-delete-vs-hard-delete)
15. [Authorization model](#15-authorization-model)

---

## 1. Consistency vs availability

**The trade-off.** When two parts of the system disagree (database replicas, services), you can either refuse to serve until they agree (consistency) or serve possibly-stale data to keep responding (availability). You can't have both perfectly under failure.

**How to spot it.** User mentions money, inventory, bookings, voting, anything where two people might race for the same resource. Or user says things like "must always be accurate" or "can't be down."

**The default lean.** If money or scarce resources are involved → consistency. If user-generated content and reads vastly outnumber writes → availability.

**The business-language question.** "If the system has a hiccup and two people try to grab the last item at the same time, would you rather one of them get a clear 'sorry, taken' message, or risk both thinking they got it and us cleaning up later?"

**ADR notes.**
- *Cost of consistency:* slower under load, more expensive to scale, possible brief unavailability during failover.
- *Cost of availability:* possible brief inconsistencies the user might see, requires reconciliation logic, harder to reason about edge cases.

---

## 2. Synchronous vs asynchronous

**The trade-off.** When work happens, does the user wait for it (sync) or does the system promise to do it and free the user immediately (async)?

**How to spot it.** Anything involving external APIs (payment, email, SMS), heavy computation (PDF generation, image processing, AI calls), or long-running workflows.

**The default lean.** Core path that the user is staring at the screen for → sync. Side effects, notifications, third-party callbacks, anything > 2 seconds → async.

**The business-language question.** "When [the slow thing happens], should the user wait and see the result on screen, or is it okay if we say 'we'll let you know in a moment' and they can do other things?"

**ADR notes.**
- *Cost of sync:* user waits, page can hang, retries are user-visible failures, single point of failure on the slow operation.
- *Cost of async:* need a queue + workers, harder to debug, user may not see immediate confirmation, requires status communication (email, push, polling).

---

## 3. Strong schema vs flexible

**The trade-off.** Define every field upfront with strict types (strong schema) or use flexible structures (JSON columns, document DB) that can evolve without migrations.

**How to spot it.** User is unsure about the data model, mentions "different types of X each with different fields," or the product is in early/exploratory phase.

**The default lean.** Core transactional entities (users, accounts, orders, money) → strong schema, every time. Configuration, content payloads, integrations, extensibility points → flexible.

**The business-language question.** "Do you expect [this data] to look basically the same for everyone, or will every customer/use-case need different fields?"

**ADR notes.**
- *Cost of strong schema:* every change is a migration, slower to evolve early on, harder to onboard new shapes of data.
- *Cost of flexible:* harder to query, no enforced invariants, easy to accumulate junk, harder to refactor when it stabilizes.

---

## 4. Monolith vs split

**The trade-off.** One deployable unit (monolith) vs multiple independent services.

**How to spot it.** User asks "should we use microservices" or describes a system where different parts have very different scaling/reliability needs, or different teams will own different parts.

**The default lean.** Always start as a monolith. Split only when there is a concrete forcing function: a part that must scale independently, must deploy independently, must be owned by a separate team, or has fundamentally different reliability requirements.

**The business-language question.** "Are you imagining one team owning the whole product, or different people owning different parts? And do any parts need to handle wildly more traffic than others?"

**ADR notes.**
- *Cost of monolith:* every deploy is a full deploy, scaling means scaling everything, tighter coupling between modules.
- *Cost of split:* dramatically more operational complexity (deployments, monitoring, networking, consistency, testing), worth it only when it solves a real problem.

---

## 5. Build vs buy

**The trade-off.** Build a capability in-house or use an off-the-shelf service.

**How to spot it.** Anything cross-cutting that already has mature SaaS providers: auth, payments, email, search, file storage, video, analytics, feature flags, monitoring.

**The default lean.** Buy by default. Build only when (a) it is the company's core differentiator, (b) it is cheap to build and the SaaS is genuinely worse, or (c) regulatory constraints require it.

**The business-language question.** "[X] is something we can either build ourselves or hand off to a service like [provider]. The service costs money but saves us months. My instinct is [recommendation] — does that sit right with you?"

**ADR notes.**
- *Cost of build:* engineering time, ongoing maintenance, security/compliance burden, opportunity cost.
- *Cost of buy:* recurring fees, vendor lock-in, integration complexity, less control over the experience.

---

## 6. Read-write coupled vs separated

**The trade-off.** Reads and writes hit the same database (coupled) vs reads go to a cache or replica (separated).

**How to spot it.** User describes much higher read traffic than write, or mentions performance concerns on lookup-heavy operations.

**The default lean.** Coupled until you have a measured reason to separate. Premature read-write separation is a major source of bugs.

**The business-language question.** "Is it okay if someone occasionally sees data that's a few seconds out of date — like an article view count — or does everything need to be exactly current?"

**ADR notes.**
- *Cost of coupled:* read load can hurt write performance under scale.
- *Cost of separated:* cache invalidation bugs, eventual consistency confusion, more infrastructure.

---

## 7. Optimistic vs pessimistic locking

**The trade-off.** When two users edit the same thing, do you lock it for one of them (pessimistic) or let both try and resolve the conflict at write time (optimistic)?

**How to spot it.** Inventory, counters, concurrent edits, financial balances.

**The default lean.** Pessimistic for money and scarce resources. Optimistic for collaborative editing and low-conflict UIs.

**The business-language question.** "If two of [actors] try to do [action] on the same thing at the same time, should the second one get a 'sorry, someone is editing' message, or should we let them both try and one of them gets a 'this changed, please refresh' error?"

**ADR notes.**
- *Cost of pessimistic:* worse UX in collaborative scenarios, can deadlock, lock management complexity.
- *Cost of optimistic:* user may lose work and have to redo it, requires version fields, harder for non-technical users to understand the conflict.

---

## 8. Early vs late abstraction

**The trade-off.** Build an abstraction now (DRY, generic) or wait until the third instance demands it.

**How to spot it.** User describes "different kinds of X" — different payment methods, different login providers, different export formats. The temptation is to build a generic plugin system.

**The default lean.** Late by default, especially in the AI-coding era — duplication is cheap to remove later, but a wrong abstraction is expensive to escape.

**The business-language question.** Usually you don't need to ask the user. This is a CTO judgment call. Default to "let's just build the two cases concretely first; if we need a third, we'll generalize then."

**ADR notes.**
- *Cost of early:* you might lock in the wrong shape, abstractions are sticky.
- *Cost of late:* some duplication in the codebase, slightly more work when you do extract.

---

## 9. Single-tenant vs multi-tenant

**The trade-off.** Each customer/team gets their own isolated environment (single-tenant) vs all customers share the same instance, separated logically (multi-tenant).

**How to spot it.** B2B SaaS, anything where companies/teams sign up.

**The default lean.** Multi-tenant logical isolation (tenant_id column on every table, enforced at the query layer) is the default for SaaS. Single-tenant only when (a) regulatory, (b) very large customers, (c) per-customer customization is the product.

**The business-language question.** "Six months from now, do you imagine many separate companies/teams using this, or one big shared community of users?"

**ADR notes.**
- *Cost of multi-tenant:* every query must filter by tenant; bugs can leak data between customers; upgrades affect everyone simultaneously.
- *Cost of single-tenant:* one ops team per customer, hard to update everyone, expensive to scale customer count.

---

## 10. Real-time vs eventual

**The trade-off.** Updates appear immediately to other users (real-time, via websocket/SSE/polling) vs they appear when the user refreshes or navigates.

**How to spot it.** Chat, collaborative editing, live dashboards, multi-user games, notifications.

**The default lean.** Eventual until proven necessary. Real-time has heavy infrastructure cost.

**The business-language question.** "When [user A] does [thing], does [user B] need to see the change instantly without doing anything, or is it okay if they see it next time they open the page?"

**ADR notes.**
- *Cost of real-time:* websocket infrastructure, connection management, scaling complexity, harder to test.
- *Cost of eventual:* less "alive" feeling product, possible confusion if multiple people are looking at the same thing.

---

## 11. Server-rendered vs client-rendered

**The trade-off.** HTML rendered on the server (fast first paint, SEO, simpler) vs in the browser (richer interactions, app-like).

**How to spot it.** Public-facing content, marketing pages, search-indexed pages, vs internal tools, dashboards, editors.

**The default lean.** Public/content-heavy → server-rendered (Next.js App Router, server components). Internal/highly-interactive → client-rendered SPA.

**The business-language question.** "Is this product something people will discover via Google search, or will they always come in via a login?"

**ADR notes.**
- *Cost of server-rendered:* heavier interactive features need careful client-component splitting.
- *Cost of client-rendered:* worse SEO, slower initial paint, requires more careful loading-state design.

---

## 12. Stateless vs stateful

**The trade-off.** Servers hold no per-user state in memory (stateless) — every request carries what it needs — vs servers remember things between requests (stateful).

**How to spot it.** Almost universal default. Only an issue when user mentions long-lived sessions, real-time, sticky connections.

**The default lean.** Stateless API servers, always. State lives in databases, Redis, or queues. Stateful is reserved for specific cases (websocket gateways, in-memory caches with explicit invalidation).

**The business-language question.** Usually doesn't need to surface — this is an internal CTO decision.

**ADR notes.** Just bake "API servers are stateless" into `arch.md` as an invariant.

---

## 13. Push vs pull

**The trade-off.** External system tells us when something happens (push, webhook) vs we ask them periodically (pull, polling).

**How to spot it.** Any third-party integration where state can change on their side: payments, OAuth, file storage, calendar sync.

**The default lean.** Push (webhook) when the provider supports it — fewer wasted calls, faster reaction. Pull when no webhook available or for periodic batch sync.

**The business-language question.** Usually internal CTO call. Surface only if it affects the user experience ("should we show the user the moment payment confirms, or is checking once a minute okay?")

**ADR notes.**
- *Cost of push:* webhook signature verification, idempotency (webhooks retry), public endpoint exposure.
- *Cost of pull:* wasted calls, latency, rate limit pressure.

---

## 14. Soft delete vs hard delete

**The trade-off.** Mark deleted records with a flag and keep them (soft) vs actually remove them from the database (hard).

**How to spot it.** Any product where user can "delete" things — content, accounts, orders.

**The default lean.** Soft delete by default. Disk is cheap; recovering from an accidental hard delete is painful and lost data is permanently lost. Add a separate purge process for hard deletion only when GDPR or storage forces it.

**The business-language question.** "If a user deletes [thing], should they be able to undo it, or is it gone forever?"

**ADR notes.**
- *Cost of soft delete:* every query needs a filter, schema gets cluttered, more storage.
- *Cost of hard delete:* unrecoverable, no audit trail, can violate referential integrity in surprising ways.

---

## 15. Authorization model

**The trade-off.** Simple ownership (each row has a user_id, only owner can access) vs roles (admin/user/guest) vs full RBAC (roles + permissions matrix) vs ABAC (attribute-based, dynamic).

**How to spot it.** Any multi-user product. The user often doesn't know they're picking a model — listen for who can do what.

**The default lean.** Start with ownership + 2-3 fixed roles. Move to RBAC only when permission matrix is genuinely complex (5+ roles or fine-grained permissions). ABAC almost never — it's a compliance/enterprise tool.

**The business-language question.** "Walk me through who's going to use this and what each kind of person can do. Is it just users-on-their-own-stuff, or are there managers/admins who can act on others' behalf?"

**ADR notes.**
- *Cost of ownership-only:* breaks the moment you need teams or admins.
- *Cost of full RBAC:* significant up-front modeling, complex UI for permission management.

---

## ADR template

When you record an architectural decision in `arch.md`, use this format:

```
### Decision N: <axis-of-tradeoff>

- **Choice:** <what was decided>
- **Rationale:** <one or two sentences, in plain language, why this fits the user's stated drivers>
- **Cost / what we're giving up:** <what becomes harder or impossible because of this choice>
- **Trigger to revisit:** <a concrete signal that would force reconsidering — e.g., "if QPS exceeds 5000", "when we onboard a customer in a regulated region">
```

The "trigger to revisit" field is unusually important. It tells future engineers (and future AI agents) when to come back and reopen the decision. Most architectural debt accumulates because old decisions are never reconsidered. Make the revisit condition explicit.

---

## A note on the "default lean"

The default leans above are heuristics for typical SaaS / web products. Override the default when the user's drivers (recorded in `arch.md` §1) clearly point the other way.

Examples of override-worthy signals:
- User explicitly says "this is for healthcare data" → consistency, audit log, soft delete, encryption all become more rigid
- User explicitly says "this is a hackathon prototype" → buy everything, monolith always, defer everything deferrable
- User says "I want to scale to a million users on day one" → push back on this politely; almost no one needs that
