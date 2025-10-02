# Feature Specification: RNA-seq Workflow Runner

**Feature Branch**: `001-build-a-barebones`
**Created**: 2025-10-02
**Status**: Draft
**Input**: User description: "Build a barebones application that lets a bench scientist: upload a CSV sample sheet, 2) see row-level validation, 3) click 'Run' to start a single pinned RNA-seq workflow, and 4) open a single HTML report when done."

## Clarifications

### Session 2025-10-02
- Q: What authentication method should the login system use? → A: simple passphrase
- Q: How should Admin and Runner roles differ in v0? → A: Only Admins can delete/cancel runs; both can create/view
- Q: How long should completed runs and their reports be retained? → A: Auto-delete after 30 days
- Q: How often should the run status page refresh during execution? → A: Real-time updates (every second)
- Q: Should the system validate file accessibility when processing the CSV? → A: Check existence and read permissions

## Execution Flow (main)
```
1. User logs into application
   → Simple passphrase authentication (single shared passphrase for v0)
2. User navigates to /runs/new
   → Upload CSV sample sheet
3. System validates CSV immediately
   → Display row-level errors if invalid
   → Show success if valid
4. User clicks "Run" button
   → System generates run_id
   → Redirects to /runs/[id]
5. System executes pinned RNA-seq workflow
   → Poll status updates every second
   → Display real-time progress to user
6. On completion, show report link
   → User can view HTML report
   → Display provenance information
7. User can re-run with same parameters
   → "Re-run exact" button available
```

---

## ⚡ Quick Guidelines
- ✅ Focus on WHAT users need and WHY
- ❌ Avoid HOW to implement (no tech stack, APIs, code structure)
- 👥 Written for business stakeholders, not developers

### Section Requirements
- **Mandatory sections**: Must be completed for every feature
- **Optional sections**: Include only when relevant to the feature
- When a section doesn't apply, remove it entirely (don't leave as "N/A")

### For AI Generation
When creating this spec from a user prompt:
1. **Mark all ambiguities**: Use [NEEDS CLARIFICATION: specific question] for any assumption you'd need to make
2. **Don't guess**: If the prompt doesn't specify something (e.g., "login system" without auth method), mark it
3. **Think like a tester**: Every vague requirement should fail the "testable and unambiguous" checklist item
4. **Common underspecified areas**:
   - User types and permissions
   - Data retention/deletion policies  
   - Performance targets and scale
   - Error handling behaviors
   - Integration requirements
   - Security/compliance needs

---

## User Scenarios & Testing *(mandatory)*

### Primary User Story
A bench scientist needs to run RNA-seq analysis on their samples. They log into the system, upload a CSV file containing sample information, verify the data is correct, start the analysis workflow, monitor its progress, and download the results report when complete. The system ensures reproducibility by tracking exact versions and parameters used.

### Acceptance Scenarios
1. **Given** a user on the /runs/new page with an invalid CSV (missing required header), **When** they upload the file, **Then** they see specific error message indicating which header is missing within 2 seconds
2. **Given** a user uploads a valid CSV with 12 samples, **When** they click "Run", **Then** they receive a run_id and are redirected to /runs/[id] showing "Processing" status
3. **Given** a completed successful run, **When** viewing /runs/[id], **Then** the user sees a clickable report link and provenance box showing workflow version and container digests
4. **Given** a user viewing a completed run, **When** they click "Re-run exact", **Then** a new run is created with identical parameters and the user is redirected to the new run's page
5. **Given** a CSV with duplicate sample_id values, **When** uploaded, **Then** the system displays row-level errors highlighting the duplicate entries
6. **Given** a CSV with non-.fastq.gz file extensions, **When** uploaded, **Then** the system displays errors for the affected rows indicating invalid file format
7. **Given** a CSV with inaccessible file URIs, **When** uploaded, **Then** the system displays row-level errors for files that don't exist or lack read permissions

*Note: Constitution requires Playwright smoke test for Upload → Validate → Run → Report path*

### Edge Cases
- What happens when uploading a CSV with more than 12 rows? System rejects with clear error message
- How does system handle malformed CSV files? Display parsing error with line number
- What if workflow fails mid-execution? Status page shows "Failed" with error information
- How does system handle simultaneous uploads from same user? Each gets unique run_id
- What if input files become inaccessible after validation? Workflow fails at execution with clear error message

## Requirements *(mandatory)*

