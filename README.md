# Best Practices PRD for Vibe Coding

> **A Complete Spec-Driven Blueprint for Building Production-Ready Apps with AI Agents (Gemini, OpenCode, Aider, Antigravity, Cursor)**

Writing a Product Requirements Document (PRD) for an AI coding assistant is fundamentally different from writing one for a human team. While a human PRD focuses on *business justification and high-level vision*, an **AI-First PRD** functions as a strict, context-dense architectural blueprint that eliminates ambiguity, prevents hallucinated dependencies, and stops architectural drift.

---

## 1. Core Philosophy: Why Tech Stack & Infrastructure Must Be Explicit

In traditional development, the PRD covers the *"What"* and *"Why"*, leaving the *"How"* to a separate Technical Design Document (TDD). In **Vibe Coding & AI-Assisted Engineering**, merging your PRD and TDD into a modular **Master Specification** is mandatory:

1. **Prevents Default-Stack Drift:** If you leave the stack blank, LLMs default to the highest-probability patterns in their training data (often bloated React/Next.js or legacy Express boilerplate). If you want a lean setup (e.g., **Bun + Elysia.js + Prisma + Tailwind**), you must lock those constraints in upfront.
2. **Aligns Code with Deployment Boundaries:** An AI writes very different code depending on where the app lives. Specifying whether the app runs on a **Dockerized Linux VPS (e.g., via Dokploy)**, uses **Cloudflare** proxies, or connects to managed PostgreSQL with strict SSL (**Neon / Aiven**) dictates how it structures environment variables, connection pooling, and file storage.
3. **Enforces Consistency Across Sessions:** Defining folder trees, API JSON envelopes, and error handling upfront ensures that Feature #15 written next week matches the exact architecture of Feature #1 written today.

---

## 2. Traditional PRD vs. AI-First PRD

| Dimension | Traditional Human PRD | AI-First PRD (Vibe Coding Spec) |
| :--- | :--- | :--- |
| **Primary Focus** | Market problem, user personas, KPIs | Data models, state transitions, strict constraints |
| **Tech Stack** | Omitted or high-level | Explicit down to package names and runtime APIs |
| **Feature Style** | Open-ended user stories (*"As a user..."*) | Deterministic `Input -> Validation -> Process -> Output` flows |
| **Ambiguity** | Resolved via team meetings | Fatal (causes hallucinated code and refactoring loops) |
| **Structure** | Single massive Word/Google Doc | Modular `.md` files in repo to prevent "Context Rot" |

---

## 3. Recommended Modular PRD Folder Structure

Feeding a single 3,000-line `PRD.md` into every prompt wastes API tokens and causes the AI to forget rules in the middle of the document ("Context Rot"). Instead, use a **Modular Spec-Driven Folder Structure** inside your repository:

```text
project-root/
├── AGENTS.md                     # 📌 AUTO-LOADED: Global rules, tech stack, commands (<150 lines)
├── GEMINI.md                     # (Or symlink to AGENTS.md depending on your CLI tool)
├── PROGRESS.md                   # 📌 STATE TRACKER: Current phase, active tasks, completed [x] log
│
├── specs/                        # 📂 THE MODULAR PRD DIRECTORY
│   ├── 00-overview.md            # Vision, User Roles (RBAC matrix), Core business rules
│   ├── 01-architecture.md        # Infra (Docker/VPS), Env vars, Folder structure, Security rules
│   ├── 02-data-model.md          # Complete DB Schema (Prisma/SQL), Enums, Indexing, Seed plan
│   ├── 03-api-conventions.md     # "Golden Examples" of JSON envelopes, Error codes, Auth headers
│   ├── 04-ui-design-system.md    # Color tokens, Typography, Mobile/Desktop rules, Empty/Loading states
│   │
│   ├── features/                 # 📂 FEATURE-SPECIFIC PRDs (Loaded 1 by 1 during coding)
│   │   ├── feat-01-auth.md       # Login, Register, Session/JWT, Password reset flows
│   │   ├── feat-02-dashboard.md  # Role-based views, KPI widgets, Data aggregation queries
│   │   ├── feat-03-core-crud.md  # Specific business module (Input -> Validation -> Output)
│   │   └── feat-04-reports.md    # Export PDF/Excel logic, Filtering, Pagination rules
│   │
│   └── qa/                       # 📂 VERIFICATION & TESTING SPECS
│       ├── test-scenarios.md     # Edge cases, Security/Tenant isolation checks, UAT checklist
│       └── seed-data-spec.md     # Exact dummy personas and test accounts required
│
├── src/                          # Application source code
└── package.json
```

---

## 4. The 7 Essential Pillars of an AI-Ready Spec

