# Implementation Tasks: RNA-seq Workflow Runner

**Feature Branch**: `001-build-a-barebones`
**Generated**: 2025-10-02
**Total Tasks**: 40
**Estimated Hours**: 60-80

## Overview
Build a barebones web application for RNA-seq workflow execution with CSV validation, status tracking, and report generation. Implementation follows UI-first + contract-first approach with parallel frontend/backend development.

## Tech Stack
- **Backend**: FastAPI, Python 3.x, SQLite, pyright, ruff, uv
- **Frontend**: SvelteKit, TypeScript, Vanilla Extract, @tanstack/svelte-query, zod
- **Testing**: pytest, Playwright, Prism
- **Tooling**: mise, pnpm workspaces, Biome

## Task Execution

### Phase 1: Project Setup & Tooling (T001-T005)
**Can run in parallel**: None (foundational)
**Estimated time**: 2-3 hours

#### T001: Initialize project structure and mise tooling
**File**: `.mise.toml`, `package.json`, `pyproject.toml`
- Create root `.mise.toml` with pinned versions:
  - Node LTS (20.x)
  - Python 3.11
  - uv 0.4.x
  - pnpm 8.x
  - pyright 1.1.x
  - ruff 0.5.x
  - biome 1.8.x
  - svelte-check 3.x
  - prism 5.x
- Create directory structure:
  ```
  backend/
    src/api/, src/models/, src/services/, tests/
  frontend/
    src/routes/, src/lib/, tests/
  ```
- Initialize git repository with .gitignore

#### T002: Configure backend project with uv
**File**: `backend/pyproject.toml`, `backend/requirements.txt`
**Prerequisites**: T001
- Initialize uv project in backend/
- Add dependencies to pyproject.toml:
  - fastapi[standard]==0.115.0
  - uvicorn==0.30.0
  - pydantic==2.8.0
  - python-multipart==0.0.9
  - aiosqlite==0.20.0
  - pytest==8.3.0
  - pytest-asyncio==0.24.0
  - httpx==0.27.0
- Configure pyright in pyproject.toml (strict mode)
- Configure ruff with line length 100, target Python 3.11

#### T003: Configure frontend project with SvelteKit
**File**: `frontend/package.json`, `frontend/svelte.config.js`
**Prerequisites**: T001
- Initialize SvelteKit project with TypeScript
- Add dependencies:
  - @sveltejs/kit@2.x
  - @vanilla-extract/css@1.15.x
  - @tanstack/svelte-query@5.x
  - zod@3.23.x
  - papaparse@5.4.x
  - openapi-typescript@7.x
- Configure TypeScript strict mode in tsconfig.json
- Setup Vite with Lightning CSS plugin

#### T004: Setup linting and formatting commands
**File**: `.mise.toml` (tasks section)
**Prerequisites**: T002, T003
- Add mise tasks:
  ```toml
  [tasks.lint]
  run = ["ruff check backend/", "biome check frontend/"]

  [tasks.typecheck]
  run = ["pyright backend/", "svelte-check --tsconfig frontend/tsconfig.json", "tsc --noEmit -p frontend/"]

  [tasks.test]
  run = ["pytest backend/tests/", "pnpm --dir frontend test"]
  ```
- Configure Biome for frontend (no semicolons, single quotes)
- Setup pre-commit hooks for linting

#### T005: Create CI/CD pipeline configuration
**File**: `.github/workflows/ci.yml`
**Prerequisites**: T004
- Setup GitHub Actions workflow:
  - Checkout code
  - Install mise
  - Run `mise install`
  - Run `mise run lint`
  - Run `mise run typecheck`
  - Run `mise run test`
- Add branch protection rules for main branch

### Phase 2: API Contract & Mocking (T006-T010)
**Can run in parallel**: T006-T008 [P]
**Estimated time**: 3-4 hours

