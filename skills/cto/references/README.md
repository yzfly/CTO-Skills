# References — Three Layers

The CTO skill's reference tree is split into three layers by **purpose**, not topic. Each layer has a different role, a different audience, and a different loading rule.

```
references/
├── runtime/      Layer 1 — CTO's working memory
├── personas/     Layer 2 — CTO's advisory board
└── payload/      Layer 3 — Downstream deliverables
```

## Layer 1 — `runtime/`

**What it is.** The files the CTO loads into its own context during a design conversation. Short, dense, written for the CTO itself.

**Loading rule.** Lazy. Load each file the moment its trigger condition fires, not upfront.

**Contents.**

| File | Trigger to load |
|------|----------------|
| `industry-reflexes.md` | The user's industry/domain becomes clear |
| `tradeoffs.md` | An architectural inflection point appears (consistency vs availability, sync vs async, etc.) |
| `templates.md` | First time writing to `brief.md`, `arch.md`, or a feature spec |
| `stack-practices.md` | After tech stack is settled in `arch.md` §3 — bridges to `payload/` |

## Layer 2 — `personas/`

**What it is.** 10 legendary CTO/engineer decision personas. Used as a private "second opinion" — not role-play, but architectural pressure tests.

**Loading rule.** Selective. Always read `personas/README.md` first (scene→persona index), then load only the persona file(s) the situation calls for.

**Contents.** Werner Vogels, Linus Torvalds, Patrick Collison, Martin Fowler, DHH, Kelsey Hightower, Charity Majors, Jeff Dean, Andrej Karpathy, Will Larson.

## Layer 3 — `payload/`

**What it is.** The coding standards, framework guides, and engineering methods the CTO bundles into the user's project as deliverables for the next coding agent.

**Loading rule.** **Never load into the conversation.** Bundle the relevant subset into the user's project output as `<project>/references/`. The bridge file `runtime/stack-practices.md` is the manifest that tells you which subset to bundle.

**Contents.**

| Category | Source | What's in it |
|----------|--------|--------------|
| `standards/` | Google, Uber, Airbnb, official | Python, Go, JS, TS, Rust, SQL style guides |
| `frontend/` | Vercel, Vue/Nuxt | React/Next.js (45 rules), Vue/Nuxt patterns |
| `backend/` | samber, zhanymkanov | compressed Go backend playbook, FastAPI best practices |
| `design/` | Microsoft, Google, PostgreSQL | API design, schema design, system design |
| `review/` | Google eng-practices | Code review (reviewer + author guides) |
| `security/` | utkusen | 13 SAST vulnerability detection skills |
| `methods/` | Addy Osmani | 17 engineering method skills (TDD, CI/CD, etc.) |

## How the three layers cooperate

```
┌─────────────┐  load when triggered    ┌──────────────┐
│ runtime/    │ ──────────────────────▶ │ CTO context  │
└─────────────┘                         │              │
┌─────────────┐  load selectively       │ (conversation│
│ personas/   │ ──────────────────────▶ │  with user)  │
└─────────────┘                         │              │
                                        └──────┬───────┘
                              stack-practices.md │ decides bundle
                                                 ▼
┌─────────────┐                         ┌──────────────┐
│ payload/    │ ──── bundle subset ───▶ │ user project │
└─────────────┘                         │ /references/ │
                                        └──────────────┘
```

The CTO never sees `payload/` files. The downstream coding agent never sees `runtime/` or `personas/` files. Each layer ships to a different reader.
