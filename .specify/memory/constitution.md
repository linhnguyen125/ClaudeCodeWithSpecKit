<!--
Sync Impact Report — FlowPilot Constitution v0.1.0

Version change:  N/A → 1.0.0  (initial creation)
Type of bump:   MAJOR (initial ratification)
Ratified:       2026-04-05
Last Amended:   2026-04-05

Modified principles:  none (initial creation)
Added sections:       Core Principles (7), Technology Stack Constraints,
                       Security Requirements, Performance Standards,
                       Testing Standards, Git Workflow, Governance
Removed sections:     none

Templates requiring updates:
  ✅ .specify/templates/plan-template.md         — Constitution Check section present (no change needed)
  ✅ .specify/templates/spec-template.md         — no conflicts
  ✅ .specify/templates/tasks-template.md         — no conflicts
  ✅ .specify/templates/agent-file-template.md   — no conflicts
  ⚠️ .specify/templates/checklist-template.md    — no conflicts
  ⚠️ .specify/templates/constitution-template.md — preserved as read-only template

Runtime docs updated:
  ⚠️ FlowPilotUI/CLAUDE.md  — references existing rules; constitution principles are additive
  ⚠️ FlowPilotUI/.claude/rules/*.md — existing rules align with constitution principles; no change needed

Follow-up TODOs:
  TODO(PYTHON_BACKEND): Backend (Python/FastAPI) not yet implemented.
    All Python/backend code must comply with constitution once written.
    Define Python linting/formatting/tooling in a future amendment.
  TODO(BACKEND_TESTS): Backend test standards deferred until Python backend
    is implemented.
  TODO(PYTHON_STRUCTURE): Backend project structure (backend/ directory)
    must be defined when FastAPI backend is created.
-->

# FlowPilot Constitution

## Core Principles

### I. Strict Structural Adherence

FlowPilot maintains a strict separation between frontend and backend. All frontend code lives in `FlowPilotUI/` (Electron + Vite + Vue 3 + TypeScript). All backend code (Python + FastAPI) must live in a dedicated `backend/` directory at the repository root. Cross-boundary communication uses typed IPC channels only; no direct imports across the boundary.

All code must live in the correct layer:
- **Frontend layer** (`FlowPilotUI/`): Electron main process, preload scripts, Vue 3 renderer. No Python, no FastAPI imports.
- **Backend layer** (`backend/`): FastAPI routes, services, models. No Vue components, no IPC handlers.

**Rationale**: Mixing layers causes tight coupling, circular dependencies, and build complexity. A clear boundary enables independent scaling, testing, and deployment.

---

### II. Frontend Language Constraint

The frontend (Electron/Vite/Vue) MUST use only TypeScript and Vue 3. JavaScript is prohibited for new code. No transpiled languages (CoffeeScript, JSX without TypeScript, etc.). The `FlowPilotUI/` codebase must pass `npm run typecheck`, `npm run lint`, and `npm run format` before any commit.

**Rationale**: TypeScript eliminates an entire class of runtime bugs and enables confident refactoring. The existing codebase already enforces this; extending it as a constitutional principle prevents regression.

---

### III. Backend Language Constraint

The backend MUST use Python 3.10+ with FastAPI. Python code must pass `ruff check` and `ruff format` (or equivalent linter/formatter) before any commit. A `pyproject.toml` or equivalent must declare all dependencies pinned to known-compatible versions. Backend test tooling must be defined before backend development begins (see TODO).

**Rationale**: Constraining the backend stack prevents dependency sprawl and tooling fragmentation. FastAPI is the stated choice; deviation requires a constitutional amendment.

---

### IV. IPC-First Communication

All communication between the Electron main process and the renderer process MUST use typed IPC channels following the naming convention `domain:action-subaction` (e.g., `automation:run-task`). Every IPC handler MUST:
1. Validate all input arguments (type check, null check, bounds check).
2. Return a typed `IpcResponse<T>` shape: `{ success: boolean; data?: T; error?: string }`.
3. Use `async/await` with try-catch; never `.then()` chains in handlers.
4. Log errors with sufficient context, never with sensitive data.

**Rationale**: Typed IPC is the only safe bridge between sandboxed renderer and privileged main process. Input validation prevents attacks and runtime crashes. The existing `.claude/rules/03-electron-ipc.md` codifies this; this principle elevates it to binding rule.

---

### V. Security Defaults (Non-Negotiable)

The following Electron security settings MUST be true for ALL build configurations:

| Setting             | Required Value |
|---------------------|----------------|
| `sandbox`           | `true`         |
| `contextIsolation`  | `true`         |
| `nodeIntegration`    | `false`        |
| `webSecurity`       | `true`         |
| `allowRunningInsecureContent` | `false` |

Any deviation requires:
1. A written technical justification checked into the PR.
2. A documented mitigation for the increased attack surface.
3. Explicit mention in the Constitutional Check during plan review.

Sensitive data (tokens, credentials, file contents) MUST NOT be logged. IPC path traversal MUST be blocked on all file-handling handlers.

**Rationale**: Electron apps are a high-value attack surface by default. These settings are the industry-standard baseline; breaking them without documented justification is unacceptable.

---

### VI. Performance-First Design

Performance is not an afterthought. The following MUST be observed:
- **Lazy loading**: All non-critical routes and heavy components use dynamic imports.
- **Memoization**: Expensive computed values use `computed()`; avoid recalculating on every render.
- **List virtualization**: Any list expected to exceed 100 items MUST be virtualized.
- **No deep reactivity for large objects**: Use `ref()` with reassignment over deep `reactive()` for frequently updated data.
- **Debounce/throttle**: All scroll, resize, and input handlers that trigger computation or IPC calls MUST be debounced or throttled.
- **Profile before optimizing**: Optimization decisions must be backed by measurements, not intuition.

**Rationale**: Desktop apps compete with native performance expectations. Vue's reactivity system is powerful but can cause performance issues if misused. The existing `.claude/rules/07-performance.md` covers specifics; this principle mandates the discipline.

---

### VII. Lean Dependency Policy

No library may be added without explicit justification in the PR. Before adding a dependency:
1. Verify no existing dependency already provides the needed functionality.
2. Evaluate bundle-size impact (for frontend).
3. Check for active maintenance, known vulnerabilities (`npm audit`), and TypeScript support.
4. Ensure the dependency does not pull in conflicting versions of shared packages.

Frontend dependencies are declared in `FlowPilotUI/package.json`. Backend dependencies are declared in `backend/pyproject.toml`. Both MUST be audited regularly (`npm audit` / `pip-audit`).

**Rationale**: Every added dependency is a long-term maintenance and security obligation. FlowPilot's package.json is already lean (pinia, vue-router, tailwindcss, electron-toolkit). New dependencies must earn their place.

---

## Technology Stack Constraints

### Frontend Stack (FlowPilotUI/)

| Concern           | Technology                            |
|-------------------|---------------------------------------|
| Framework         | Electron + electron-vite              |
| UI Framework      | Vue 3 (Composition API, `<script setup>`) |
| Language          | TypeScript (strict mode, no `any`)    |
| State Management  | Pinia (Composition API style)         |
| Styling           | Tailwind CSS v4                       |
| Routing           | Vue Router                            |
| Linting           | ESLint + Prettier                     |
| Type Checking     | vue-tsc / tsc                         |
| Testing           | Vitest + Vue Test Utils               |

No other UI frameworks (React, Angular, Svelte) or state management libraries may be introduced without a constitutional amendment.

### Backend Stack (backend/)

| Concern           | Technology                            |
|-------------------|---------------------------------------|
| Language          | Python 3.10+                          |
| Framework         | FastAPI                               |
| Packaging         | pyproject.toml (hatch or poetry)      |
| Linting/Formatting| ruff                                  |
| Testing           | pytest (details deferred — see TODO)  |

Backend structure MUST be defined when the backend is created. Python tooling and conventions will be added via a future constitutional amendment.

---

## Security Requirements

- **Sandbox always on**: `sandbox: true` is the default for all BrowserWindow instances. Dev-mode overrides (`sandbox: false`) must never reach production builds.
- **Context isolation**: Renderer never has direct access to Node.js or Electron internals. All privileged operations go through IPC.
- **Input validation on all IPC handlers**: No exceptions. Validate type, length, nullability, and range.
- **Path traversal protection**: Any IPC handler that accepts file paths MUST reject `..`, `~`, and other path traversal sequences.
- **No secrets in logs**: Credentials, tokens, file contents, and PII must never appear in log output.
- **CSP headers**: Content Security Policy must be configured for the renderer.
- **External links**: All external URLs must open in the OS browser, not in the app window.
- **Dependency auditing**: `npm audit` (frontend) and `pip-audit` or `safety` (backend) must be run regularly and before releases.

---

## Performance Standards

- Initial load: < 2 seconds
- Time to Interactive: < 3 seconds
- IPC round-trip (handler → response): < 50ms for non-blocking operations
- Bundle size (frontend gzipped): < 500 KB
- Lazy-load all non-critical routes and heavy components
- Virtualize lists with > 100 items
- No synchronous blocking operations on the main thread

Performance regressions require investigation and justification. "Works on my machine" is not a valid justification.

---

## Testing Standards

### Frontend Testing (FlowPilotUI/)

| Level        | Tool            | Coverage Target            |
|--------------|-----------------|---------------------------|
| Unit         | Vitest          | Stores, composables, utilities |
| Component    | Vitest + Vue Test Utils | UI components (Fp-prefixed) |
| IPC Handler  | Vitest + vi.mock | All handler validation paths |
| Integration  | Vitest          | Cross-component flows      |

Co-located test files: `FpButton.vue` → `FpButton.spec.ts`. Tests MUST be committed alongside the code they test.

### Backend Testing (backend/)

Deferred until FastAPI backend is implemented. Python test tooling (pytest) and standards will be defined in a future amendment.

### CI Integration

All tests MUST pass before merging. The CI pipeline runs:
```bash
npm run typecheck && npm run lint && npm run test:ci
```

---

## Git Workflow

### Branch Strategy

```
main                ← production, always deployable
    └── develop     ← integration branch
          ├── feature/*    ← new features
          ├── fix/*        ← bug fixes
          ├── refactor/*   ← code improvements
          ├── docs/*       ← documentation only
          └── chore/*       ← maintenance, deps, tooling
```

### Commit Convention

```
<type>(<scope>): <subject>

<body>
```

Types: `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `chore`, `style`, `ci`, `revert`.

### Quality Gates (Pre-Commit)

Before opening a PR, ALL of the following MUST pass:
- `npm run typecheck`
- `npm run lint`
- `npm run format`
- `npm run test:ci` (when tests exist)
- `npm run build` (verification build, not required to pass on feature branches before review)

### Commit Rights

No force-pushes to `main`. All changes to `main` require a PR with at least one review.

---

## Governance

### Constitutional Supremacy

This constitution supersedes all informal development guidelines, comments, and convention documents. If a conflict exists between this constitution and any other document (including `.claude/rules/*.md`, `CLAUDE.md`, or `README.md`), this constitution takes precedence.

### Amendment Procedure

1. **Proposal**: Any team member may propose an amendment by opening a PR that modifies ``.specify/memory/constitution.md`` with a clear description of the change.
2. **Justification**: The PR must explain the rationale, the problem being solved, and any backward-incompatible implications.
3. **Review**: Amendments require approval from at least one other team member before merge.
4. **Version bump**: Amendments MUST bump the version number according to semantic versioning rules:
   - **MAJOR**: Backward-incompatible changes (removal or redefinition of principles, breaking structural changes).
   - **MINOR**: New principles or materially expanded guidance.
   - **PATCH**: Clarifications, wording fixes, non-semantic refinements.
5. **Sync propagation**: The amending PR must update all affected templates listed in the Sync Impact Report.
6. **Notification**: After merging, update any affected runtime docs (CLAUDE.md, rule files, etc.) and run `npm run typecheck && npm run lint` to verify no regressions.

### Compliance Review

- All PRs MUST include a "Constitutional Check" note confirming which principles are affected and that the implementation complies.
- The `/speckit.plan` command template includes a Constitutional Check gate that MUST be filled in for every feature plan.
- Non-compliant PRs will not be merged until the constitutional violation is resolved.

### Deferred Items

This constitution acknowledges the following items as TODO and will address them via amendment when the relevant work begins:

| Item                          | Reason Deferred                         |
|-------------------------------|----------------------------------------|
| Python backend tooling        | Backend not yet implemented             |
| Backend test standards        | Backend not yet implemented             |
| Backend project structure     | Backend not yet implemented             |
| FastAPI IPC integration plan  | Backend not yet implemented             |

### Runtime Guidance

For development guidance not covered in this constitution, consult:
- `FlowPilotUI/CLAUDE.md` — project overview, commands, architecture summary
- `FlowPilotUI/.claude/rules/` — detailed implementation rules for TypeScript, Vue 3, IPC, Pinia, error handling, security, performance, testing, Git workflow, and project structure

These files are supplementary and must not contradict this constitution.

---

**Version**: 1.0.0 | **Ratified**: 2026-04-05 | **Last Amended**: 2026-04-05