#### T006: Create OpenAPI specification [P]
**File**: `contracts/openapi.yaml`
- Define complete OpenAPI 3.0 spec with:
  - POST /api/auth/login
  - POST /api/validate
  - POST /api/runs
  - GET /api/runs
  - GET /api/runs/{id}
  - DELETE /api/runs/{id}
  - POST /api/runs/{id}/cancel
  - POST /api/runs/{id}/rerun
- Add 3 example responses per endpoint (success, validation error, server error)
- Define all schemas with strict types

#### T007: Generate TypeScript types from OpenAPI [P]
**File**: `frontend/src/lib/types/api.ts`
**Prerequisites**: T006
- Run openapi-typescript to generate types
- Create type-safe API client wrapper
- Export request/response interfaces

#### T008: Generate Pydantic models from OpenAPI [P]
**File**: `backend/src/api/schemas/`
**Prerequisites**: T006
- Create Pydantic models for all schemas:
  - AuthRequest, AuthResponse
  - ValidationRequest, ValidationResponse
  - RunCreateRequest, RunResponse
  - ErrorResponse
- Add custom validators for business rules

#### T009: Setup Prism mock server
**File**: `scripts/start-mock.sh`
**Prerequisites**: T006
- Create script to start Prism with OpenAPI spec
- Configure dynamic response selection
- Add to mise tasks for easy startup

#### T010: Create API contract tests
**File**: `backend/tests/contract/test_openapi.py`
**Prerequisites**: T006, T008
- Use schemathesis to validate API against OpenAPI
- Create fixture for test client
- Test all endpoints match specification
- These tests should fail initially (TDD approach)

### Phase 3: Database & Models (T011-T015)
**Can run in parallel**: T012-T015 [P] after T011
**Estimated time**: 4-5 hours

#### T011: Create database schema and migrations
**File**: `backend/src/models/database.py`, `backend/migrations/`
- Create SQLite database setup with aiosqlite
- Define schema creation script:
  - users table
  - runs table
  - sample_sheets table
  - reports table
  - provenance table
  - sessions table
- Add indexes for performance
- Create CLI command for initialization

#### T012: Implement User model [P]
**File**: `backend/src/models/user.py`
**Prerequisites**: T011
- Create User SQLAlchemy model
- Add methods: create(), get_by_id(), update_last_login()
- Implement role validation (Admin/Runner)
- Add session management methods

#### T013: Implement Run model [P]
**File**: `backend/src/models/run.py`
**Prerequisites**: T011
- Create Run model with status state machine
- Add methods: create(), update_status(), get_by_user()
- Implement 30-day auto-deletion logic
- Add JSON field handling for sample_data

#### T014: Implement SampleSheet model [P]
**File**: `backend/src/models/sample_sheet.py`
**Prerequisites**: T011
- Create SampleSheet model
- Add validation error storage
- Implement relationship to Run
- Add methods for validation status

#### T015: Implement Report and Provenance models [P]
**File**: `backend/src/models/report.py`, `backend/src/models/provenance.py`
**Prerequisites**: T011
- Create Report model with file path tracking
- Create Provenance model with immutable fields
- Add manifest hash generation
- Implement cascade delete with Run

### Phase 4: Core Services (T016-T020)
**Can run in parallel**: T017-T019 [P] after T016
**Estimated time**: 6-8 hours

#### T016: Create CSV validation service
**File**: `backend/src/services/validator.py`
- Implement CSV validation rules:
  - Check required headers (sample_id, group, read1_uri, read2_uri)
  - Validate unique sample_ids
  - Check .fastq.gz extensions
  - Verify file existence and read permissions
  - Max 12 rows validation
- Return row-level error details
- Add unit tests in `backend/tests/unit/test_validator.py`

#### T017: Create authentication service [P]
**File**: `backend/src/services/auth.py`
**Prerequisites**: T012
- Implement passphrase validation
- Create session management (24hr expiry)
- Add role-based access control helpers
- Generate secure session IDs

