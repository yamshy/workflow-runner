# Implementation Tasks: RNA-seq Workflow Runner

**Feature Branch**: `001-build-a-barebones`
**Generated**: 2025-10-02
**Total Tasks**: 45
**Estimated Hours**: 65-85

## Overview
Build a barebones web application for RNA-seq workflow execution with CSV validation, status tracking, and report generation. Implementation follows UI-first + contract-first approach with parallel frontend/backend development.

## Tech Stack
- **Backend**: FastAPI, Python 3.x, SQLite, pyright, ruff, uv
- **Frontend**: SvelteKit, TypeScript, Vanilla Extract, @tanstack/svelte-query, zod
- **Testing**: pytest, Playwright, Prism
- **Tooling**: mise, pnpm workspaces, Biome

## Task Execution

### Phase 1: Project Setup & Tooling (T001-T006)
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

#### T004: Setup linting, formatting, and commit standards
**File**: `.mise.toml` (tasks section), `.gitmessage`, `.husky/`
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
- Setup pre-commit hooks for linting and Conventional Commits:
  - Install husky: `pnpm add -D husky @commitlint/cli @commitlint/config-conventional`
  - Configure commitlint for Conventional Commits enforcement
  - Create commit message template with type(scope): format
  - Add commit-msg hook to validate format
  - Valid types: feat, fix, docs, style, refactor, test, chore, perf, ci, build, revert

#### T005: Setup Nextflow runtime environment
**File**: `.mise.toml`, `backend/config/nextflow.config`
**Prerequisites**: T001
- Add Nextflow to `.mise.toml`:
  - nextflow 23.10.x (LTS version)
  - docker or singularity for container runtime
- Create Nextflow configuration:
  - Set nf-core/rnaseq v3.12.0 as pinned workflow
  - Configure container registry with sha256 digests
  - Set resource limits (CPU, memory)
  - Configure work directory location
  - Set execution profiles (local, docker)
- Download and cache workflow:
  - `nextflow pull nf-core/rnaseq -revision 3.12.0`
  - Verify workflow manifest
- Create wrapper script for workflow execution
- Test Nextflow installation with dry-run

#### T006: Create CI/CD pipeline configuration
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

### Phase 2: API Contract & Mocking (T007-T011)
**Can run in parallel**: T007-T009 [P]
**Estimated time**: 3-4 hours

#### T007: Create OpenAPI specification [P]
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

#### T008: Generate TypeScript types from OpenAPI [P]
**File**: `frontend/src/lib/types/api.ts`
**Prerequisites**: T007
- Run openapi-typescript to generate types
- Create type-safe API client wrapper
- Export request/response interfaces

#### T009: Generate Pydantic models from OpenAPI [P]
**File**: `backend/src/api/schemas/`
**Prerequisites**: T007
- Create Pydantic models for all schemas:
  - AuthRequest, AuthResponse
  - ValidationRequest, ValidationResponse
  - RunCreateRequest, RunResponse
  - ErrorResponse
- Add custom validators for business rules

#### T010: Setup Prism mock server
**File**: `scripts/start-mock.sh`
**Prerequisites**: T007
- Create script to start Prism with OpenAPI spec
- Configure dynamic response selection
- Add to mise tasks for easy startup

#### T011: Create API contract tests
**File**: `backend/tests/contract/test_openapi.py`
**Prerequisites**: T007, T009
- Use schemathesis to validate API against OpenAPI
- Create fixture for test client
- Test all endpoints match specification
- These tests should fail initially (TDD approach)

### Phase 3: Database & Models (T012-T016)
**Can run in parallel**: T013-T016 [P] after T012
**Estimated time**: 4-5 hours

#### T012: Create database schema and migrations
**File**: `backend/src/models/database.py`, `backend/migrations/`
- Create SQLite database setup with aiosqlite
- Define schema creation script:
  - runs table
  - sample_sheets table
  - reports table
  - provenance table
  - sessions table (temporary auth storage only - 24hr TTL, no persistent user data per spec requirement)