### Non-Functional Requirements
- **NFR-001**: System MUST complete CSV validation within 2 seconds for files up to 200 rows
- **NFR-002**: System MUST achieve first paint in under 2 seconds on broadband connections (>25 Mbps)
- **NFR-003**: System MUST handle up to 10 concurrent users without performance degradation
- **NFR-004**: System MUST maintain 99% uptime during business hours (8 AM - 6 PM)
- **NFR-005**: System MUST use HTTPS for all communications in production
- **NFR-006**: System MUST log all errors with sufficient context for debugging without exposing sensitive data
- **NFR-007**: System MUST support modern browsers (Chrome/Edge 90+, Firefox 88+, Safari 14+)
- **NFR-008**: System MUST be deployable via Docker Compose with single command
- **NFR-009**: System MUST not store passwords or secrets in plain text
- **NFR-010**: System MUST gracefully handle workflow failures with clear error messages

### Functional Requirements
- **FR-001**: System MUST provide a login page at /login using a single shared passphrase for authentication
- **FR-002**: System MUST distinguish between Admin and Runner user types (Admins can delete/cancel runs; both can create/view)
- **FR-003**: System MUST provide CSV upload capability at /runs/new page
- **FR-021**: System MUST allow Admin users to cancel in-progress runs
- **FR-022**: System MUST allow Admin users to delete completed or failed runs
- **FR-004**: System MUST validate CSV files contain exactly these headers: sample_id, group, read1_uri, read2_uri
- **FR-005**: System MUST ensure all sample_id values in CSV are unique
- **FR-006**: System MUST verify all URI values end with .fastq.gz extension and validate file existence with read permissions
- **FR-007**: System MUST reject CSV files containing more than 12 rows of data
- **FR-008**: System MUST provide row-level validation feedback within 2 seconds for small CSVs
- **FR-009**: System MUST generate a unique run_id upon successful validation and "Run" button click
- **FR-010**: System MUST redirect users to /runs/[id] after initiating a run
- **FR-011**: System MUST display current processing status on /runs/[id] page with real-time updates (every second)
- **FR-012**: System MUST execute a single, pinned version of the RNA-seq workflow
- **FR-013**: System MUST generate an HTML report upon successful workflow completion
- **FR-014**: System MUST display a clickable link to the HTML report on completion
- **FR-015**: System MUST show provenance box containing workflow version and container digests from manifest
- **FR-016**: System MUST provide "Re-run exact" button for completed runs
- **FR-017**: System MUST create new run with identical parameters when "Re-run exact" is clicked
- **FR-018**: System MUST provide /runs page listing all runs for the user
- **FR-019**: System MUST complete successful runs and generate report within 15 minutes for typical 12-sample runs
- **FR-020**: System MUST use non-technical language in all user-facing messages
- **FR-023**: System MUST automatically delete runs and associated reports after 30 days from completion

### Key Entities *(include if feature involves data)*
- **Session**: Represents authenticated sessions with role selection (Admin or Runner) at login time, stored in secure cookies with 24hr expiry. No persistent user records are stored - roles are session-only
- **Run**: Represents a workflow execution instance with unique run_id, status, associated CSV data, timestamps, auto-deleted after 30 days
- **Sample Sheet**: CSV file containing sample metadata with required headers and validation rules
- **Workflow**: Single pinned RNA-seq analysis pipeline executed via Nextflow DSL2 (nf-core/rnaseq v3.12.0) with fixed version and container specifications using sha256 digests
- **Report**: HTML output file generated after successful workflow completion, linked to specific run
- **Provenance**: Metadata tracking workflow version, container digests, and execution parameters

---

## Review & Acceptance Checklist
*GATE: Automated checks run during main() execution*

### Content Quality
- [x] No implementation details (languages, frameworks, APIs)
- [x] Focused on user value and business needs
- [x] Written for non-technical stakeholders
- [x] All mandatory sections completed

### Requirement Completeness
- [x] No [NEEDS CLARIFICATION] markers remain
- [x] Requirements are testable and unambiguous
- [x] Success criteria are measurable
- [x] Scope is clearly bounded
- [x] Dependencies and assumptions identified

---

## Execution Status
*Updated by main() during processing*

- [x] User description parsed
- [x] Key concepts extracted
- [x] Ambiguities marked (none found - description was comprehensive)
- [x] User scenarios defined
- [x] Requirements generated
- [x] Entities identified
- [x] Review checklist passed

---

## Out of Scope (v0)
As explicitly defined in the feature description:
- SSO/roles (beyond basic Admin/Runner distinction)
- Multiple workflows (only single pinned RNA-seq workflow)
- Dockstore import functionality
- Cloud bursting capabilities
- Dashboard views beyond basic /runs listing