#### T018: Create workflow runner service (stub) [P]
**File**: `backend/src/services/runner.py`
**Prerequisites**: T013
- Create background task system
- Implement state transitions (queued→running→done)
- Add mock delay (5-15 seconds)
- Generate placeholder MultiQC HTML report
- Store provenance metadata

#### T019: Create report storage service [P]
**File**: `backend/src/services/storage.py`
**Prerequisites**: T015
- Setup static file serving directory
- Implement report file management
- Add cleanup job for 30-day retention
- Generate public URLs for reports

#### T020: Unit tests for all services
**File**: `backend/tests/unit/test_*.py`
**Prerequisites**: T016, T017, T018, T019
- Write comprehensive unit tests
- Mock database interactions
- Test validation edge cases
- Test authentication flows
- Achieve >80% coverage

### Phase 5: API Implementation (T021-T025)
**Can run in parallel**: None (depends on services)
**Estimated time**: 5-6 hours

#### T021: Implement authentication endpoint
**File**: `backend/src/api/routes/auth.py`
**Prerequisites**: T017
- Create POST /api/auth/login endpoint
- Validate passphrase
- Create session cookie
- Return user info and role

#### T022: Implement validation endpoint
**File**: `backend/src/api/routes/validation.py`
**Prerequisites**: T016
- Create POST /api/validate endpoint
- Parse multipart form upload
- Run CSV validation
- Return row-level errors

#### T023: Implement runs creation endpoint
**File**: `backend/src/api/routes/runs.py` (POST /api/runs)
**Prerequisites**: T018
- Create POST /api/runs endpoint
- Validate CSV before creating run
- Start background runner task
- Return run_id and initial status

#### T024: Implement runs query endpoints
**File**: `backend/src/api/routes/runs.py` (GET endpoints)
**Prerequisites**: T013
- Create GET /api/runs endpoint with filtering
- Create GET /api/runs/{id} with full details
- Include provenance data
- Add report URLs when available

#### T025: Implement admin-only endpoints
**File**: `backend/src/api/routes/runs.py` (DELETE, cancel, rerun)
**Prerequisites**: T017, T018
- Create DELETE /api/runs/{id} (admin only)
- Create POST /api/runs/{id}/cancel (admin only)
- Create POST /api/runs/{id}/rerun
- Add role-based middleware

### Phase 6: Frontend Core (T026-T030)
**Can run in parallel**: T026-T029 [P]
**Estimated time**: 8-10 hours

#### T026: Create design system with Vanilla Extract [P]
**File**: `frontend/src/lib/styles/tokens.css.ts`, `frontend/src/lib/styles/global.css.ts`
- Define design tokens (colors, spacing, typography)
- Create global styles
- Setup theme variables
- Configure Lightning CSS optimization

#### T027: Implement authentication store and guard [P]
**File**: `frontend/src/lib/stores/auth.ts`, `frontend/src/routes/+layout.svelte`
**Prerequisites**: T007
- Create auth store with user/role state
- Implement session checking
- Add route guards for protected pages
- Create login redirect logic

#### T028: Create CSV upload component [P]
**File**: `frontend/src/lib/components/CSVUploader.svelte`
**Prerequisites**: T007
- Implement file input with drag-drop
- Parse CSV with papaparse
- Show preview table
- Client-side validation with zod
- Display validation errors inline

#### T029: Create run status component [P]
**File**: `frontend/src/lib/components/RunStatus.svelte`
**Prerequisites**: T007
- Display current status with progress
- Implement 1-second polling with @tanstack/svelte-query
- Show completion percentage
- Add cancel button for admins

#### T030: Create provenance display component [P]
**File**: `frontend/src/lib/components/ProvenanceBox.svelte`
- Display workflow version
- Show container digests
- List execution parameters
- Format as collapsible box

### Phase 7: Frontend Pages (T031-T035)
**Can run in parallel**: T032-T034 [P] after T031
**Estimated time**: 6-8 hours

#### T031: Implement login page
**File**: `frontend/src/routes/+page.svelte`
**Prerequisites**: T027
- Create passphrase input form
- Add role selection (Admin/Runner)
- Handle login API call
- Redirect to /runs on success

