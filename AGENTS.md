# AGENTS.md

This repo is a small monorepo with a Vite/React frontend and a Spring Boot backend.
Use this file as the working agreement for automated coding agents (and humans).

## Goals for agent work

- Be correct and boring: prefer small, verifiable changes over cleverness.
- Keep changes scoped: **do not modify frontend and backend in the same code-change set** unless the task explicitly requires it.
- Keep cross-repo awareness: plans may reference both sides, but implementations should be done one side at a time.

## Repo layout

- `front-todo-reac-spring/` — React 19 + Vite frontend (JavaScript + JSX)
- `TodoToUseReact/` — Spring Boot 4 backend (Java 21)
- `docker-compose.yml` — Local Postgres + backend + frontend containers

> Note: the frontend folder name is `front-todo-reac-spring/` (missing “t” in `react`). Do not “fix” the name unless explicitly requested.

## How to work (recommended workflow)

1) **Plan first (read-only):**
   - Identify the contract/API change (routes, request/response shapes, status codes).
   - List impact on both sides (backend + frontend).
   - Choose an implementation order (usually backend → frontend).
2) **Implement in one place:**
   - Make the backend change (and tests), commit.
3) **Then implement the other place:**
   - Update the frontend to match, commit.
4) **Verify:**
   - Run relevant tests and/or `docker compose up --build` to validate integration.

### Boundaries

- Do not “synchronize” changes by editing both codebases at once.
- Avoid breaking API changes unless explicitly allowed.
- Prefer updating existing code paths over adding new dependencies.

## Build, lint, test

### Docker Compose (repo root)

- Full stack: `docker compose up --build`
- Stop: `docker compose down`
- Remove volumes (destructive): `docker compose down -v`

### Frontend (Vite/React)

Run from `front-todo-reac-spring/`.

- Install: `npm ci` (preferred) or `npm install`
- Dev server: `npm run dev`
- Build: `npm run build`
- Preview build: `npm run preview`
- Lint: `npm run lint`

Tests:

- No frontend test runner is configured yet.
- If you add tests, document the command here and in `package.json`.

### Backend (Spring Boot)

Run from `TodoToUseReact/`.

- Build: `mvn clean package`
- Run: `mvn spring-boot:run`
- Tests (all): `mvn test`
- Tests (single class): `mvn -Dtest=TodoJustForFunApplicationTests test`
- Tests (single method): `mvn -Dtest=TodoJustForFunApplicationTests#methodName test`

Linting / formatting:

- No Java lint/format tool is configured (no Checkstyle/Spotless).
- If you introduce one, add its command here and document any required config.

## Git workflow

- Prefer small commits with clear messages.
- Keep unrelated formatting churn to a minimum.
- When changing API contracts:
  - document the change (see “API/contract changes” below),
  - update one side at a time (backend then frontend),
  - verify with compose (or equivalent local run).

## API/contract changes

- Treat the backend as the source of truth for request/response semantics.
- When changing request/response shapes or status codes:
  - Update backend DTOs + controller/service behavior first.
  - Then update frontend API calls and UI expectations.
- Keep error shapes consistent; avoid introducing silent failures.

If this repo gains a shared contract folder (recommended):
- Put shared API descriptions in `contracts/` (e.g., OpenAPI or JSON examples).
- Keep it updated before/alongside backend changes to reduce drift.

## OpenCode usage conventions (monorepo-friendly)

### Agents

Recommended roles (names are suggestions):

- `plan` — read-only analysis/design; may reference both frontend and backend.
- `backend-build` — makes code changes **only** under `TodoToUseReact/**`.
- `ui-build` — makes code changes **only** under `front-todo-reac-spring/**`.
- `review` — read-only code review; do not modify files.

When configuring agents, enforce scope via instructions and permissions (path-based if available).
Keep the “one side at a time” rule: the plan can mention both, but build agents should edit only their area.

### Rules files (progressive disclosure)

If this file grows too large, split detailed checklists into small rule files and reference them from here.
Keep this root file concise and focused on commands, structure, workflow, and boundaries.

## Agent skills (what they are and how to use them)

Agent skills are reusable instruction modules stored as folders containing `SKILL.md`.
Use skills for repeatable workflows (e.g., “API change checklist”, “PR review checklist”, “release notes”).

Recommended skills to add for this repo:

- `api-change-checklist` — steps for safe backend→frontend contract changes.
- `spring-endpoint-review` — verify validation, error handling, logging, and tests.
- `react-api-integration` — consistent Axios usage, loading/error states, and UI feedback.
- `docker-compose-sanity` — quick checks after integration changes.

