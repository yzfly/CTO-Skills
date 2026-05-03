# Industry Reflexes

Load this file when you've identified the user's industry/domain. It contains the "fill in the silent 80%" reflexes — what an experienced CTO already knows to ask about, by category.

The format for each domain:

1. **The hidden questions** — what the user almost certainly hasn't told you but you need to know to design well. Most you can fill in with sensible defaults; some are destiny forks worth surfacing.
2. **The classic mistakes** — what naive designs get wrong here, so you don't repeat them.
3. **What goes in `brief.md`** — the domain concepts and rules that should be locked in early.

You don't need to ask every question in the list. Use it as a checklist to cross-reference what you already know, and surface only the gaps that actually matter.

---

## Table of contents

1. [Marketplace / two-sided platform](#1-marketplace--two-sided-platform)
2. [SaaS (B2B subscription)](#2-saas-b2b-subscription)
3. [Content platform / UGC](#3-content-platform--ugc)
4. [E-commerce](#4-e-commerce)
5. [Fintech / money movement](#5-fintech--money-movement)
6. [Internal tool / admin panel](#6-internal-tool--admin-panel)
7. [AI application / LLM-powered product](#7-ai-application--llm-powered-product)
8. [Social / community](#8-social--community)
9. [Booking / scheduling](#9-booking--scheduling)
10. [Education / learning](#10-education--learning)

If the user's product falls outside these, do your best to map it to the closest neighbor and adapt.

---

## 1. Marketplace / two-sided platform

### Hidden questions

- **Who pays whom, and how does the platform make money?** Take-rate? Subscription on one side? Listing fees? Lead fees? — this is the entire economic engine; surface it early.
- **How does supply onboard?** Self-serve, invite-only, manual review? Liability for fakes/scams sits with whoever lets them in.
- **How is trust established?** Reviews, ratings, verifications, KYC? A marketplace without trust mechanisms dies in the first cohort of bad actors.
- **What happens when something goes wrong?** Disputes, refunds, chargebacks, bad reviews — who arbitrates?
- **Is there a holding period on payouts?** Most marketplaces hold funds until delivery confirmed. This shapes the entire money-flow design.
- **Search and discovery** — sorted by what? Recency, distance, ratings, paid placement? The ranking algorithm is product strategy.

### Classic mistakes

- Treating the two sides as symmetric. They're not. Supply is usually orders of magnitude harder to acquire and needs different tooling.
- Forgetting about disputes until a real one happens.
- Letting the take-rate be a "settings" field instead of a deliberate per-vertical/per-tier decision.

### What goes in `brief.md`

- Domain concepts: `Buyer`, `Seller` (or domain-specific terms — `Host`/`Guest`, `Driver`/`Rider`), `Listing`, `Order`, `Dispute`, `Payout`.
- Business rules: take-rate, minimum payout threshold, holding period, dispute window.
- Non-goals worth declaring: "no in-platform messaging in v1," "no buyer-to-buyer" — whatever the user has implicitly excluded.

---

## 2. SaaS (B2B subscription)

### Hidden questions

- **Tenancy model.** One company per workspace? Multiple workspaces per user (think Slack)? This is foundational.
- **Roles inside a tenant.** Owner, admin, member, viewer? Custom roles? — surface this destiny fork.
- **Billing model.** Per-seat, per-usage, flat tiers, hybrid? Trial period? Free tier? Annual vs monthly?
- **Onboarding flow.** Self-serve sign-up, sales-led, or both? This shapes everything from auth to UX.
- **Data export and account closure.** GDPR-relevant. What happens to data when they cancel?
- **Single sign-on.** Will enterprise customers demand SAML/OIDC? Usually yes, eventually — design auth to allow it.
- **Audit log and compliance.** Most B2B buyers above a certain size will ask for SOC 2. Plan for an audit log from day 1; retrofit is painful.

### Classic mistakes

- Building a single-tenant prototype and discovering tenant isolation is hard to retrofit.
- Hardcoding "the user's role" instead of "the user's role in this tenant" — same human can be admin in one workspace and member in another.
- Forgetting that the person who signs up is rarely the person who uses the product daily.

### What goes in `brief.md`

- Domain concepts: `Tenant` (or `Workspace`/`Organization`), `Membership`, `Role`, `Subscription`, `Plan`, `Seat`.
- Business rules: tenant isolation invariant ("no query may return data across tenants"), role-permission matrix.
- Non-goals: "no cross-tenant features in v1," etc.

---

## 3. Content platform / UGC

### Hidden questions

- **Moderation strategy.** Pre-moderation, post-moderation, reactive (only on report)? This determines whether you need a moderation queue, ML classifiers, human reviewers.
- **What constitutes abuse?** Definitions matter. Hate speech? Spam? Adult content? Misinformation? The product's stance on each shapes operations.
- **DMCA and copyright.** Will users upload media? You almost certainly need a takedown process.
- **Discovery / feed algorithm.** Chronological, personalized, curated? This is half the product.
- **Account-level abuse.** Shadowbans, suspensions, appeals.
- **Content lifecycle.** Edit history? Soft-delete with grace period? Retention policy?

### Classic mistakes

- Underestimating moderation operations. "We'll figure it out later" → first viral moment becomes a crisis.
- Not separating "reporting" from "moderation" in the data model — reports are claims, moderation is action; conflating them limits the audit trail.
- Letting the feed algorithm be an afterthought when it's actually the entire product.

### What goes in `brief.md`

- Domain concepts: `Author`, `Content` (post/article/video — domain-specific), `Reaction`, `Report`, `ModerationAction`.
- Business rules: content states (`draft` / `published` / `flagged` / `removed` / `restored`), retention policy.
- Non-goals: explicit policy stance ("no NSFW in v1," "no political content," etc.).

---

## 4. E-commerce

### Hidden questions

- **Inventory model.** Physical SKUs, digital, dropship, made-to-order? This decides whether stock counts are real.
- **Catalog structure.** Single product? Variants (size/color)? Bundles? Subscriptions?
- **Pricing complexity.** Fixed, dynamic, tiered, tax-inclusive vs exclusive, multi-currency, promotions, coupons.
- **Cart vs checkout vs order vs fulfillment.** These are four distinct concepts. Conflating them causes bugs.
- **Tax.** US sales tax (jurisdiction-dependent), EU VAT, etc. — usually buy this (TaxJar, Stripe Tax).
- **Shipping.** Rate calculation, carrier integration, multi-warehouse?
- **Returns and refunds.** Window, condition, restocking fee, partial refund.

### Classic mistakes

- Storing prices as floats. Always integers in smallest currency unit.
- Not snapshotting price/tax/shipping at order creation time — promotions change, tax tables update, but historical orders must remain accurate forever.
- Letting the cart and order share a state machine when they have very different lifecycles.

### What goes in `brief.md`

- Domain concepts: `Product`, `Variant`, `Cart`, `Order`, `LineItem`, `Payment`, `Shipment`, `Return`.
- Business rules: currency precision, order state machine, tax authority, promotion stacking rules.
- Non-goals: "no international shipping in v1," etc.

---

## 5. Fintech / money movement

### Hidden questions

- **Regulatory posture.** Are you a money transmitter? Holding funds? KYC required? This is the difference between "easy startup" and "two-year licensing process." Often the answer is "use a partner like Stripe/Wise/Modern Treasury who is already licensed."
- **The ledger.** You need a real double-entry ledger from day 1, not an `account.balance` column. Every credit has a corresponding debit. Auditable.
- **Idempotency.** Every money-moving endpoint must be idempotent. Network retries, webhook retries, button double-clicks must never double-charge.
- **Reconciliation.** Internal ledger vs partner balance vs bank account — they must reconcile, and you must detect when they don't.
- **Fraud and risk.** Velocity limits, manual review, holds, reversals.
- **Audit and compliance.** Every state change logged with actor, timestamp, before/after.

### Classic mistakes

- Storing balance as a single field that gets updated. Use a ledger; balance is derived.
- Floats for money. Integers in smallest currency unit, always.
- Treating webhooks as fire-and-forget. Webhooks must be verified, idempotent, and logged.
- "We'll add KYC later." KYC is foundational; bolting it on means going back through every existing user.

### What goes in `brief.md`

- Domain concepts: `Account`, `LedgerEntry`, `Transaction`, `Hold`, `Payout`, `Reconciliation`.
- Business rules: every money movement is two ledger entries, every entry has an immutable timestamp, balance is derived.
- Non-goals: explicit regulatory boundaries ("we are not a custodian," "we use Stripe Connect for all payouts").

---

## 6. Internal tool / admin panel

### Hidden questions

- **Who exactly will use this?** Internal team only? How many people? How sophisticated?
- **What does it sit on top of?** Existing database? External APIs? Spreadsheets being replaced?
- **Auth.** Often SSO with the company's existing identity (Google Workspace, Okta).
- **Auditability.** If someone uses this tool to do something destructive, you need to know who and when.
- **How often does it change?** If schema/operations change weekly, lean toward generic tools (Retool, internal admin frameworks). If stable, custom is fine.

### Classic mistakes

- Over-engineering. Internal tools serve 5-50 people. Don't build them like consumer apps.
- Under-securing. "It's internal" is not security. Internal tools can do destructive operations and need permission gates.
- Ignoring the audit log because "we trust our team." When something breaks, the audit log is the only way to know what happened.

### What goes in `brief.md`

- Domain concepts: borrow from the underlying system being administered.
- Business rules: who can do what (role list), what's audited, who can see audit log.
- Non-goals: "this is not a customer-facing product, no need for marketing/SEO/onboarding flow."

---

## 7. AI application / LLM-powered product

### Hidden questions

- **What's the prompt strategy?** Single shot, multi-turn, agent loop, RAG? This determines architecture.
- **Latency expectation.** Streaming response? Background processing? Both?
- **Cost per interaction.** LLM calls cost real money. Need to model cost per user-action and decide caching/limits.
- **Eval and quality.** How will you know if the AI is getting better or worse? Need an eval framework from day 1, not later.
- **Fallback when AI fails.** Hallucination, refusal, API outage — what does the user see?
- **PII / sensitive data.** Will user data be sent to the model provider? OK with that? Need on-prem model?
- **Safety / abuse.** Jailbreak attempts, prompt injection, PII extraction. What's the policy?

### Classic mistakes

- Treating prompts like static config. Prompts are critical product logic; they need versioning, tests, and rollback.
- No eval. You can't improve what you can't measure. Quality regression goes undetected.
- Synchronous LLM calls in the request path with no timeout. Single point of failure for the whole product.
- Sending raw user input straight to the model with no input validation. Easy abuse vector and easy way to leak system prompts.

### What goes in `brief.md`

- Domain concepts: `Prompt`, `Conversation`, `Message`, `EvalCase`, `ModelRun`.
- Business rules: model provider, fallback model, cost ceiling per user, eval pass threshold for prompt changes.
- Non-goals: explicit safety boundaries ("we will refuse X categories of prompts").

---

## 8. Social / community

### Hidden questions

- **Identity model.** Real names, pseudonyms, anonymous, multiple personas?
- **Connection model.** Follow (asymmetric), friend (symmetric), group, none?
- **Notification strategy.** Email, push, in-app, all of the above? Frequency limits?
- **Anti-spam, anti-abuse.** Rate limits on actions, account age requirements, mass-action detection.
- **Privacy controls.** Profile visibility, post visibility, block/mute, message permissions.
- **The first-100-users problem.** Social products are worthless when empty. What's the strategy for cold start?

### Classic mistakes

- Letting users send unlimited DMs to strangers. Spam follows immediately.
- No block/mute from day 1. People will need it on day 1.
- No notification preferences. Email-blasting users is a fast path to unsubscribes.

### What goes in `brief.md`

- Domain concepts: `Profile`, `Connection` (or `Follow`/`Friend`), `Post`, `Reaction`, `Notification`, `Block`.
- Business rules: connection symmetry, privacy defaults, notification batching rules.

---

## 9. Booking / scheduling

### Hidden questions

- **Resource model.** What's being booked — a person's time, a physical resource, a service window?
- **Availability rules.** Working hours, recurring blocks, exceptions, time zones (always a nightmare).
- **Conflict resolution.** What if two people try to book the same slot at the same time? (See `tradeoffs.md` §1 and §7.)
- **Buffer time, prep time, cleanup time.** Often forgotten and causes ops issues.
- **Cancellation and rescheduling policy.** Window? Fees? Refunds?
- **Confirmation and reminders.** Email, SMS, calendar invites.
- **Calendar integration.** Google Calendar, Outlook, etc. — usually expected.

### Classic mistakes

- Time zone bugs. Always store UTC. Always render in user's local. Never store local time without zone.
- Allowing double-booking because of race conditions on availability checks.
- Forgetting that "available" is the absence of a booking, not a stored fact — and storing it as a stored fact gets out of sync.

### What goes in `brief.md`

- Domain concepts: `Resource` (or `Provider`), `AvailabilityRule`, `Booking`, `Cancellation`.
- Business rules: time zone policy, booking window (how far in advance), cancellation window.

---

## 10. Education / learning

### Hidden questions

- **Course structure.** Linear, branching, freeform? Modules, lessons, units?
- **Progress tracking.** Per-lesson, per-quiz, mastery-based? Resume-where-you-left-off?
- **Assessment.** MCQ, free-form, peer review, instructor grading?
- **Cohort vs self-paced.** Live classes? Discussion forums? Office hours?
- **Certification.** Completion certificates? Verifiable? Industry-recognized?
- **Content authoring.** Who creates content — staff, instructors, users? Authoring tools?
- **Plagiarism / academic integrity.** If credentials are issued, this matters.

### Classic mistakes

- Building "a course" as a single content blob. Should be: modules → lessons → blocks. Future-self thanks you.
- Not separating "content" from "enrollment" from "progress." Three distinct concerns.
- Designing for one mode (self-paced) when the product needs to flex (cohorts, live, etc.).

### What goes in `brief.md`

- Domain concepts: `Course`, `Module`, `Lesson`, `Enrollment`, `Progress`, `Assessment`.
- Business rules: progress definition, completion criteria, retake policy.

---

## A note on cross-domain products

Many real products span multiple domains. A coaching marketplace = marketplace + booking + content. A learning platform with payments = education + e-commerce. When this happens:

1. Identify the *primary* domain — usually the one the user emphasized first, or the one carrying the business model.
2. Layer the secondary domains on top, importing only the relevant concepts and rules.
3. Be wary of conflicts: e-commerce wants integer-cents prices, fintech wants a ledger, marketplace wants payouts. They can coexist but the domain model needs to reconcile them.

When you spot a cross-domain product, mention this to yourself and load the multiple sections you need. The `brief.md` domain concept list will naturally be longer.
