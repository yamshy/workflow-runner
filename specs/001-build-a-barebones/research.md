# Research & Technical Decisions

**Feature**: RNA-seq Workflow Runner
**Date**: 2025-10-02
**Status**: Complete

## Executive Summary
Research conducted to resolve technical decisions for implementing a UI-first, contract-first RNA-seq workflow runner with FastAPI backend and SvelteKit frontend.

## Key Decisions

### 1. API Mocking Strategy
**Decision**: Use Prism for OpenAPI mocking
**Rationale**:
- Automatically generates mock responses from OpenAPI spec
- Zero configuration required
- Supports example values and response variations
- Enables immediate frontend development
**Alternatives considered**:
- MSW: More setup, requires manual mock definitions
- Custom mock server: Too much overhead for MVP

### 2. CSV Validation Architecture
**Decision**: Two-tier validation (client + server)
**Rationale**:
- Client-side (zod): Instant feedback, no server round-trip
- Server-side (Python with pyright): File access checks, authoritative validation
- Papaparse for robust CSV parsing on frontend
**Alternatives considered**:
- Server-only: Poor UX with network latency
- Client-only: Cannot validate file accessibility

### 3. Workflow Runner Implementation
**Decision**: Background task with status queue (v0 stub)
**Rationale**:
- Simple async pattern using FastAPI BackgroundTasks
- SQLite for persistent state across restarts
- Stub implementation simulates workflow (queued→running→done)
- Easy swap to real Nextflow later
**Alternatives considered**:
- Celery: Overkill for single workflow
- Direct Nextflow integration: Too complex for MVP

### 4. Authentication Implementation
**Decision**: Session-based with shared passphrase
**Rationale**:
- Spec clarified: single shared passphrase for v0
- Simple session storage in SQLite
- Role stored in session after login
**Alternatives considered**:
- JWT tokens: Unnecessary complexity for shared passphrase
- No auth: Security requirement per spec

### 5. Real-time Status Updates
**Decision**: Client-side polling with @tanstack/svelte-query
**Rationale**:
- Simpler than WebSockets for MVP
- @tanstack/svelte-query handles caching and refetch logic
- 1-second interval per user requirement
**Alternatives considered**:
- WebSockets: More complex setup, overkill for status updates
- SSE: Not well supported in all environments

### 6. Report Storage
**Decision**: Static file server with nginx/FastAPI
**Rationale**:
- HTML reports are static files
- Simple URL pattern: /static/reports/{run_id}/report.html
- 30-day cleanup via cron job
**Alternatives considered**:
- Database storage: Inefficient for large HTML files
- S3: Unnecessary external dependency for MVP

### 7. Provenance Tracking
**Decision**: JSON manifest in SQLite
**Rationale**:
- Store workflow version, container digests, parameters
- Immutable once created
- Enables "Re-run exact" feature
**Alternatives considered**:
- Separate provenance service: Over-engineered for MVP
- File-based: Harder to query and manage

### 8. Frontend State Management
**Decision**: SvelteKit built-in stores + @tanstack/svelte-query
**Rationale**:
- Svelte stores for auth state
- @tanstack/svelte-query for server state (runs, status)
- No additional state library needed
**Alternatives considered**:
- Redux: Overkill for simple state
- Zustand: Unnecessary with Svelte's built-in stores

### 9. Styling Architecture
**Decision**: Vanilla Extract + Svelte scoped styles + Lightning CSS
**Rationale**:
- Vanilla Extract for type-safe design tokens and themes
- Svelte scoped styles for component-specific CSS
- Lightning CSS for optimized build output
- Aligns with constitutional requirements
**Alternatives considered**:
- Tailwind: Violates constitution (utility frameworks prohibited)
- CSS Modules: Less integrated with Svelte
- Plain CSS: No type safety or design system support

### 10. Type Generation Strategy
**Decision**: openapi-typescript for frontend types
**Rationale**:
- Generates TypeScript types from OpenAPI spec
- Single source of truth for API contract
- Automatic synchronization
**Alternatives considered**:
- Manual type definitions: Error-prone, drift over time
- GraphQL: Different paradigm, more complex setup

### 11. Testing Strategy
**Decision**: Layered testing approach with mise orchestration
**Rationale**:
- Unit tests: CSV validator logic (pytest)
- Contract tests: OpenAPI compliance (schemathesis)
- E2E tests: Critical user path (Playwright)
- Mock-first development with Prism
- All tests run via `mise run test`
**Alternatives considered**:
- Integration-only: Slower feedback loop
- Unit-only: Misses interaction bugs

### 12. Development Tooling
**Decision**: mise for unified tool management
**Rationale**:
- Single `.mise.toml` controls all versions (Node, Python, uv, pnpm, etc.)
- Consistent environments across all developers
- Simple commands: `mise run lint`, `mise run typecheck`, `mise run test`
- Constitutional requirement for pinned versions
**Alternatives considered**:
- asdf: Less integrated task runner
- Docker-only: Slower for local development
- Manual setup: Prone to version drift

## Implementation Notes

### OpenAPI First
1. Define OpenAPI spec with examples
2. Generate types for both frontend and backend
3. Run Prism mock server
4. Develop frontend against mocks
5. Implement backend to match contract

### Parallel Development Flow
- Frontend team works against Prism mocks immediately
- Backend team implements to match OpenAPI contract
- Contract tests ensure compatibility
- Integration happens naturally when both match spec

### File Validation Approach
```python
# Server-side validation includes file access checks
def validate_sample_sheet(csv_data: List[Dict]) -> ValidationResult:
    errors = []
    for row_num, row in enumerate(csv_data):
        # Check file existence and permissions
        for uri_field in ['read1_uri', 'read2_uri']:
            if not os.path.exists(row[uri_field]):
                errors.append(f"Row {row_num}: File not found: {row[uri_field]}")
            elif not os.access(row[uri_field], os.R_OK):
                errors.append(f"Row {row_num}: No read permission: {row[uri_field]}")
    return ValidationResult(valid=len(errors)==0, errors=errors)
```

### Stub Runner Pattern
```python
# Temporary workflow runner for v0
async def stub_runner(run_id: str):
    # Simulate workflow execution
    await update_status(run_id, "running")
    await asyncio.sleep(random.randint(5, 15))  # Simulate work

    # Generate placeholder report
    report_html = generate_multiqc_placeholder()
    save_report(run_id, report_html)

    await update_status(run_id, "completed")
```

## Resolved Questions
✅ All technical decisions resolved
✅ No NEEDS CLARIFICATION items remaining
✅ Implementation path clear

## Next Steps
→ Phase 1: Generate OpenAPI spec, data models, and quickstart guide