- Add indexes for performance
- Create CLI command for initialization

#### T013: Implement simple role configuration [P]
**File**: `backend/src/models/roles.py`
**Prerequisites**: T012
- Create simple role enumeration (Admin/Runner)
- NO user table needed - roles selected at login and stored in temporary session only
- Store role in session cookie only
- Sessions expire after 24 hours and are not user accounts
- Add role validation helpers

#### T014: Implement Run model [P]
**File**: `backend/src/models/run.py`
**Prerequisites**: T012
- Create Run model with status state machine
- Add methods: create(), update_status(), get_by_user()
- Implement 30-day auto-deletion logic
- Add JSON field handling for sample_data

#### T015: Implement SampleSheet model [P]
**File**: `backend/src/models/sample_sheet.py`
**Prerequisites**: T012
- Create SampleSheet model
- Add validation error storage
- Implement relationship to Run
- Add methods for validation status

#### T016: Implement Report and Provenance models [P]
**File**: `backend/src/models/report.py`, `backend/src/models/provenance.py`
**Prerequisites**: T012
- Create Report model with file path tracking
- Create Provenance model with immutable fields
- Add manifest hash generation
- Implement cascade delete with Run

### Phase 4: Core Services (T017-T021)
**Can run in parallel**: T018-T020 [P] after T017
**Estimated time**: 6-8 hours

#### T017: Create CSV validation service
**File**: `backend/src/services/validator.py`
- Implement CSV validation rules:
  - Check required headers (sample_id, group, read1_uri, read2_uri)
  - Validate unique sample_ids
  - Check .fastq.gz extensions
  - Verify file existence and read permissions
  - Max 12 rows validation
- Return row-level error details
- Add unit tests in `backend/tests/unit/test_validator.py`

#### T018: Create authentication service [P]
**File**: `backend/src/services/auth.py`
**Prerequisites**: T013
- Implement single shared passphrase validation (from environment variable)
- Create session with selected role (Admin/Runner)
- Generate secure session IDs
- Set 24hr session expiry
- NO user database operations needed

#### T019: Create workflow runner service with real execution [P]
**File**: `backend/src/services/runner.py`
**Prerequisites**: T005, T014
- Implement actual workflow execution using Nextflow runtime
- Execute pinned RNA-seq workflow (nf-core/rnaseq v3.12.0) via subprocess
- Capture stdout/stderr for debugging
- Implement state transitions (queued→running→done/failed)
- Generate real MultiQC HTML report from workflow outputs
- Store workflow manifest with container SHAs
- Handle workflow failures gracefully

#### T019a: Create workflow manifest service
**File**: `backend/src/services/manifest.py`
**Prerequisites**: T019
- Parse workflow definition for container images
- Extract sha256 digests from Docker registry
- Generate immutable manifest JSON
- Store manifest with each run
- Implement manifest comparison for re-run validation

#### T020: Create report storage service [P]
**File**: `backend/src/services/storage.py`
**Prerequisites**: T016
- Setup static file serving directory
- Implement report file management
- Add cleanup job for 30-day retention
- Generate public URLs for reports

#### T020a: Create cleanup scheduler for 30-day retention
**File**: `backend/src/services/cleanup.py`
**Prerequisites**: T014, T020
**Implements**: FR-023 (30-day auto-deletion requirement)
- Implement scheduled job (cron or APScheduler)
- Query runs older than 30 days from completion
- Delete associated reports from filesystem
- Delete run records from database
- Log cleanup operations
- Run daily at low-traffic time (e.g., 3 AM)
- Ensure idempotent operation (safe to re-run)

#### T021: Unit tests for all services
**File**: `backend/tests/unit/test_*.py`
**Prerequisites**: T017, T018, T019, T020
- Write comprehensive unit tests
- Mock database interactions
- Test validation edge cases
- Test authentication flows
- Achieve >80% coverage