Skill placement:

- Project skills: `.opencode/skills/<skill-name>/SKILL.md`
- Global skills: `~/.config/opencode/skills/<skill-name>/SKILL.md`

Each `SKILL.md` must start with YAML frontmatter including `name` and `description`.

## General coding conventions

- Follow existing patterns in the file you touch (indentation, quotes, semicolons).
- Keep functions small and focused; prefer clear names over comments.
- Avoid introducing new dependencies without a clear need.
- Keep API changes backward compatible unless the task explicitly allows breaking changes.

## Frontend code style (React)

Language/stack:

- JavaScript + JSX (no TypeScript).
- React function components and hooks.
- Vite + ESLint (flat config in `front-todo-reac-spring/eslint.config.js`).

Imports:

- External imports first, then local modules, then CSS.
- Keep import groups compact; avoid unused imports (eslint enforces this).

Formatting:

- The codebase mixes single/double quotes and semicolons.
- When editing, match the existing file's style to minimize churn.
- Keep JSX attributes readable; multi-line where needed.

Components:

- Prefer function components with named exports for pages, default exports for small components.
- Keep UI state in component state; derive values from state rather than duplicating.
- Use controlled inputs for forms; trim user input before submit.

State & async:

- Use `useState` for local state and `useEffect` for data fetching on mount.
- Wrap API calls in `try/catch/finally` and update loading/error flags.
- Use optimistic updates only when the UI can revert on failure.

API usage:

- Prefer the shared Axios instance (`front-todo-reac-spring/src/api/api.js`) when adding new API modules.
- Existing code sometimes calls `axios` directly in `TodoList.jsx`; follow local conventions unless refactoring.

Naming:

- `camelCase` for variables/functions, `PascalCase` for components.
- Boolean names should read as predicates (`isOpen`, `hasError`, `shouldFetch`).

Error handling:

- Surface errors in the UI where possible; avoid silent failures.
- When catching Axios errors, check `err.response?.status` and `err.response?.data`.

Styling:

- Plain CSS with class names (e.g., `todo-*`, `auth-*`).
- Keep class names semantic; avoid inline styles except small overrides.
- Update related CSS in `*.css` alongside component changes.

## Backend code style (Spring Boot)

Language/stack:

- Java 21 with Spring Boot 4.
- Lombok in entities, record-based DTOs in `dto` package.

Packages:

- Base package: `com.example.todojustforfun`.
- Keep controllers in `controllers`, services in `services`, repositories in `repositories`.

Imports:

- Use explicit imports (no wildcard) as seen in existing files.
- Keep Java imports grouped by standard, third-party, and project packages.

Formatting:

- 4-space indentation; braces on the same line.
- One class per file; file name matches class/record.

Controllers:

- Use `@RestController` and `ResponseEntity` for responses.
- Validate input with `@Valid` and Jakarta validation annotations on DTOs.
- Favor RESTful routes and HTTP verbs (see `TodoController`).

DTOs:

- Use `record` for request/response types.
- Add validation annotations on request records (`@NotBlank`, `@Size`, `@Email`).

Services:

- Prefer interface + `Impl` pattern with constructor injection.
- Keep business logic in services, not controllers.
- Use `@Transactional` for write operations.

Repositories:

- Use Spring Data repository method naming conventions.
- Prefer repository queries over custom SQL unless necessary.

Entities:

- Use JPA annotations and Lombok (`@Getter`, `@Setter`, `@NoArgsConstructor`).
- Keep fields private; let Lombok generate accessors.

Error handling:

- Current services throw `IllegalArgumentException` for not-found or invalid data.
- If adding new error cases, either continue using `IllegalArgumentException` or introduce a structured exception + `@ResponseStatus` or `@ControllerAdvice`.
- Avoid returning null from services unless the controller explicitly handles it (see `AuthController#me`).

Security:

- Authentication is session-based; keep session management consistent with `AuthController`.
- If adding secured endpoints, ensure `SecurityConfig` is updated accordingly.

Database & migrations:

- Flyway migrations live in `TodoToUseReact/src/main/resources/db/migration/`.
- New migrations should be sequential (`V5__...sql`) and additive.
- Keep schema changes aligned with JPA entities.

## What to document when you change things

- Add new build/test/lint commands to this file.
- If you add a lint/format tool, describe the command and any required config.
- Update notes for API/contract changes that affect both frontend and backend.
