# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Paperclip is an open-source orchestration platform for zero-human companies — a control plane that coordinates AI agents (OpenClaw, Claude Code, Codex, Cursor) into org charts with goals, budgets, governance, and task management.

## Key Commands

```bash
# Development
pnpm dev              # Full dev (API + UI, watch mode)
pnpm dev:once         # Full dev without file watching
pnpm dev:server       # Server only
pnpm dev:ui           # UI only
pnpm build            # Build all packages
pnpm typecheck        # Type checking across all packages

# Testing
pnpm test             # Run tests in watch mode
pnpm test:run        # Run tests once

# Database
pnpm db:generate      # Generate new migration
pnpm db:migrate       # Apply migrations

# CLI
pnpm paperclipai run           # One-command local run (auto-onboard + doctor + start)
pnpm paperclipai doctor         # Health checks with repair flag
pnpm paperclipai configure      # Configure settings
```

## Monorepo Structure

```
├── server/           # Express REST API and orchestration services
├── ui/               # React + Vite board UI (served by API in dev)
├── cli/              # CLI tool for setup and client operations
├── packages/
│   ├── db/           # Drizzle schema, migrations, DB clients (PostgreSQL/PGlite)
│   ├── shared/       # Shared types, constants, validators, API path constants
│   ├── adapters/    # Agent adapters
│   │   ├── claude-local/
│   │   ├── codex-local/
│   │   ├── cursor-local/
│   │   ├── openclaw-gateway/
│   │   ├── opencode-local/
│   │   └── pi-local/
│   └── adapter-utils/
├── doc/              # Operational and product documentation
└── docs/             # Mintlify public docs
```

## Architecture Patterns

### Company-Scoped Design
Every domain entity (agents, projects, goals, issues) is scoped to a company. Company boundaries must be enforced in all routes/services.

### Control Plane Invariants
- **Single-assignee tasks**: Issues have one assignee; atomic checkout required for `in_progress` transition
- **Approval gates**: Board must approve hires and CEO strategy proposals
- **Budget enforcement**: Monthly budgets with hard-stop auto-pause when exceeded
- **Activity logging**: All mutating actions are logged for audit

### Data Flow
1. Board creates company → defines goals → hires/creates agents
2. Agents receive tasks via heartbeat invocations
3. Task execution tracked through issues/comments with full audit visibility
4. Cost events ingested and rolled up per agent/task/project/company
5. Board monitors via dashboard and can intervene (pause, override, terminate)

## Before Making Changes

Read these in order:
1. `doc/GOAL.md` - Product vision
2. `doc/PRODUCT.md` - Product details
3. `doc/SPEC-implementation.md` - V1 build contract (authoritative for V1)
4. `doc/DEVELOPING.md` - Dev setup and workflows
5. `doc/DATABASE.md` - Schema documentation

## Database Changes

When modifying the data model:

1. Edit schema files in `packages/db/src/schema/`
2. Ensure new tables are exported from `packages/db/src/schema/index.ts`
3. Generate migration: `pnpm db:generate`
4. Validate: `pnpm -r typecheck`

Note: `packages/db/drizzle.config.ts` reads compiled schema from `dist/schema/*.js`, so the db package is compiled before migration generation.

## Contract Synchronization

If you change schema/API behavior, update all impacted layers:
- `packages/db` - schema and exports
- `packages/shared` - types/constants/validators
- `server` - routes and services
- `ui` - API clients and components

## Verification Before Hand-off

Run this full check:
```bash
pnpm -r typecheck
pnpm test:run
pnpm build
```

## Local Development

- **Database**: Leave `DATABASE_URL` unset to use embedded PostgreSQL at `~/.paperclip/instances/default/db`
- **Storage**: Default is `local_disk` at `~/.paperclip/instances/default/data/storage`
- **Agent workspaces**: Fall back to `~/.paperclip/instances/default/workspaces/<agent-id>`

Override with `PAPERCLIP_HOME` and `PAPERCLIP_INSTANCE_ID` environment variables.

## API Conventions

- Base path: `/api`
- Board access = full-control operator context
- Agent access = bearer API keys (`agent_api_keys`), hashed at rest
- Return consistent HTTP errors: `400/401/403/404/409/422/500`
- Write activity log entries for all mutations
- Apply company access checks and enforce actor permissions (board vs agent)

## Deployment Modes

See `doc/DEPLOYMENT-MODES.md` for mode definitions. The canonical model is `local_trusted` and `authenticated` with `private/public` exposure policy.

## Additional Resources

- `AGENTS.md` - Detailed contributor guidance
- `CONTRIBUTING.md` - Contributing guidelines
- `README.md` - Quickstart and feature overview