#### T021a: Create user message validation
**File**: `backend/src/services/messages.py`, `backend/tests/unit/test_messages.py`
**Prerequisites**: T021
- Define non-technical message templates
- Create message formatter with placeholders
- Write unit tests to verify no technical jargon in user-facing strings
- Terms to avoid: "exception", "null", "undefined", "stack trace", "segfault"
- Use friendly alternatives: "error", "not found", "missing", "problem occurred"

### Phase 5: API Implementation (T022-T026)
**Can run in parallel**: None (depends on services)
**Estimated time**: 5-6 hours

#### T022: Implement authentication endpoint
**File**: `backend/src/api/routes/auth.py`
**Prerequisites**: T018
- Create POST /api/auth/login endpoint
- Validate passphrase
- Create session cookie
- Return user info and role

#### T023: Implement validation endpoint
**File**: `backend/src/api/routes/validation.py`
**Prerequisites**: T017
- Create POST /api/validate endpoint
- Parse multipart form upload
- Run CSV validation
- Return row-level errors

#### T024: Implement runs creation endpoint
**File**: `backend/src/api/routes/runs.py` (POST /api/runs)
**Prerequisites**: T019
- Create POST /api/runs endpoint
- Validate CSV before creating run
- Start background runner task
- Return run_id and initial status

#### T025: Implement runs query endpoints
**File**: `backend/src/api/routes/runs.py` (GET endpoints)
**Prerequisites**: T014
- Create GET /api/runs endpoint with filtering
- Create GET /api/runs/{id} with full details
- Include provenance data
- Add report URLs when available

#### T026: Implement admin-only endpoints
**File**: `backend/src/api/routes/runs.py` (DELETE, cancel, rerun)
**Prerequisites**: T018, T019
- Create DELETE /api/runs/{id} (admin only)
- Create POST /api/runs/{id}/cancel (admin only)
- Create POST /api/runs/{id}/rerun
- Add role-based middleware

#### T026a: Implement health check endpoint
**File**: `backend/src/api/routes/health.py`
**Prerequisites**: T022
**Implements**: NFR-004 (99% uptime requirement)
- Create GET /api/health endpoint
- Return service status and version info
- Check database connectivity
- Verify Nextflow runtime availability
- Return 200 OK if healthy, 503 if degraded
- Add uptime counter for monitoring

### Phase 6: Frontend Core (T027-T031)
**Can run in parallel**: T027-T030 [P]
**Estimated time**: 8-10 hours

#### T027: Create design system with Vanilla Extract [P]
**File**: `frontend/src/lib/styles/tokens.css.ts`, `frontend/src/lib/styles/global.css.ts`
- Define design tokens (colors, spacing, typography)
- Create global styles
- Setup theme variables
- Configure Lightning CSS optimization

#### T028: Implement authentication store and guard [P]
**File**: `frontend/src/lib/stores/auth.ts`, `frontend/src/routes/+layout.svelte`
**Prerequisites**: T008
- Create auth store with user/role state
- Implement session checking
- Add route guards for protected pages
- Create login redirect logic

#### T029: Create CSV upload component [P]
**File**: `frontend/src/lib/components/CSVUploader.svelte`
**Prerequisites**: T008
- Implement file input with drag-drop
- Parse CSV with papaparse
- Show preview table
- Client-side validation with zod
- Display validation errors inline

#### T030: Create run status component [P]
**File**: `frontend/src/lib/components/RunStatus.svelte`
**Prerequisites**: T008
- Display current status with progress
- Implement 1-second polling with @tanstack/svelte-query
- Show completion percentage
- Add cancel button for admins

#### T031: Create provenance display component [P]
**File**: `frontend/src/lib/components/ProvenanceBox.svelte`
- Display workflow version
- Show container digests
- List execution parameters
- Format as collapsible box

