
# Implementation Plan: RNA-seq Workflow Runner

**Branch**: `001-build-a-barebones` | **Date**: 2025-10-02 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-build-a-barebones/spec.md`

## Execution Flow (/plan command scope)
```
1. Load feature spec from Input path
   → If not found: ERROR "No feature spec at {path}"
2. Fill Technical Context (scan for NEEDS CLARIFICATION)
   → Detect Project Type from file system structure or context (web=frontend+backend, mobile=app+api)
   → Set Structure Decision based on project type
3. Fill the Constitution Check section based on the content of the constitution document.
4. Evaluate Constitution Check section below
   → If violations exist: Document in Complexity Tracking
   → If no justification possible: ERROR "Simplify approach first"
   → Update Progress Tracking: Initial Constitution Check
5. Execute Phase 0 → research.md
   → If NEEDS CLARIFICATION remain: ERROR "Resolve unknowns"
6. Execute Phase 1 → contracts, data-model.md, quickstart.md, agent-specific template file (e.g., `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for GitHub Copilot, `GEMINI.md` for Gemini CLI, `QWEN.md` for Qwen Code or `AGENTS.md` for opencode).
7. Re-evaluate Constitution Check section
   → If new violations: Refactor design, return to Phase 1
   → Update Progress Tracking: Post-Design Constitution Check
8. Plan Phase 2 → Describe task generation approach (DO NOT create tasks.md)
9. STOP - Ready for /tasks command
```

**IMPORTANT**: The /plan command STOPS at step 7. Phases 2-4 are executed by other commands:
- Phase 2: /tasks command creates tasks.md
- Phase 3-4: Implementation execution (manual or via tools)

## Summary
Build a barebones web application that enables bench scientists to upload CSV sample sheets, validate them with row-level feedback, execute a pinned RNA-seq workflow, and access HTML reports. The system uses a UI-first + contract-first approach with FastAPI backend, SvelteKit frontend, and SQLite storage for rapid parallel development.

## Technical Context
**Language/Version**: Python 3.x (backend), TypeScript 5.x with Node LTS (frontend)
**Primary Dependencies**: FastAPI, SvelteKit, SQLite, Vanilla Extract, zod, papaparse, @tanstack/svelte-query
**Storage**: SQLite for runs database, static file server for HTML reports
**Testing**: pytest (unit), Playwright (E2E), Prism (mocking)
**Target Platform**: Web application (modern browsers)
**Project Type**: web - frontend + backend structure
**Performance Goals**: <2s CSV validation for small files, 1s status polling interval
**Constraints**: Single pinned workflow, shared passphrase auth, 30-day auto-deletion
**Scale/Scope**: v0 MVP - max 12 samples per run, 2 user roles (Admin/Runner)
**Tooling**: mise for version management, uv for Python, pnpm workspaces for frontend, strict typing throughout

## Constitution Check
*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Verify compliance with the Workflow Runner Constitution (v1.1.0):

**Code Quality & Tooling**
- [x] .mise.toml exists with pinned versions (Node, Python, uv, pnpm, pyright, ruff, Biome)
- [x] No tool versions specified outside mise

**Type Safety**
- [x] Frontend uses strict TypeScript + svelte-check
- [x] Backend uses pyright
- [x] No unjustified `any` or `# type: ignore`

**Linting & Formatting**
- [x] `mise run lint` command configured
- [x] ruff for Python (lint + format)
- [x] Biome for TS/JS/JSON

**Testing & Contracts**
- [x] Unit tests planned for CSV validation
- [x] Contract tests enforced against OpenAPI
- [x] Playwright smoke test for Upload → Report path
- [x] CI runs on every PR

**UX Consistency**
- [x] Flow follows Upload → Validate → Run → Report
- [x] Error messages include row + column
- [x] Forms are accessible (labels, focus, keyboard)

**Performance**
- [x] First paint target < 2s on broadband
- [x] Validation target ≤ 2s for 200 rows
- [ ] Status polling at 10s with exponential backoff (USER SPECIFIED: 1s polling for better UX)

**Styling System**
- [x] No utility CSS frameworks (no Tailwind/UnoCSS)
- [x] Vanilla Extract for tokens/themes
- [x] Svelte scoped styles for components
- [x] Lightning CSS in build

**Reproducibility & Security**
- [x] Workflow version pinned
- [x] Container images use sha256 digests (no :latest)
- [x] Run manifest is immutable
- [x] "Re-run exact" feature planned
- [x] No secrets in repo

**Scope Control**
- [x] v0 limited to one workflow + one preset
- [x] Feature has specification document

**Commit Standards**
- [x] Commit messages follow Conventional Commits format (type(scope): description)
- [x] Valid types: feat, fix, docs, style, refactor, test, chore, perf, ci, build, revert
- [x] Breaking changes marked with `!` or `BREAKING CHANGE:` in body

*Any violations must be documented in Complexity Tracking section*

## Project Structure

### Documentation (this feature)
```
specs/[###-feature]/
├── plan.md              # This file (/plan command output)
├── research.md          # Phase 0 output (/plan command)
├── data-model.md        # Phase 1 output (/plan command)
├── quickstart.md        # Phase 1 output (/plan command)
├── contracts/           # Phase 1 output (/plan command)
└── tasks.md             # Phase 2 output (/tasks command - NOT created by /plan)
```

### Source Code (repository root)
```
backend/
├── src/
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── run.py
│   │   └── sample_sheet.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── validator.py
│   │   ├── runner.py
│   │   └── auth.py
│   └── api/
│       ├── __init__.py
│       ├── main.py
│       ├── routes/
│       │   ├── validation.py
│       │   ├── runs.py
│       │   └── auth.py
│       └── schemas/
│           ├── validation.py
│           └── runs.py
├── tests/
│   ├── unit/
│   │   └── test_validator.py
│   └── contract/
│       └── test_openapi.py
└── static/
    └── reports/

frontend/
├── src/
│   ├── routes/
│   │   ├── +layout.svelte
│   │   ├── +page.svelte (login)
│   │   ├── runs/
│   │   │   ├── +page.svelte
│   │   │   ├── new/
│   │   │   │   └── +page.svelte
│   │   │   └── [id]/
│   │   │       └── +page.svelte
│   ├── lib/
│   │   ├── components/
│   │   │   ├── CSVUploader.svelte
│   │   │   ├── ValidationResults.svelte
│   │   │   ├── RunStatus.svelte
│   │   │   └── ProvenanceBox.svelte
│   │   ├── stores/
│   │   │   └── auth.ts
│   │   └── schemas/
│   │       └── csv.ts
├── tests/
│   └── e2e/
│       └── upload-to-report.spec.ts
└── static/
```

**Structure Decision**: Web application structure with separate backend (FastAPI) and frontend (SvelteKit) directories for parallel development. Contract-first approach using OpenAPI specification.

## Phase 0: Outline & Research
1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:
   ```
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

## Phase 1: Design & Contracts
*Prerequisites: research.md complete*

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Generate contract tests** from contracts:
   - One test file per endpoint
   - Assert request/response schemas
   - Tests must fail (no implementation yet)

4. **Extract test scenarios** from user stories:
   - Each story → integration test scenario
   - Quickstart test = story validation steps

5. **Update agent file incrementally** (O(1) operation):
   - Run `.specify/scripts/bash/update-agent-context.sh claude`
     **IMPORTANT**: Execute it exactly as specified above. Do not add or remove any arguments.
   - If exists: Add only NEW tech from current plan
   - Preserve manual additions between markers
   - Update recent changes (keep last 3)
   - Keep under 150 lines for token efficiency
   - Output to repository root

**Output**: data-model.md, /contracts/*, failing tests, quickstart.md, agent-specific file

## Phase 2: Task Planning Approach
*This section describes what the /tasks command will do - DO NOT execute during /plan*

**Task Generation Strategy**:
The /tasks command will generate approximately 30-35 ordered tasks by analyzing:
1. OpenAPI contract → Generate contract tests and API endpoints
2. Data model → Create database schema and model classes
3. UI components from user flow → Build frontend pages and components
4. Validation logic → Implement CSV validation with tests
5. Stub workflow runner → Create background task system

**Task Categories & Sequencing**:
1. **Setup Tasks** (1-5): Project initialization, dependencies, configuration
2. **Database Tasks** (6-10): Schema creation, models, migrations
3. **Contract Tasks** (11-15): OpenAPI tests, type generation [P]
4. **Backend Core** (16-20): Auth, validation service, API routes
5. **Frontend Core** (21-25): Layout, auth store, components [P]
6. **Integration** (26-30): Connect frontend to backend, polling setup
7. **Testing** (31-35): E2E tests, performance validation

**Ordering Principles**:
- Infrastructure before features
- Contracts before implementation (TDD approach)
- Backend API before frontend consumption
- Core features before admin features
- [P] marks indicate parallelizable tasks

**Key Task Slices** (from user input):
1. Initialize .mise.toml with Node, Python, uv, pnpm, pyright, ruff, biome, svelte-check, prism; add CI tasks
2. Write OpenAPI + 3 example responses per endpoint; stand up Prism mock; generate types
3. Scaffold SvelteKit with Vanilla Extract + Lightning CSS build; create /runs/new with CSV preview + zod client checks
4. Implement POST /validate (backend) with real CSV rules; wire UI to server validation
5. Implement POST /runs (stub runner) + manifest persistence; return run_id
6. Build /runs/[id] status page + polling (svelte-query); render provenance + stub report
7. Add Playwright smoke; harden lints/typechecks; ship /runs list + simple login guard

**Estimated Output**:
- 35 numbered tasks in dependency order
- Clear prerequisites for each task
- Parallel execution opportunities marked
- Acceptance criteria from quickstart scenarios

**IMPORTANT**: This phase is executed by the /tasks command, NOT by /plan

## Phase 3+: Future Implementation
*These phases are beyond the scope of the /plan command*

**Phase 3**: Task execution (/tasks command creates tasks.md)  
**Phase 4**: Implementation (execute tasks.md following constitutional principles)  
**Phase 5**: Validation (run tests, execute quickstart.md, performance validation)

## Complexity Tracking
*Fill ONLY if Constitution Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| 1s polling interval (vs 10s) | User specified real-time updates for better UX | 10s polling would feel sluggish for workflow monitoring |


## Progress Tracking
*This checklist is updated during execution flow*

**Phase Status**:
- [x] Phase 0: Research complete (/plan command)
- [x] Phase 1: Design complete (/plan command)
- [x] Phase 2: Task planning complete (/plan command - describe approach only)
- [ ] Phase 3: Tasks generated (/tasks command)
- [ ] Phase 4: Implementation complete
- [ ] Phase 5: Validation passed

**Gate Status**:
- [x] Initial Constitution Check: PASS (with documented violations)
- [x] Post-Design Constitution Check: PASS
- [x] All NEEDS CLARIFICATION resolved
- [x] Complexity deviations documented

---
*Based on Constitution v1.1.0 - See `.specify/memory/constitution.md`*