### Pillar 1: Strict Stack & "Forbidden Patterns" (`AGENTS.md`)
Tell the AI both what to use and what **never** to touch.
* **Explicit Stack:** Runtime, Backend Framework, ORM, Database, Frontend Bundler, Styling, Icon Library.
* **Forbidden List:** *"Do NOT use `axios` (use native `fetch`). Do NOT use `moment.js` (use `date-fns` or native `Intl`). Do NOT install unrequested npm packages without asking."*

### Pillar 2: Complete Data Model & Seed Strategy (`02-data-model.md`)
Never let an AI guess database relationships from prose.
* Provide a complete **Prisma Schema** or SQL DDL draft.
* Define exact Enums (e.g., `Role: SUPER_ADMIN | STAFF | USER`, `Status: PENDING | APPROVED | REJECTED`).
* Specify soft-delete rules (`deletedAt` timestamp) vs. hard deletes.
* **Mandatory Seed Script (`prisma/seed.ts`):** Require realistic dummy personas and test credentials so the app is testable immediately on boot.

### Pillar 3: "Golden Code Examples" (`03-api-conventions.md`)
LLMs mimic concrete code examples 10x better than abstract descriptions. Paste a 15-line snippet of your ideal API response envelope and controller pattern:
```json
{
  "success": true,
  "data": { "id": "usr_123", "role": "ADMIN" },
  "error": null,
  "meta": { "page": 1, "limit": 20, "total": 100 }
}
```

### Pillar 4: Deterministic Feature Flows (`specs/features/feat-XX.md`)
Write features as state machines:
1. **Trigger & Endpoint:** e.g., `POST /api/v1/submissions`
2. **Auth & RBAC Guard:** Allowed roles, tenant isolation rule (`where: { tenantId: user.tenantId }`).
3. **Validation Rules:** Exact field constraints (min/max, regex, file size limits).
4. **Happy Path Logic:** Database transaction steps, side effects (e.g., email/notification trigger).
5. **Edge Cases & Error Codes:** Explicit HTTP status codes (`400`, `401`, `403`, `409`, `422`).

### Pillar 5: UI/UX Tokens & State Completeness (`04-ui-design-system.md`)
Prevent generic or inconsistent UI generation:
* **Viewport Target:** Explicitly state **Mobile-First** (touch targets >= 44px, bottom nav) or **Desktop-First** (dense tables, sidebar nav).
* **The 4 Mandatory View States:** Every screen fetching data must implement:
  1. `Loading` (Skeleton shimmer)
  2. `Empty` (Zero-data illustration + Call-to-Action button)
  3. `Error` (Retry banner + friendly message)
  4. `Success` (Populated view)

### Pillar 6: Security & Multi-Tenancy Guardrails (`01-architecture.md`)
* **Zero PII Leakage:** Never return `passwordHash` or sensitive internal tokens in API payloads; never log raw request bodies to stdout.
* **Tenant Isolation:** Every query on multi-tenant tables must enforce `tenantId` scoping at the service layer.
* **Atomic Operations:** Require `prisma.$transaction()` for any operation modifying more than one table or handling financial/status balances.

### Pillar 7: Self-Verification & Definition of Done (`AGENTS.md` & `PROGRESS.md`)
Give the AI executable terminal commands to verify its own work before finishing a turn:
* Typecheck: `bun run tsc --noEmit`
* Linter: `bunx biome check .`
* Tests: `bun test`

---

## 5. Starter Boilerplate Templates (Copy & Paste)

### Template 1: `AGENTS.md` / `GEMINI.md` (Root Directory)

```markdown
# AI Agent Engineering Constitution

## 1. Tech Stack (Strict)
- **Runtime:** Bun (latest) - Use native `Bun.password`, `Bun.file`, and `bun test`.
- **Backend:** Elysia.js with TypeBox validation.
- **Database & ORM:** PostgreSQL 16 + Prisma ORM.
- **Frontend:** React + Vite + TypeScript (Strict mode) + Tailwind CSS + Lucide Icons.
- **Deployment Target:** Docker container on Linux VPS behind Cloudflare proxy.

## 2. Forbidden Patterns
- DO NOT use `express`, `axios`, `moment`, or `lodash`.
- DO NOT use `any` type in TypeScript.
- DO NOT install new packages without explicit confirmation.
- DO NOT hardcode secrets; validate all variables via `src/config/env.ts` at startup.

## 3. Target Source Structure
src/
├── config/           # env validation, constants
├── db/               # prisma client & seed script
├── middlewares/      # auth guard, RBAC, global error handler
├── modules/          # feature verticals (controller, service, schema)
└── shared/           # utils, standard response helpers

## 4. Workflow & Verification Rules
1. Before writing code, read `PROGRESS.md` and the active spec in `specs/features/`.
2. Follow the API response envelope in `specs/03-api-conventions.md`.
3. After implementing a task, ALWAYS run:
   - `bun run tsc --noEmit` (Must pass with 0 errors)
   - `bun test` (All unit/integration tests must pass)
4. Update `PROGRESS.md` by checking off `[x]` completed tasks before ending your turn.
```