### Phase 7: Frontend Pages (T032-T036)
**Can run in parallel**: T033-T035 [P] after T032
**Estimated time**: 6-8 hours

#### T032: Implement login page
**File**: `frontend/src/routes/+page.svelte`
**Prerequisites**: T028
- Create passphrase input form
- Add role selection (Admin/Runner)
- Handle login API call
- Redirect to /runs on success

#### T033: Implement new run page [P]
**File**: `frontend/src/routes/runs/new/+page.svelte`
**Prerequisites**: T029
- Integrate CSV uploader component
- Add validate button
- Show validation results
- Add run button when valid
- Handle run creation and redirect

#### T034: Implement run details page [P]
**File**: `frontend/src/routes/runs/[id]/+page.svelte`
**Prerequisites**: T030, T031
- Load run details on mount
- Show status with live updates
- Display report link when done
- Show provenance information
- Add re-run button

#### T035: Implement runs list page [P]
**File**: `frontend/src/routes/runs/+page.svelte`
**Prerequisites**: T008
- Fetch and display all runs
- Add status filters
- Show run metadata
- Link to detail pages
- Add delete button for Admin users (FR-022)
- Add cancel button for in-progress runs for Admin users (FR-021)
- Show role-based UI elements conditionally
- Confirm destructive actions with modal dialog

#### T036: Add error handling and loading states
**File**: All page components
**Prerequisites**: T033, T034, T035
- Add loading spinners
- Handle API errors gracefully
- Show user-friendly error messages
- Add retry mechanisms

### Phase 8: Integration & Testing (T037-T041)
**Can run in parallel**: T038-T040 [P] after T037
**Estimated time**: 6-8 hours

#### T037: Integration tests for complete workflows
**File**: `backend/tests/integration/test_workflows.py`
**Prerequisites**: T026
- Test complete upload→validate→run→report flow
- Test admin operations
- Test error scenarios
- Test concurrent runs

#### T038: End-to-end Playwright tests [P]
**File**: `frontend/tests/e2e/upload-to-report.spec.ts`
**Prerequisites**: T036
- Test login flow
- Test CSV upload and validation
- Test run creation and monitoring
- Test report access
- Test re-run functionality

#### T039: Performance testing [P]
**File**: `tests/performance/test_validation.py`
**Prerequisites**: T037
- Test 200-row CSV validates in <2s
- Test first paint <2s
- Verify 1s polling interval
- Test concurrent user load

#### T040: Security audit [P]
**File**: `tests/security/audit.py`
**Prerequisites**: T037
- Check for SQL injection
- Verify session security
- Test role-based access
- Scan for exposed secrets

#### T041: Documentation and deployment prep
**File**: `README.md`, `docs/deployment.md`
**Prerequisites**: All previous tasks
- Write deployment guide
- Document environment variables
- Create Docker Compose setup
- Add production config examples
- Update quickstart guide
- Verify browser compatibility (Chrome/Edge 90+, Firefox 88+, Safari 14+) per NFR-007

## Execution Examples

### Running parallel tasks with Task agent
```bash
# After T001 completes, run T002 and T003 in parallel:
Task T002: Configure backend project with uv
Task T003: Configure frontend project with SvelteKit

# After T007 completes, run type generation in parallel:
Task T008: Generate TypeScript types from OpenAPI
Task T009: Generate Pydantic models from OpenAPI

# After T012 completes, run all models in parallel:
Task T013: Implement simple role configuration
Task T014: Implement Run model
Task T015: Implement SampleSheet model
Task T016: Implement Report and Provenance models
```

### Sequential task execution
```bash
# These must run in order:
Task T001: Initialize project structure
Task T004: Setup linting (needs T002, T003)
Task T006: Create CI/CD pipeline (needs T004)
Task T012: Create database schema
Task T017: Create CSV validation service
Task T023: Implement validation endpoint (needs T017)
```

## Success Criteria
- [ ] All 45 tasks completed
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