# Payload — Downstream Deliverables

This directory holds the coding standards, framework guides, and engineering methods the CTO bundles into the user's project as deliverables for the next coding agent.

## Loading rule

**The CTO never loads files from this directory into the conversation.**

Files here are bundled, not read. The CTO uses `../runtime/stack-practices.md` as the manifest to decide which subset of `payload/` to copy into the user's project as `<project>/references/`.

## Categories (MECE)

| Directory | Source | Contents |
|-----------|--------|----------|
| `standards/` | Google, Uber, Airbnb, official | Python, Go, JS, TS, Rust, SQL style guides |
| `frontend/` | Vercel, Vue/Nuxt | `react-nextjs/`, `vue-nuxt/` |
| `backend/` | samber, zhanymkanov | `golang/` (42 sub-skills), `python-fastapi/` |
| `design/` | Microsoft, Google, PostgreSQL | `api/`, `database/`, `system/` |
| `review/` | Google eng-practices | Code review (reviewer + author guides) |
| `security/` | utkusen | 13 SAST vulnerability detection skills |
| `methods/` | Addy Osmani | 17 engineering method skills |

## A note on nested `SKILL.md` files

Several upstream sources (samber's Go skills, Vercel's React/Next.js rules) ship as Claude Code skills with their own `SKILL.md` files. We preserve the upstream directory structure for traceability and source fidelity.

**These nested `SKILL.md` files are not loaded by Claude Code as standalone skills.** They sit inside `payload/` as plain documentation. Claude Code's skill discovery only looks at `skills/<name>/SKILL.md` — anything deeper is invisible to the loader.

When a downstream coding agent receives the bundled `references/`, it should read these files as documentation, not invoke them as skills.

## Bundling rules

The CTO follows these rules at delivery time (see `../runtime/stack-practices.md` for the full manifest):

- **Always bundle**: matching `standards/` (by language), `methods/`, `review/`, `security/`
- **Conditionally bundle**: `frontend/` (matching framework), `backend/` (matching framework), `design/` (matching concern: API design? schema design? system design?)
- **Output structure in user project**: flat — `<project>/references/{standards,frontend,backend,...}` — without the `payload/` wrapper