#### T032: Implement new run page [P]
**File**: `frontend/src/routes/runs/new/+page.svelte`
**Prerequisites**: T028
- Integrate CSV uploader component
- Add validate button
- Show validation results
- Add run button when valid
- Handle run creation and redirect

#### T033: Implement run details page [P]
**File**: `frontend/src/routes/runs/[id]/+page.svelte`
**Prerequisites**: T029, T030
- Load run details on mount
- Show status with live updates
- Display report link when done
- Show provenance information
- Add re-run button

#### T034: Implement runs list page [P]
**File**: `frontend/src/routes/runs/+page.svelte`
**Prerequisites**: T007
- Fetch and display all runs
- Add status filters
- Show run metadata
- Link to detail pages
- Add delete button for admins

#### T035: Add error handling and loading states
**File**: All page components
**Prerequisites**: T032, T033, T034
- Add loading spinners
- Handle API errors gracefully
- Show user-friendly error messages
- Add retry mechanisms

### Phase 8: Integration & Testing (T036-T040)
**Can run in parallel**: T037-T039 [P] after T036
**Estimated time**: 6-8 hours

#### T036: Integration tests for complete workflows
**File**: `backend/tests/integration/test_workflows.py`
**Prerequisites**: T025
- Test complete upload→validate→run→report flow
- Test admin operations
- Test error scenarios
- Test concurrent runs

#### T037: End-to-end Playwright tests [P]
**File**: `frontend/tests/e2e/upload-to-report.spec.ts`
**Prerequisites**: T035
- Test login flow
- Test CSV upload and validation
- Test run creation and monitoring
- Test report access
- Test re-run functionality

#### T038: Performance testing [P]
**File**: `tests/performance/test_validation.py`
**Prerequisites**: T036
- Test 200-row CSV validates in <2s
- Test first paint <2s
- Verify 1s polling interval
- Test concurrent user load

#### T039: Security audit [P]
**File**: `tests/security/audit.py`
**Prerequisites**: T036
- Check for SQL injection
- Verify session security
- Test role-based access
- Scan for exposed secrets

#### T040: Documentation and deployment prep
**File**: `README.md`, `docs/deployment.md`
**Prerequisites**: All previous tasks
- Write deployment guide
- Document environment variables
- Create Docker Compose setup
- Add production config examples
- Update quickstart guide

## Execution Examples

### Running parallel tasks with Task agent
```bash
# After T001 completes, run T002 and T003 in parallel:
Task T002: Configure backend project with uv
Task T003: Configure frontend project with SvelteKit

# After T006 completes, run type generation in parallel:
Task T007: Generate TypeScript types from OpenAPI
Task T008: Generate Pydantic models from OpenAPI

# After T011 completes, run all models in parallel:
Task T012: Implement User model
Task T013: Implement Run model
Task T014: Implement SampleSheet model
Task T015: Implement Report and Provenance models
```

### Sequential task execution
```bash
# These must run in order:
Task T001: Initialize project structure
Task T004: Setup linting (needs T002, T003)
Task T005: Create CI/CD pipeline (needs T004)
Task T011: Create database schema
Task T016: Create CSV validation service
Task T022: Implement validation endpoint (needs T016)
```

## Success Criteria
- [ ] All 40 tasks completed
- [ ] `mise run lint` passes with no errors
- [ ] `mise run typecheck` passes with no errors
- [ ] `mise run test` passes with >80% coverage
- [ ] Playwright E2E tests pass
- [ ] Performance requirements met (<2s validation, <2s first paint)
- [ ] OpenAPI contract tests pass
- [ ] Deployment successful

## Notes
- Tasks marked [P] can be executed in parallel as they work on different files
- Each task is self-contained with clear inputs/outputs
- Follow TDD approach: write tests before implementation
- Commit after each task with conventional commits format
- Use feature flags for incomplete features