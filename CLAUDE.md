# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MBA AI challenge project focused on transforming technical meeting transcriptions into a complete design documentation package for a webhook notification system feature. The challenge emphasizes using AI as a production tool while maintaining critical review and iteration.

The deliverable is **purely documentation** — no code modifications are allowed. The codebase serves as context and reference only.

## Challenge Structure

**Primary Artifact**: Design documentation package for an Order Management System webhook notification feature (`TRANSCRICAO.md`)

**Deliverables** (in `docs/`):
- `PRD.md` — Product requirements (business/problem level)
- `RFC.md` — Technical proposal (architecture level, 2-4 pages)
- `FDD.md` — Implementation details (developer actionable)
- `adrs/` — 5-8 Architecture Decision Records (decision level)
- `TRACKER.md` — Traceability matrix linking all items to source
- `README.md` — Process documentation (replaces challenge description)

## Common Development Commands

```bash
# Development
npm run dev              # Start with live reload (http://localhost:3000)
npm run build           # Compile TypeScript to dist/
npm run start           # Run compiled code
npm run lint            # ESLint check
npm run format          # Prettier format

# Database
npm run db:migrate      # Run Prisma migrations
npm run db:reset        # Wipe and reset database
npm run db:seed         # Populate seed data

# Testing
npm run test            # Run all tests once
npm run test:watch      # Watch mode (rerun on file change)
npm run test -- tests/orders.test.ts  # Single test file
npm run test -- -t "test name pattern"  # Tests matching pattern
```

## Codebase Architecture

### High-Level Structure

```
src/
├── app.ts                    # Express app setup (routes, middlewares)
├── server.ts                 # Server entrypoint (listening)
├── config/                   # Environment and database config
├── middlewares/              # Express middlewares (auth, logging, validation, error handling)
├── shared/                   # Shared utilities
│   ├── errors/              # Custom error classes (AppError, HTTP errors)
│   ├── http/                # HTTP response formatting
│   └── logger/              # Pino logger configuration
└── modules/                 # Feature modules (customers, products, orders, auth, users)
    └── orders/
        ├── order.service.ts       # Business logic (state transitions, changeStatus)
        ├── order.repository.ts    # Data access
        ├── order.controller.ts    # Route handlers
        ├── order.routes.ts        # Route definitions
        └── order.schemas.ts       # Zod validation schemas

prisma/                      # Database schema and migrations
tests/                       # Vitest test suite
```

### Key Architectural Patterns

**Modular Structure**: Each feature (orders, customers, products, auth, users) follows the same pattern: controller → service → repository. Routes are defined in a separate routes file.

**Error Handling**: 
- Custom `AppError` class in `src/shared/errors/app-error.ts` — base for all errors
- HTTP-specific errors in `src/shared/errors/http-errors.ts` (BadRequest, NotFound, Unauthorized, etc.)
- Centralized error middleware in `src/middlewares/error.middleware.ts` — catches and formats all errors

**Authentication**: JWT-based via `src/middlewares/auth.middleware.ts`. Includes `requireRole` middleware for authorization checks.

**Logging**: Pino logger configured in `src/shared/logger/index.ts`. Used via `pino-http` for request/response logging.

**Validation**: Zod schemas defined in module `*.schemas.ts` files, validated via `src/middlewares/validate.middleware.ts`.

**Database**: Prisma ORM with MySQL. Schema in `prisma/schema.prisma`. Migrations via `npx prisma migrate`.

**Order State Machine**: Orders follow a controlled lifecycle via `changeStatus` in `order.service.ts` — guards state transitions and manages transactional stock updates.

## Critical Patterns for Documentation Task

### The Webhook Feature Context

The challenge involves designing a webhook notification system that:
- Integrates with existing order state changes (`changeStatus` method)
- Uses Outbox pattern in MySQL for reliability
- Implements retry logic with DLQ (Dead Letter Queue)
- Provides HMAC-SHA256 authentication per webhook endpoint
- Guarantees at-least-once delivery via idempotency headers

### Key Integration Points (Reference for FDD)

When referencing existing code in documentation:

1. **Order State Transitions** — `src/modules/orders/order.service.ts::changeStatus()` — integrate webhook events here
2. **Error Handling** — `src/shared/errors/http-errors.ts` — webhook errors should follow existing pattern (e.g., `WebhookError extends AppError`)
3. **Logger** — `src/shared/logger/index.ts` — use for webhook processing observability
4. **Middleware Auth** — `src/middlewares/auth.middleware.ts::requireRole()` — model for webhook endpoint authorization if needed
5. **Validation** — `src/modules/*/[name].schemas.ts` — define webhook schemas similarly
6. **Error Middleware** — `src/middlewares/error.middleware.ts` — already centralizes HTTP error responses

### Transcription as Source of Truth

Always trace documentation back to `TRANSCRICAO.md`:
- Use timestamps `[hh:mm] Speaker: text` format when citing
- Distinguish between: explicitly decided, explicitly deferred, explicitly excluded
- Challenge vague or generic statements with specific references to what was discussed

### Tracker Requirements

The `docs/TRACKER.md` is your anti-hallucination safeguard:
- Every requirement/decision must map to `TRANSCRICAO` (timestamp) or `CODIGO` (file path)
- If you can't fill the "Source" and "Localização" columns, the content should not be in the docs
- Aim for 80%+ traceability

## Testing Approach for This Task

No code changes, so no unit tests to run. Focus instead on:

1. **Consistency checks**: Do RFC alternatives match what's in the transcript? Do FDD examples reference real code paths?
2. **Traceability checks**: Can every line in PRD/RFC/FDD be traced to TRANSCRICAO or CODIGO?
3. **Completeness checks**: Are all 6 major decisions covered in ADRs? Are "out of scope" items explicitly mentioned?
4. **Cross-reference checks**: Do RFC links to ADRs resolve? Do FDD integrations reference real files?

## Iteration Workflow

The challenge explicitly expects 3-5 cycles of: generate → review critically → refine prompts → regenerate.

**Antipatterns to avoid**:
- Accepting AI output without verification against source
- Generic/boilerplate language (especially in RFC alternatives or ADR consequences)
- Missing concrete examples or code references
- Duplicated content across doc levels (PRD detail shouldn't repeat in RFC)

**Quality signals**:
- Specific trade-offs named (not "pro: good, con: bad")
- Every major claim traceable to TRANSCRICAO timestamp or CODIGO path
- RFC is concise (~2-4 pages) and focuses on decisions; FDD is detailed and implementation-focused
- ADRs each defend one decision with real alternatives considered and explicit consequences

## No Code Modifications

Absolute constraint: Do not alter `src/`, `prisma/`, `tests/`, or configuration files. The codebase is reference material only. All work is in `docs/` and updated `README.md`.

## Key Files to Reference When Writing Docs

- `TRANSCRICAO.md` — The meeting transcript (source of requirements and decisions)
- `src/modules/orders/order.service.ts` — Order state machine and changeStatus method
- `src/shared/errors/` — Error class hierarchy
- `src/middlewares/auth.middleware.ts` — Authentication/authorization patterns
- `src/shared/logger/index.ts` — Logging infrastructure
- `prisma/schema.prisma` — Database schema for webhook events storage planning
