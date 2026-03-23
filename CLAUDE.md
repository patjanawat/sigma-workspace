# CLAUDE.md — Sigma Workspace

## Workspace Overview

This is the **Sigma Workspace** — a meta-repository that bundles the Sigma platform codebase and AI agent configurations together for development with Claude Code.

| Submodule | Path | Purpose |
|---|---|---|
| sigma-agents | `.claude/agents/` | Agent definitions (source of truth) |
| project-ops | `project-ops/` | Sigma platform codebase |

---

## Project Summary

**Sigma** is an enterprise infrastructure operations platform. It enables DevOps/infrastructure teams to manage projects, resources, deployments, templates, and access control across Azure cloud environments.

- **Framework**: Next.js 14 (App Router), TypeScript strict mode
- **Database**: PostgreSQL via Prisma 6
- **Auth**: NextAuth.js v5 (Keycloak, Entra ID, Credentials)
- **Package Manager**: Bun
- **Styling**: Tailwind CSS 3.4 + Radix UI

---

## Working in This Workspace

All code changes happen inside `project-ops/`:

```bash
# Install dependencies
cd project-ops && bun install

# Start dev server
cd project-ops && bun dev

# Run migrations
cd project-ops && bun run migrate:dev

# Run tests
cd project-ops && bun test
```

**Always `cd project-ops` before running any project commands.**

---

## Agents

Agents live in `.claude/agents/` (submodule from `sigma-agents` repo):

| Agent | Trigger |
|---|---|
| `api-developer` | Creating or modifying API routes in `src/app/api/` |
| `prisma-expert` | Schema changes, migrations, Prisma queries |
| `ui-component` | Creating or modifying UI components |
| `code-reviewer` | Reviewing code before merging |

To update agents to latest version:
```bash
git submodule update --remote .claude/agents
git add .claude/agents
git commit -m "chore: update agents submodule"
git push
```

---

## Workspace Rules

- **Before starting any task — summarize what will be done, which files will be affected, and wait for confirmation before proceeding**
- **Always create a new branch before making any code changes** — never work directly on `main` or `master`
- **Never commit or push without explicit user confirmation**
- All code changes target `project-ops/` — never modify agent files here (edit in `sigma-agents` repo instead)
- `project-ops/` is on Bitbucket — use `git -C project-ops` for git operations inside it
- `.claude/settings.local.json` is gitignored — each developer maintains their own

### Workflow for Every Task

1. **Summarize the plan** — describe what will change, which files, any risks — wait for confirmation before writing any code
2. **Create branch** — `git -C project-ops checkout -b <type>/<ticket-or-description>`
3. **Implement** — make changes
4. **Summarize what was done** — list files changed and a brief description of each change — wait for confirmation
5. **Commit & push** — only after explicit user confirmation

---

## Key Paths (inside project-ops/)

| Path | Purpose |
|---|---|
| `src/app/api/` | API routes (87 route handlers) |
| `src/components/` | UI components |
| `src/lib/middleware/` | Auth & RBAC middleware |
| `src/types/` | TypeScript types, Zod schemas, enums |
| `prisma/schema.prisma` | Database schema (35+ models) |
| `src/auth.ts` | NextAuth configuration |
