# CLAUDE.md — Sigma Workspace

## Workspace Overview

This is the **Sigma Workspace** — a meta-repository that bundles the Sigma platform codebase and AI agent configurations together for development with Claude Code.

| Path | Purpose |
|---|---|
| `.claude/agents/` | Agent definitions (part of this workspace) |
| `project-ops/` | Sigma platform codebase (submodule, working path: `d:\2026\sigma\sigma-workspace\project-ops\`) |

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

Agents live in `.claude/agents/` (part of this workspace — edit directly here):

| Agent | Trigger |
|---|---|
| `api-developer` | Creating or modifying API routes in `src/app/api/` |
| `prisma-expert` | Schema changes, migrations, Prisma queries |
| `ui-component` | Creating or modifying UI components |
| `code-reviewer` | Reviewing code before merging |
| `orchestrator` | Complex multi-step tasks spanning API + UI + DB |

---

## Workspace Rules

- **Before starting any task — summarize what will be done, which files will be affected, and wait for confirmation before proceeding**
- **Always create a new branch before making any code changes** — always branch from `develop`, never work directly on `main`, `master`, or `develop`
- **Never commit or push without explicit user confirmation**
- All code changes target `project-ops/` — agent files in `.claude/agents/` are part of this workspace and can be edited directly
- `project-ops/` is a git submodule on Bitbucket — always use `git -C project-ops` for git operations inside it
- The working path for project-ops is `d:\2026\sigma\sigma-workspace\project-ops\` — never edit files at `d:\2026\sigma\project-ops\` (standalone checkout, not used)
- `.claude/settings.local.json` is gitignored — copy from `settings.local.json.example` to get started:
  ```bash
  cp settings.local.json.example .claude/settings.local.json
  ```

### Workflow for Every Task

1. **Summarize the plan** — describe what will change, which files, any risks — wait for confirmation before writing any code
2. **Create branch** — always from `develop`:
   ```bash
   git -C project-ops checkout develop && git -C project-ops pull
   git -C project-ops checkout -b <type>/<ticket-or-description>
   ```

   Branch naming convention:
   ```
   feature/SIGMA-123-short-description    # new feature
   bugfix/SIGMA-456-short-description     # bug fix
   hotfix/SIGMA-789-short-description     # urgent production fix
   chore/short-description                # maintenance, deps, config
   refactor/short-description             # code refactor
   ```
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