---

### Template 2: `PROGRESS.md` (Root Directory)

```markdown
# Project Implementation Tracker

## Current Status
- **Active Phase:** Phase 1 - Foundation & Auth
- **Current Task:** `specs/features/feat-01-auth.md`

## Phase 1: Scaffolding & Database
- [x] Initialize Bun + TypeScript + Tailwind workspace
- [x] Configure `Dockerfile` and `.env.example`
- [ ] Implement `prisma/schema.prisma` from `specs/02-data-model.md`
- [ ] Create deterministic `prisma/seed.ts` with test accounts

## Phase 2: Authentication & RBAC (`feat-01-auth.md`)
- [ ] Implement `POST /api/auth/login` & JWT cookie middleware
- [ ] Implement Role Guard (`SUPER_ADMIN`, `ADMIN`, `USER`)
- [ ] Build Login UI with validation and error toast states
- [ ] Write integration tests for Auth routes

## Architectural Decisions Log
- *2026-10-24:* Selected HTTP-only cookies for JWT storage to prevent XSS token theft.
```

---

### Template 3: `specs/features/feat-01-example.md` (Feature Slice)

```markdown
# Feature 01: User Authentication & Session Management

## 1. Context & Dependencies
- **Touches Models:** `User`, `AuditLog` (see `specs/02-data-model.md`)
- **Allowed Roles:** Public (Login), Authenticated (Logout, `/me`)

## 2. API Endpoints

### `POST /api/v1/auth/login`
- **Input Schema:**
  - `email`: string (format: email, trim, lowercase)
  - `password`: string (min: 8, max: 72)
- **Process Logic:**
  1. Query `User` by `email` where `deletedAt IS NULL`.
  2. If not found or `isActive == false`, return `401 Unauthorized`.
  3. Verify password via `Bun.password.verify(password, user.passwordHash)`.
  4. Generate signed JWT (`sub: user.id`, `role: user.role`, `exp: 24h`) and set in `HttpOnly`, `Secure`, `SameSite=Lax` cookie.
  5. Insert record into `AuditLog` (`action: "USER_LOGIN"`).
- **Edge Cases & Errors:**
  - `400 Bad Request`: Validation failed (return field-level error map).
  - `401 Unauthorized`: `"Invalid email or password"` (never reveal which one failed).
  - `403 Forbidden`: `"Account is suspended. Contact administrator."`

## 3. UI Requirements (`/login`)
- Clean centered card layout (Mobile-first responsive).
- Show inline validation errors under inputs.
- Disable submit button and show spinner during API request.
- Redirect to role-specific dashboard on `200 OK`.

## 4. Acceptance Criteria (Definition of Done)
- [ ] Valid credentials set HTTP-only cookie and return user profile (excluding `passwordHash`).
- [ ] Invalid password returns `401` with standard error JSON envelope.
- [ ] Suspended user (`isActive: false`) receives `403`.
- [ ] Unit/Integration tests in `src/modules/auth/auth.test.ts` pass 100%.
```

---

## 6. Daily Vibe Coding Prompting Workflow

Once your repository is set up with this structure, use these four prompts to drive development cleanly from start to finish:

1. **The Architect Review (Before Coding):**
   > *"Read `AGENTS.md`, `specs/01-architecture.md`, and `specs/02-data-model.md`. Act as a Senior System Architect and identify any logical gaps, missing foreign keys, or security bottlenecks before we write code."*
2. **The Scaffolding Prompt (Phase 1):**
   > *"Read `AGENTS.md` and `specs/02-data-model.md`. Set up the project folder structure, `schema.prisma`, `.env.example`, and `prisma/seed.ts`. Verify with `bun run tsc --noEmit` and update `PROGRESS.md`."*
3. **The Feature Execution Prompt (Phase 2+):**
   > *"Read `AGENTS.md`, `PROGRESS.md`, and `specs/features/feat-01-auth.md`. Implement the backend routes, UI views, and tests for Feature 01 following `specs/03-api-conventions.md`. Run typecheck and tests when finished, then mark completed items in `PROGRESS.md`."*
4. **The Context Reset Prompt (New Chat Session):**
   > *"Read `AGENTS.md` and `PROGRESS.md` to understand our project rules and current state, then tell me what our next immediate task is."*
