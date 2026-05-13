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
| `backend/` | samber, zhanymkanov | `golang/` (single compressed playbook), `python-fastapi/` |
| `design/` | Microsoft, Google, PostgreSQL | `api/`, `database/`, `system/` |
| `review/` | Google eng-practices | Code review (reviewer + author guides) |
| `security/` | utkusen | 13 SAST vulnerability detection skills |
| `methods/` | Addy Osmani | 17 engineering method skills |

## A note on nested `SKILL.md` files

Some upstream sources (such as Vercel's React/Next.js rules) ship as Claude Code skills with their own `SKILL.md` files. Some agents ignore nested `SKILL.md` files, while installers and catalogs may still index them recursively. Keep nested `SKILL.md` files only when the payload truly needs to preserve a standalone upstream skill.

Inside `payload/`, nested `SKILL.md` files should be treated as plain documentation unless the receiving tool explicitly supports recursive skill discovery.

When a downstream coding agent receives the bundled `references/`, it should read these files as documentation, not invoke them as skills.

## Bundling rules

The CTO follows these rules at delivery time (see `../runtime/stack-practices.md` for the full manifest):

- **Always bundle**: matching `standards/` (by language), `methods/`, `review/`, `security/`
- **Conditionally bundle**: `frontend/` (matching framework), `backend/` (matching framework), `design/` (matching concern: API design? schema design? system design?)
- **Output structure in user project**: flat — `<project>/references/{standards,frontend,backend,...}` — without the `payload/` wrapper
